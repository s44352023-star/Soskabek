import asyncio
import logging
import os
from aiogram import Bot, Dispatcher, F
from aiogram.client.default import DefaultBotProperties
from aiogram.enums import ParseMode
from aiogram.filters import CommandStart
from aiogram.types import (
    Message,
    ReplyKeyboardMarkup,
    KeyboardButton,
    URLInputFile
)
from dotenv import load_dotenv

# .env faylidan tokenni o'qish
load_dotenv()
BOT_TOKEN = os.getenv("BOT_TOKEN")

logging.basicConfig(level=logging.INFO)

bot = Bot(token=BOT_TOKEN, default=DefaultBotProperties(parse_mode=ParseMode.HTML))
dp = Dispatcher()

# ==================== MA'lumotlar bazasi (PRODUCTS dict) ====================
PRODUCTS = {
    "Pizza": {
        "title": "🍕 Pepperoni Pitsa",
        "price": "85,000 so'm",
        "desc": "Mazali pishloq, kolbasa, maxsus pomidor sousi va ziravorlar bilan tayyorlangan klassik pitsa.",
        "image": "https://images.unsplash.com/photo-1513104890138-7c749659a591?w=600"
    },
    "Burger": {
        "title": "🍔 Cheeseburger",
        "price": "45,000 so'm",
        "desc": "Sharbatli mol go'shti kotleti, chedar pishlog'i, yangi salat barglari va maxsus sous.",
        "image": "https://images.unsplash.com/photo-1568901346375-23c9450c58cd?w=600"
    },
    "Taco": {
        "title": "🌮 Meksikancha Tako",
        "price": "40,000 so'm",
        "desc": "Qarsildoq tako noni ichida tovuq go'shti, sabzavotlar va achchiq meksikancha sous.",
        "image": "https://images.unsplash.com/photo-1551504734-5ee1c4a1479b?w=600"
    },
    "Lag'mon": {
        "title": "🍜 Uycha Lag'mon",
        "price": "38,000 so'm",
        "desc": "Cho'zilma qo'lbola xamir, tansiq go'sht va sarxil sabzavotlardan tayyorlangan milliy taom.",
        "image": "https://images.unsplash.com/photo-1569718212165-3a8278d5f624?w=600"
    },
    "Choy": {
        "title": "🍵 Ko'k Choy (Choynakda)",
        "price": "8,000 so'm",
        "desc": "An'anaviy damlangan xushbo'y chinni choynakdagi ko'k choy.",
        "image": "https://images.unsplash.com/photo-1576092768241-dec231879fc3?w=600"
    },
    "Kofe": {
        "title": "☕ Cappuccino",
        "price": "25,000 so'm",
        "desc": "Kuchli espresso va ko'pirtirilgan sutdan tayyorlangan qaymoqli qahva.",
        "image": "https://images.unsplash.com/photo-1572442388796-11668a67e53d?w=600"
    },
    "Sok": {
        "title": "🧃 Mevali Sok",
        "price": "15,000 so'm",
        "desc": "Tabiiy mevalardan tayyorlangan sovuq va vitaminli ichimlik.",
        "image": "https://images.unsplash.com/photo-1621506289937-a8e4df240d0b?w=600"
    }
}

# ==================== KLAVIATURALAR ====================

# 1. Asosiy menyu (4 ta tugma)
main_menu = ReplyKeyboardMarkup(
    keyboard=[
        [KeyboardButton(text="🍲 Taomlar"), KeyboardButton(text="🥤 Ichimliklar")],
        [KeyboardButton(text="📞 Bog'lanish"), KeyboardButton(text="ℹ️ Bot haqida")]
    ],
    resize_keyboard=True
)

# 2. Taomlar submenyusi
foods_menu = ReplyKeyboardMarkup(
    keyboard=[
        [KeyboardButton(text="Pizza"), KeyboardButton(text="Burger")],
        [KeyboardButton(text="Taco"), KeyboardButton(text="Lag'mon")],
        [KeyboardButton(text="🔙 Orqaga")]
    ],
    resize_keyboard=True
)

# 2. Ichimliklar submenyusi
drinks_menu = ReplyKeyboardMarkup(
    keyboard=[
        [KeyboardButton(text="Choy"), KeyboardButton(text="Kofe"), KeyboardButton(text="Sok")],
        [KeyboardButton(text="🔙 Orqaga")]
    ],
    resize_keyboard=True
)

# ==================== HANDLER'LAR ====================

# /start buyrug'i
@dp.message(CommandStart())
async def cmd_start(message: Message):
    await message.answer(
        "Assalomu alaykum! Restoranimiz botiga xush kelibsiz. Kerakli bo'limni tanlang:",
        reply_markup=main_menu
    )

# Asosiy menyu: Bog'lanish
@dp.message(F.text == "📞 Bog'lanish")
async def contact_handler(message: Message):
    text = (
        "📞 <b>Biz bilan bog'lanish:</b>\n\n"
        "• Telefon: <code>+998 90 123-45-67</code>\n"
        "• Manzil: Toshkent shahri, Amir Temur ko'chasi 1-uy\n"
        "• Ish vaqti: 09:00 - 23:00"
    )
    await message.answer(text, reply_markup=main_menu)

# Asosiy menyu: Bot haqida
@dp.message(F.text == "ℹ️ Bot haqida")
async def about_handler(message: Message):
    text = (
        "ℹ️ <b>Bot haqida:</b>\n\n"
        "Ushbu bot <i>aiogram 3.x</i> va <i>URLInputFile</i> texnologiyalari yordamida "
        "3 darajali menyu tizimini namoyish qilish uchun yaratildi."
    )
    await message.answer(text, reply_markup=main_menu)

# Asosiy menyu: Taomlar bo'limiga o'tish (2-daraja)
@dp.message(F.text == "🍲 Taomlar")
async def foods_section(message: Message):
    await message.answer("Marhamat, taom turini tanlang:", reply_markup=foods_menu)

# Asosiy menyu: Ichimliklar bo'limiga o'tish (2-daraja)
@dp.message(F.text == "🥤 Ichimliklar")
async def drinks_section(message: Message):
    await message.answer("Marhamat, ichimlik turini tanlang:", reply_markup=drinks_menu)

# 'Orqaga' tugmasi (Asosiy menyuga qaytish)
@dp.message(F.text == "🔙 Orqaga")
async def back_to_main(message: Message):
    await message.answer("Asosiy menyu:", reply_markup=main_menu)

# 3-daraja kontent: Mahsulotlarni tanlash (Pizza, Burger, Choy va h.k.)
@dp.message(F.text.in_(PRODUCTS.keys()))
async def product_detail_handler(message: Message):
    product_key = message.text
    item = PRODUCTS[product_key]
    
    caption = (
        f"<b>{item['title']}</b>\n\n"
        f"📝 <i>{item['desc']}</i>\n\n"
        f"💰 <b>Narxi:</b> <code>{item['price']}</code>"
    )
    
    # Unsplash URL orqali rasm yuborish
    photo = URLInputFile(item["image"])
    await message.answer_photo(photo=photo, caption=caption)

# Echo (catch-all) — Barcha noma'lum matnlar uchun eng oxirida turadi
@dp.message(F.text)
async def echo_handler(message: Message):
    await message.answer(
        "Iltimos, pastdagi menyu tugmalaridan foydalaning! 🔽",
        reply_markup=main_menu
    )

# ==================== MAIN ====================
async def main():
    await bot.delete_webhook(drop_pending_updates=True)
    print("Bot ishga tushdi...")
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
