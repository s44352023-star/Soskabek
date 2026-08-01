Mana siz so'ragan talablar asosida to'liq, mukammal va professional darajada loyihalashtirilgan Telegram E-commerce / Delivery Bot arxitekturasi va uning asosiy qismlari uchun kodlar to'plami.

📁 1. Toza Loyiha Strukturasi (Project Structure)
Loyihani kengaytirish qulay bo'lishi uchun src/ papkasi ostida quyidagi arxitekturadan foydalanamiz:

Plaintext
ecommerce_bot/
├── .env.example
├── .gitignore
├── Dockerfile
├── docker-compose.yml
├── pyproject.toml
├── README.md
├── alembic.ini
├── alembic/
│   ├── env.py
│   └── versions/
├── src/
│   ├── __init__.py
│   ├── config.py
│   ├── main.py
│   ├── database/
│   │   ├── base.py
│   │   └── session.py
│   ├── models/
│   │   ├── user.py
│   │   ├── product.py
│   │   ├── order.py
│   │   └── ...
│   ├── repositories/
│   │   ├── user_repo.py
│   │   └── product_repo.py
│   ├── services/
│   │   ├── order_service.py
│   │   └── payment_service.py
│   ├── handlers/
│   │   ├── user/
│   │   │   ├── catalog.py
│   │   │   ├── cart.py
│   │   │   └── checkout.py
│   │   ├── admin/
│   │   │   ├── products.py
│   │   │   └── broadcast.py
│   │   └── courier/
│   │       └── delivery.py
│   ├── middlewares/
│   │   ├── i18n.py
│   │   └── db.py
│   └── utils/
│       ├── states.py
│       └── logger.py
└── tests/
    ├── test_user.py
    └── test_order.py
🗄️ 2. SQLAlchemy Async Modellar & 8+ DB Jadvallar
Asosiy jadvallar: Users, Categories, Products, Carts, CartItems, Orders, OrderItems, Couriers, Reviews, Loyalty.

Python
# src/models/base_model.py
from datetime import datetime
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
from sqlalchemy import DateTime

class Base(DeclarativeBase):
    pass

class TimestampMixin:
    created_at: Mapped[datetime] = mapped_column(DateTime, default=datetime.utcnow)
    updated_at: Mapped[datetime] = mapped_column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
Python
# src/models/user.py
from sqlalchemy.orm import Mapped, mapped_column, relationship
from sqlalchemy import String, BigInteger, Enum as SQLEnum
import enum
from .base_model import Base, TimestampMixin

class UserRole(str, enum.Enum):
    USER = "user"
    ADMIN = "admin"
    COURIER = "courier"

class User(Base, TimestampMixin):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    telegram_id: Mapped[int] = mapped_column(BigInteger, unique=True, index=True)
    full_name: Mapped[str] = mapped_column(String(100))
    phone: Mapped[str | None] = mapped_column(String(20), nullable=True)
    language: Mapped[str] = mapped_column(String(5), default="uz")
    role: Mapped[UserRole] = mapped_column(SQLEnum(UserRole), default=UserRole.USER)

    orders = relationship("Order", back_populates="user")
🔄 3. Service / Repository Pattern
Ma'lumotlar bazasi va biznes logikasini bir-biridan ajratish uchun Repository va Service qatlamlari:

Python
# src/repositories/user_repo.py
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from src.models.user import User

class UserRepository:
    def __init__(self, session: AsyncSession):
        self.session = session

    async def get_by_telegram_id(self, telegram_id: int) -> User | None:
        result = await self.session.execute(select(User).where(User.telegram_id == telegram_id))
        return result.scalar_one_or_none()

    async def create_user(self, telegram_id: int, full_name: str, language: str = "uz") -> User:
        user = User(telegram_id=telegram_id, full_name=full_name, language=language)
        self.session.add(user)
        await self.session.commit()
        await self.session.refresh(user)
        return user
🛒 4. Checkout FSM & Cart (Savatcha)
Foydalanuvchi buyurtma berish jarayoni FSM (Finite State Machine) orqali boshqariladi:

Python
# src/utils/states.py
from aiogram.fsm.state import State, StatesGroup

class CheckoutState(StatesGroup):
    waiting_for_phone = State()
    waiting_for_location = State()
    waiting_for_comment = State()
    waiting_for_confirmation = State()
