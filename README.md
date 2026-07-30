import asyncio
import logging
import os
from aiogram import Bot, Dispatcher, F, html
from aiogram.client.default import DefaultBotProperties
from aiogram.enums import ParseMode
from aiogram.filters import Command, CommandStart
from aiogram.types import (
    Message,
    ReplyKeyboardMarkup,
    KeyboardButton,
    FSInputFile,
    URLInputFile,
    BufferedInputFile
)
from aiogram.utils.media_group import MediaGroupBuilder
from dotenv import load_dotenv

load_dotenv()
BOT_TOKEN = os.getenv("BOT_TOKEN")

logging.basicConfig(level=logging.INFO)

bot = Bot(token=BOT_TOKEN, default=DefaultBotProperties(parse_mode=ParseMode.HTML))
dp = Dispatcher()

# Fayllarni saqlash uchun papka
UPLOAD_DIR = "uploads"
os.makedirs(UPLOAD_DIR, exist_ok=True)

# 🔄 file_id kesh patterni (Takroriy yuklanishni oldini olish uchun)
FILE_ID_CACHE = {}

# Asosiy menyu
menu_kb = ReplyKeyboardMarkup(
    keyboard=[
        [KeyboardButton(text="/photo"), KeyboardButton(text="/doc")],
        [KeyboardButton(text="/album"), KeyboardButton(text="/voice")],
        [KeyboardButton(text="/id"), KeyboardButton(text="/help")]
    ],
    resize_keyboard=True
)

@dp.message(CommandStart())
async def cmd_start(message: Message):
    await message.answer(
        "Salom! Media va fayllar bilan ishlovchi botga xush kelibsiz. Quyidagi buyruqlardan foydalaning:",
        reply_markup=menu_kb
    )

@dp.message(Command("help"))
async def cmd_help(message: Message):
    text = (
        "<b>Buyruqlar ro'yxati:</b>\n"
        "• /photo - FSInputFile orqali lokal rasm yuborish\n"
        "• /doc - URLInputFile orqali hujjat yuborish\n"
        "• /album - MediaGroupBuilder yordamida albom yuborish\n"
        "• /voice - BufferedInputFile orqali ovozli xabar yuborish\n\n"
        "<i>Shuningdek, botga rasm, fayl, ovoz, stiker yoki lokatsiya yuborib sinashingiz mumkin!</i>"
    )
    await message.answer(text)

# ==================== 4 XIL INPUT FILE VARIANTRLARI ====================

# 1. FSInputFile (Lokal faylni yuborish)
@dp.message(Command("photo"))
async def send_local_photo(message: Message):
    # Sinov uchun vaqtinchalik rasm yaratish yoki mavjud fayl ko'rsatish
    photo_path = os.path.join(UPLOAD_DIR, "sample.jpg")
    
    # Agar fayl bo'lmasa, URL'dan yuklab saqlab qo'yamiz (namuna uchun)
    if not os.path.exists(photo_path):
        url_file = URLInputFile("https://images.unsplash.com/photo-1579353977828-2a4eab540d9f?w=600")
        with open(photo_path, "wb") as f:
            f.write(await bot.download(url_file))

    photo = FSInputFile(photo_path)
    await message.answer_photo(
        photo=photo,
        caption="<b>FSInputFile</b> orqali yuborilgan lokal rasm 🖼",
        reply_markup=menu_kb
    )

# 2. URLInputFile (Internetdagi havoladan yuborish)
@dp.message(Command("doc"))
async def send_url_document(message: Message):
    doc = URLInputFile("https://www.w3.org/WAI/ER/tests/xhtml/testfiles/resources/pdf/dummy.pdf", filename="sample.pdf")
    await message.answer_document(
        document=doc,
        caption="<b>URLInputFile</b> orqali yuborilgan hujjat 📄"
    )

# 4. BufferedInputFile (Xotiradagi baytlardan yuborish)
@dp.message(Command("voice"))
async def send_buffered_voice(message: Message):
    # Oddiy matnni baytlarga o'tkazib ovozli xabar sifatida yuborish simulyatsiyasi
    byte_data = b"Ovozli xabar baytlari..."
    voice = BufferedInputFile(byte_data, filename="voice.ogg")
    await message.answer_voice(voice=voice, caption="<b>BufferedInputFile</b> orqali yuborilgan ovoz")


