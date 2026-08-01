import os
import asyncio
from aiogram import Bot, Dispatcher, F
from aiogram.types import (
    Message,
    FSInputFile,
    URLInputFile,
    BufferedInputFile,
    ReplyKeyboardMarkup,
    KeyboardButton
)
from aiogram.utils.media_group import MediaGroupBuilder

TOKEN = "BOT_TOKENINGIZ"

bot = Bot(token=TOKEN)
dp = Dispatcher()

# Uploads papkasini yaratish
UPLOAD_DIR = "uploads"
os.makedirs(UPLOAD_DIR, exist_ok=True)

# Oddiy xotira (Cache) namunasi file_id uchun
file_cache = {}


# ==================== 1. FAYLLARNI YUBORISH (4 VARIANT) ====================
@dp.message(F.text == "/photo")
async def send_photo_variants(message: Message):
    # 1. URLInputFile orqali yuborish
    url_file = URLInputFile("https://picsum.photos/400/300")
    
    # caption va reply_markup qo'shib yuborish
    keyboard = ReplyKeyboardMarkup(
        keyboard=[[KeyboardButton(text="Zo'r rasm!")]],
        resize_keyboard=True
    )
    
    await message.answer_photo(
        photo=url_file,
        caption="Bu **URLInputFile** orqali yuborilgan rasm!",
        reply_markup=keyboard
    )
    
    # 2. FSInputFile (Agar serverda rasm bo'lsa: FSInputFile("uploads/image.jpg"))
    # 3. BufferedInputFile (Baytlardan: BufferedInputFile(b"data", filename="test.txt"))
    # 4. file_id orqali (Oldin olingan file_id ni kiritasiz):
    # await message.answer_photo(photo="AgACAgIA...")


@dp.message(F.text == "/doc")
async def send_document_example(message: Message):
    # FSInputFile yordamida hujjat yuborish
    doc_path = "uploads/sample.txt"
    with open(doc_path, "w") as f:
        f.write("Salom, bu test hujjat!")
        
    file = FSInputFile(doc_path, filename="maxsus_hujjat.txt")
    await message.answer_document(document=file, caption="Hujjat biriktirildi.")


@dp.message(F.text == "/voice")
async def send_voice_example(message: Message):
    # URL orqali ovozli xabar yuborish misoli
    voice = URLInputFile("https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3")
    await message.answer_voice(voice=voice, caption="Audio/Voice namuna")


@dp.message(F.text == "/album")
async def send_media_album(message: Message):
    # MediaGroupBuilder yordamida albom (4+ rasm) tayyorlash
    album = MediaGroupBuilder(caption="Bu MediaGroupBuilder yordamida yuborilgan 4 ta rasm albomi.")
    
    album.add_photo(media=URLInputFile("https://picsum.photos/id/1/300/200"))
    album.add_photo(media=URLInputFile("https://picsum.photos/id/2/300/200"))
    album.add_photo(media=URLInputFile("https://picsum.photos/id/3/300/200"))
    album.add_photo(media=URLInputFile("https://picsum.photos/id/4/300/200"))
    
    await message.answer_media_group(media=album.build())


# ==================== 2. FAYLLARNI QABUL QILISH VA FILTRLASH ====================

# Rasm qabul qilish, yuklab olish va lokalga saqlash
@dp.message(F.photo)
async def download_photo_handler(message: Message):
    # Eng sifatli rasmni olish (photo ro'yxatining oxirgisi eng kattasi)
    photo = message.photo[-1]
    file_id = photo.file_id
    
    # file_id ni keshga saqlash (Cache pattern)
    file_cache["last_photo"] = file_id
    
    # Telegram serveridan fayl yo'lini olish
    file_info = await bot.get_file(file_id)
    file_path = file_info.file_path
    
    # Lokal papkaga yuklab olish
    destination = os.path.join(UPLOAD_DIR, f"{photo.file_unique_id}.jpg")
    await bot.download_file(file_path, destination)
    
    await message.reply(
        f"Rasm muvaffaqiyatli saqlandi!\n"
        f"Lokal manzil: `{destination}`\n"
        f"File ID keshga yozildi: `{file_id[:10]}...`"
    )


# Hujjatlar uchun hajm (size) va MIME turiga ko'ra filter
@dp.message(F.document, lambda msg: msg.document.file_size < 5 * 1024 * 1024 and msg.document.mime_type == "application/pdf")
async def document_filter_handler(message: Message):
    await message.reply("PDF hujjat qabul qilindi va hajmi 5MB dan kichik.")


@dp.message(F.document)
async def document_rejected_handler(message: Message):
    await message.reply("Faqat 5MB dan kichik bo'lgan PDF formatidagi hujjatlar qabul qilinadi!")


# Voice (Ovozli xabar) uchun davomiylik (duration) limiti
@dp.message(F.voice, F.voice.duration <= 60)
async def voice_short_handler(message: Message):
    await message.reply(f"Ovozli xabar qabul qilindi. Davomiyligi: {message.voice.duration} soniya.")


@dp.message(F.voice)
async def voice_long_handler(message: Message):
    await message.reply("Ovozli xabar juda uzun! 1 daqiqadan qisqa xabar yuboring.")


# Sticker qabul qilish va uni qaytarish (echo)
@dp.message(F.sticker)
async def sticker_echo_handler(message: Message):
    await message.answer_sticker(sticker=message.sticker.file_id)


# Location qabul qilish va Google Maps havolasini qaytarish
@dp.message(F.location)
async def location_handler(message: Message):
    lat = message.location.latitude
    lon = message.location.longitude
    google_maps_url = f"https://www.google.com/maps?q={lat},{lon}"
    
    await message.reply(
        f"Sizning manzilingiz:\n"
        f"Kenglik (Lat): {lat}\n"
        f"Uzunlik (Lon): {lon}\n\n"
        f"[Google Maps'da ochish]({google_maps_url})",
        disable_web_page_preview=True
    )


async def main():
    print("Bot ishga tushdi...")
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