Python
# src/handlers/user/checkout.py
from aiogram import Router, F
from aiogram.types import Message, ReplyKeyboardMarkup, KeyboardButton
from aiogram.fsm.context import FSMContext
from src.utils.states import CheckoutState

router = Router()

@router.message(F.text == "🛍 Buyurtma berish")
async def start_checkout(message: Message, state: FSMContext):
    keyboard = ReplyKeyboardMarkup(
        keyboard=[[KeyboardButton(text="📱 Telefon raqamni yuborish", request_contact=True)]],
        resize_keyboard=True,
        one_time_keyboard=True
    )
    await message.answer("Buyurtmani rasmiylashtirish uchun telefon raqamingizni yuboring:", reply_markup=keyboard)
    await state.set_state(CheckoutState.waiting_for_phone)

@router.message(CheckoutState.waiting_for_phone, F.contact)
async def process_phone(message: Message, state: FSMContext):
    phone = message.contact.phone_number
    await state.update_data(phone=phone)
    
    keyboard = ReplyKeyboardMarkup(
        keyboard=[[KeyboardButton(text="📍 Lokatsiyani yuborish", request_location=True)]],
        resize_keyboard=True,
        one_time_keyboard=True
    )
    await message.answer("Endi yetkazib berish manzilini (lokatsiya) yuboring:", reply_markup=keyboard)
    await state.set_state(CheckoutState.waiting_for_location)
🚴 5. Kuryer Moduli & Yandex Maps
Kuryer yangi buyurtmani qabul qilib, mijoz manziliga yo'naltirilishi:

Python
# src/handlers/curier/delivery.py
from aiogram import Router, F
from aiogram.types import CallbackQuery, Message

router = Router()

@router.callback_query(F.data.startswith("accept_order:"))
async def courier_accept_order(call: CallbackQuery):
    order_id = int(call.data.split(":")[1])
    # Statusni "yetkazilmoqda" ga o'zgartirish logikasi
    
    lat, lon = 41.311081, 69.240562 # Misol uchun mijoz koordinatalari
    yandex_maps_url = f"https://yandex.com/maps/?rtext=~{lat},{lon}"
    
    await call.message.answer(
        f"Siz #{order_id}-sonli buyurtmani qabul qildingiz!\n\n"
        f"Mijoz manziliga yo'l olish uchun Yandex Maps'dan foydalaning:\n"
        f"[Yo'nalishni ochish]({yandex_maps_url})",
        parse_mode="Markdown"
    )
    await call.answer()
🐳 6. Docker & Production Muhiti
docker-compose.yml fayli orqali PostgreSQL, Redis va Bot konteynerlarini birgalikda ishga tushirish:

YAML
version: '3.8'

services:
  db:
    image: postgres:15-alpine
    container_name: postgres_db
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: ecommerce_db
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:alpine
    container_name: redis_cache
    ports:
      - "6379:6379"

  bot:
    build: .
    container_name: aiogram_bot
    command: python src/main.py
    volumes:
      - .:/app
    environment:
      - BOT_TOKEN=your_bot_token_here
      - DATABASE_URL=postgresql+asyncpg://postgres:password@db:5432/ecommerce_db
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis

volumes:
  pgdata:
🧪 7. Pytest Unit Testlar
Biznes logika va ma'lumotlar bazasi amallari uchun testlar:

Python
# tests/test_user.py
import pytest
from src.repositories.user_repo import UserRepository

@pytest.mark.asyncio
async def test_create_user(async_session):
    repo = UserRepository(async_session)
    user = await repo.create_user(telegram_id=123456789, full_name="Test User", language="uz")
    
    assert user.telegram_id == 123456789
    assert user.full_name == "Test User"
    assert user.language == "uz"
🚀 8. Asosiy Ishga Tushirish Fayli (main.py)
Python
# src/main.py
import asyncio
import logging
from aiogram import Bot, Dispatcher
from aiogram.fsm.storage.redis import RedisStorage
from src.config import settings
from src.handlers.user import checkout, catalog
from src.handlers.admin import products, broadcast
from src.handlers.curier import delivery

async def main():
    logging.basicConfig(level=logging.INFO)
    
    storage = RedisStorage.from_url(settings.REDIS_URL)
    bot = Bot(token=settings.BOT_TOKEN)
    dp = Dispatcher(storage=storage)

    # Routerni ulash
    dp.include_router(checkout.router)
    dp.include_router(catalog.router)
    dp.include_router(delivery.router)

    await bot.delete_webhook(drop_pending_updates=True)
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
