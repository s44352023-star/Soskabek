🛠 1. Loyihani sozlash va ishga tushirish
Terminalda quyidagi buyruqlarni bajarib virtual muhit yaratamiz va kutubxonalarni o'rnatamiz:

Bash
# Virtual muhit yaratish
python -m venv venv

# Virtual muhitni yoqish (Windows uchun)
venv\Scripts\activate

# Virtual muhitni yoqish (Mac/Linux uchun)
source venv/bin/activate

# Kerakli kutubxonalarni o'rnatish
pip install aiogram python-dotenv
📁 2. Fayllar strukturasi
Loyiha papkasida quyidagi fayllarni hosil qiling:

Plaintext
my_telegram_bot/
│
├── venv/
├── .env
├── .gitignore
└── bot.py
⚙️ 3. Konfiguratsiya fayllari
.env
Bot Father'dan olgan maxfiy tokenni shu yerga yozasiz:

Фрагмент кода
BOT_TOKEN=123456789:ABCdefGhIJKlmNoPQRsTUVwxyZ
.gitignore
Maxfiy tokenli .env fayli va virtual muhit Git'ga tushib ketmasligi uchun:

Plaintext
venv/
.env
__pycache__/
💻 4. Bot kodi (bot.py)
Quyidagi kodni bot.py fayliga yozing. Bu yerda siz so'ragan barcha handlerlar, HTML formatlash va DefaultBotProperties sozlamalari to'liq kiritilgan:

Python
import asyncio
import logging
import os
from aiogram import Bot, Dispatcher, F, html
from aiogram.client.default import DefaultBotProperties
from aiogram.enums import ParseMode
from aiogram.filters import Command, CommandStart
from aiogram.types import Message
from dotenv import load_dotenv

# .env faylidan o'zgaruvchilarni o'qish
load_dotenv()
BOT_TOKEN = os.getenv("BOT_TOKEN")

# Logging'ni sozlash
logging.basicConfig(level=logging.INFO)

# Bot va Dispatcher obyektlarini yaratish
# ParseMode.HTML ni default tarzda o'rnatish
bot = Bot(
    token=BOT_TOKEN, 
    default=DefaultBotProperties(parse_mode=ParseMode.HTML)
)
dp = Dispatcher()


# /start handler'i
@dp.message(CommandStart())
async def cmd_start(message: Message):
    user_name = html.quote(message.from_user.first_name)
    text = (
        f"Salom, <b>{user_name}</b>! 👋\n\n"
        f"Men <i>aiogram 3.x</i> yordamida yaratilgan botman.\n"
        f"Mening imkoniyatlarimni ko'rish uchun /help buyrug'ini bosing."
    )
    await message.answer(text)


# /help handler'i
@dp.message(Command("help"))
async def cmd_help(message: Message):
    text = (
        "<b>Mavjud buyruqlar:</b>\n"
        "• /start - Botni ishga tushirish\n"
        "• /help - Yordam olish\n"
        "• /info - Bot haqida ma'lumot\n"
        "• /id - Sizning Telegram ID'ingiz\n\n"
        "<i>Har qanday xabar yuborsangiz, uni Echo qilaman!</i>"
    )
    await message.answer(text)


# /info handler'i (Bot.get_me() ishlatilgan)
@dp.message(Command("info"))
async def cmd_info(message: Message):
    bot_info = await bot.get_me()
    text = (
        "🤖 <b>Bot Ma'lumotlari:</b>\n"
        f"• Nomi: <code>{html.quote(bot_info.full_name)}</code>\n"
        f"• Username: @{bot_info.username}\n"
        f"• ID: <code>{bot_info.id}</code>"
    )
    await message.answer(text)


# /id handler'i
@dp.message(Command("id"))
async def cmd_id(message: Message):
    user_id = message.from_user.id
    chat_id = message.chat.id
    text = (
        f"👤 Sizning ID: <code>{user_id}</code>\n"
        f"💬 Chat ID: <code>{chat_id}</code>"
    )
    await message.answer(text)


# Echo handler (Boshqa barcha matnli xabarlar uchun)
@dp.message(F.text)
async def echo_handler(message: Message):
    try:
        # Foydalanuvchi yuborgan matnni qaytarish
        await message.send_copy(chat=message.chat.id)
    except TypeError:
        await message.answer("Faqat matnli xabarlarni qaytara olaman!")


# Asosiy funksiya
async def main():
    # Bot ishga tushganda eski update'larni o'tkazib yuborish
    await bot.delete_webhook(drop_pending_updates=True)
    
    bot_info = await bot.get_me()
    print(f"@{bot_info.username} muvaffaqiyatli ishga tushdi!")
    
    # Polling'ni boshlash
    await dp.start_polling(bot)


if __name__ == "__main__":
    asyncio.run(main())
🚀 5. Botni ishga tushirish
Terminalda quyidagi buyruqni yozing:

Bash
python bot.py