# ==================== MEDIA GROUP (ALBUM) ====================
@dp.message(Command("album"))
async def send_media_album(message: Message):
    album_builder = MediaGroupBuilder(
        caption="<b>MediaGroupBuilder</b> yuborgan 4+ ta rasm albomi 📸"
    )
    album_builder.add(
        type="photo",
        media=URLInputFile("https://images.unsplash.com/photo-1506744038136-46273834b3fb?w=600")
    )
    album_builder.add(
        type="photo",
        media=URLInputFile("https://images.unsplash.com/photo-1470071459604-3b5ec3a7fe05?w=600")
    )
    album_builder.add(
        type="photo",
        media=URLInputFile("https://images.unsplash.com/photo-1426604966848-d7adacbd02bff?w=600")
    )
    album_builder.add(
        type="photo",
        media=URLInputFile("https://images.unsplash.com/photo-1511884642898-4c92249e20b6?w=600")
    )
    
    await message.answer_media_group(media=album_builder.build())


# ==================== HANDLER'LAR VA FILTRLAR ====================

# F.photo — Rasm qabul qilish, bot.get_file + download_file va file_id kesh
@dp.message(F.photo)
async def handle_photo(message: Message):
    # Eng sifatli rasmni olish (oxirgisi)
    photo = message.photo[-1]
    file_id = photo.file_id
    
    # 🔄 file_id cache pattern tushuntirishi:
    # Agar bu rasm oldin kelgan bo'lsa, uni serverdan qayta yuklab o'tirmaymiz,
    # to'g'ridan-to'g'ri keshdan foydalanamiz yoki tezkor javob beramiz.
    if file_id in FILE_ID_CACHE:
        print(f"Keshdan olindi: {file_id}")
    else:
        FILE_ID_CACHE[file_id] = "saved_in_memory"
        print(f"Yangi file_id keshga qo'shildi: {file_id}")

    # Faylni telegram serveridan yuklab olish
    file_info = await bot.get_file(file_id)
    file_path = file_info.file_path
    
    destination = os.path.join(UPLOAD_DIR, f"{photo.file_unique_id}.jpg")
    await bot.download_file(file_path, destination)
    
    await message.answer(
        f"Rasm qabul qilindi va <code>uploads/</code> papkasiga saqlandi! ✅\n"
        f"<b>file_id:</b> <code>{file_id}</code>"
    )

# F.document — Hajm va MIME turiga ko'ra filtr
@dp.message(F.document & (F.document.file_size < 10 * 1024 * 1024)) # 10 MB dan kichik
async def handle_document(message: Message):
    doc = message.document
    await message.answer(
        f"Hujjat qabul qilindi 📂\n"
        f"• Nomi: <code>{html.quote(doc.file_name)}</code>\n"
        f"• Hajmi: {doc.file_size} bayt\n"
        f"• MIME: {doc.mime_type}"
    )

# F.voice — Davomiyligi cheklangan ovozli xabar (masalan, 60 sekunddan kam)
@dp.message(F.voice & (F.voice.duration <= 60))
async def handle_voice(message: Message):
    await message.answer(f"Ovozli xabar qabul qilindi! 🎤 Davomiyligi: {message.voice.duration} soniya.")

# F.sticker — Stikerni qaytarish (Echo stiker)
@dp.message(F.sticker)
async def handle_sticker(message: Message):
    await message.answer_sticker(sticker=message.sticker.file_id)

# F.location — Joylashuvni qabul qilib Google Maps havolasini berish
@dp.message(F.location)
async def handle_location(message: Message):
    lat = message.location.latitude
    lon = message.location.longitude
    maps_url = f"https://maps.google.com/?q={lat},{lon}"
    
    await message.answer(
        f"📍 <b>Sizning lokatsiyangiz:</b>\n"
        f"• Kenglik (Latitude): <code>{lat}</code>\n"
        f"• Uzunlik (Longitude): <code>{lon}</code>\n\n"
        f"🗺 <a href='{maps_url}'>Google Maps'da ochish</a>"
    )

# ==================== MAIN ====================
async def main():
    await bot.delete_webhook(drop_pending_updates=True)
    print("Media bot ishga tushdi...")
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
