# ════════════════════════════════════════════════════════════════════
# DARS 8: Fayllar bilan ishlash
# ════════════════════════════════════════════════════════════════════

import asyncio
import os
import logging
from pathlib import Path
from dotenv import load_dotenv

from aiogram import Bot, Dispatcher, F
from aiogram.client.default import DefaultBotProperties
from aiogram.enums import ParseMode
from aiogram.filters import Command, CommandStart
from aiogram.types import (
    Message,
    FSInputFile, URLInputFile, BufferedInputFile,
    InputMediaPhoto,
)
from aiogram.utils.media_group import MediaGroupBuilder

load_dotenv()
logging.basicConfig(level=logging.INFO)

bot = Bot(
    token=os.getenv("BOT_TOKEN"),
    default=DefaultBotProperties(parse_mode=ParseMode.HTML),
)
dp = Dispatcher()

UPLOAD_DIR = Path("uploads")
UPLOAD_DIR.mkdir(exist_ok=True)


# file_id cache (production'da DB)
SAVED_FILE_IDS: dict[str, str] = {}


# ─────────────────────────────────────────────────────────────────────
# 1) Boshlash
# ─────────────────────────────────────────────────────────────────────

@dp.message(CommandStart())
async def cmd_start(m: Message):
    await m.answer(
        f"Salom, <b>{m.from_user.first_name}</b>!\n\n"
        f"Fayllar bilan ishlash bo'lim:\n"
        f"/photo — rasm yuborish\n"
        f"/doc — PDF yuborish\n"
        f"/album — 4 ta rasm albom\n"
        f"/voice — voice xabar\n\n"
        f"Yoki menga rasm/voice/hujjat yuboring — saqlayman."
    )


# ─────────────────────────────────────────────────────────────────────
# 2) Yuborish — 4 ta variant
# ─────────────────────────────────────────────────────────────────────

@dp.message(Command("photo"))
async def cmd_photo(m: Message):
    # 1) Lokal fayl (sekin — har safar upload)
    # await m.answer_photo(FSInputFile("static/cat.jpg"))

    # 2) URL'dan
    await m.answer_photo(
        URLInputFile("https://images.unsplash.com/photo-1514888286974-6c03e2ca1dba?w=600"),
        caption="🐱 Mushuk (Unsplash'dan)",
    )


@dp.message(Command("doc"))
async def cmd_doc(m: Message):
    # Buffer'dan (yangi yaratilgan)
    import io
    text = "Bu — bot tomonidan yaratilgan PDF mazmuni.\n\n"
    text += "Production'da reportlab yoki weasyprint bilan."
    buf = io.BytesIO(text.encode())

    await m.answer_document(
        BufferedInputFile(buf.getvalue(), filename="hisobot.txt"),
        caption="📄 Hisobot tayyor",
    )


@dp.message(Command("album"))
async def cmd_album(m: Message):
    urls = [
        "https://images.unsplash.com/photo-1514888286974-6c03e2ca1dba?w=600",
        "https://images.unsplash.com/photo-1517849845537-4d257902454a?w=600",
        "https://images.unsplash.com/photo-1543852786-1cf6624b9987?w=600",
        "https://images.unsplash.com/photo-1592194996308-7b43878e84a6?w=600",
    ]
    builder = MediaGroupBuilder(caption="📸 Mushuklar albomi")
    for i, url in enumerate(urls):
        builder.add_photo(URLInputFile(url))
    await bot.send_media_group(m.chat.id, builder.build())


@dp.message(Command("voice"))
async def cmd_voice(m: Message):
    # Demo — agar voice fayl bo'lsa
    voice_path = Path("static/voice.ogg")
    if voice_path.exists():
        await m.answer_voice(FSInputFile(voice_path))
    else:
        await m.answer("voice.ogg topilmadi. Faqat demo.")


# ─────────────────────────────────────────────────────────────────────
# 3) Qabul qilish — rasm
# ─────────────────────────────────────────────────────────────────────

@dp.message(F.photo)
async def get_photo(m: Message):
    photo = m.photo[-1]    # eng katta versiyasi

    # Telegram'dan file_path
    file = await bot.get_file(photo.file_id)

    # Local'ga saqlash
    local_path = UPLOAD_DIR / f"{m.from_user.id}_{photo.file_unique_id}.jpg"
    await bot.download_file(file.file_path, str(local_path))

    # file_id ni saqlash (qayta yuborish uchun)
    SAVED_FILE_IDS[f"user_{m.from_user.id}_last_photo"] = photo.file_id

    await m.answer(
        f"📷 Rasm saqlandi!\n"
        f"Eni × bo'yi: {photo.width} × {photo.height}\n"
        f"Hajmi: {photo.file_size:,} bayt\n"
        f"Yo'li: <code>{local_path}</code>\n"
        f"file_id: <code>{photo.file_id[:30]}...</code>"
    )


# ─────────────────────────────────────────────────────────────────────
# 4) Qabul qilish — hujjat
# ─────────────────────────────────────────────────────────────────────

@dp.message(F.document)
async def get_doc(m: Message):
    doc = m.document

    # Hajm tekshirish
    MAX_MB = 10
    if doc.file_size > MAX_MB * 1024 * 1024:
        await m.answer(f"❌ Fayl juda katta (max {MAX_MB} MB)")
        return

    # Mime tekshirish
    ALLOWED = {"application/pdf", "image/png", "image/jpeg", "text/plain"}
    if doc.mime_type not in ALLOWED:
        await m.answer(f"❌ Bu turdagi fayl qabul qilinmaydi: {doc.mime_type}")
        return

    file = await bot.get_file(doc.file_id)
    local_path = UPLOAD_DIR / f"{m.from_user.id}_{doc.file_name}"
    await bot.download_file(file.file_path, str(local_path))

    await m.answer(
        f"📄 Hujjat saqlandi!\n"
        f"Nomi: <b>{doc.file_name}</b>\n"
        f"Tur: {doc.mime_type}\n"
        f"Hajmi: {doc.file_size:,} bayt"
    )


# ─────────────────────────────────────────────────────────────────────
# 5) Qabul qilish — voice
# ─────────────────────────────────────────────────────────────────────

@dp.message(F.voice)
async def get_voice(m: Message):
    voice = m.voice
    if voice.duration > 60:
        await m.answer("❌ Voice 60 sekunddan ko'p — qisqaroq yuboring")
        return

    file = await bot.get_file(voice.file_id)
    local_path = UPLOAD_DIR / f"{m.from_user.id}_voice.ogg"
    await bot.download_file(file.file_path, str(local_path))

    await m.answer(
        f"🎤 Voice saqlandi!\n"
        f"Davomiyligi: {voice.duration}s\n"
        f"Hajmi: {voice.file_size:,} bayt\n\n"
        f"<i>Whisper bilan matn'ga aylantirish mumkin (bonus).</i>"
    )

    # Bonus: voice'ni qaytarib yuborish
    await m.answer_voice(FSInputFile(local_path), caption="🔄 Sizning voice'ingiz")


# ─────────────────────────────────────────────────────────────────────
# 6) PDF/PNG specific
# ─────────────────────────────────────────────────────────────────────

@dp.message(F.document.mime_type == "application/pdf")
async def get_pdf(m: Message):
    # PDF — alohida handler
    doc = m.document
    await m.answer(f"📕 PDF qabul qilindi: {doc.file_name}")
    # Davom — yuqoridagi F.document handler ham ishlamaydi (filter aniqroq)


# ─────────────────────────────────────────────────────────────────────
# 7) Sticker
# ─────────────────────────────────────────────────────────────────────

@dp.message(F.sticker)
async def get_sticker(m: Message):
    s = m.sticker
    await m.answer(
        f"😄 Sticker!\n"
        f"Emoji: {s.emoji}\n"
        f"Set: {s.set_name or '—'}\n"
        f"Animatsiya: {'ha' if s.is_animated else 'yo\'q'}\n"
        f"Video: {'ha' if s.is_video else 'yo\'q'}\n"
        f"file_id: <code>{s.file_id[:30]}...</code>"
    )
    # Sticker'ni qaytarish
    await m.answer_sticker(s.file_id)


# ─────────────────────────────────────────────────────────────────────
# 8) Location
# ─────────────────────────────────────────────────────────────────────

@dp.message(F.location)
async def get_location(m: Message):
    loc = m.location
    await m.answer(
        f"📍 Joylashuv:\n"
        f"Lat: <code>{loc.latitude}</code>\n"
        f"Lon: <code>{loc.longitude}</code>\n\n"
        f"Google Maps: https://maps.google.com/?q={loc.latitude},{loc.longitude}"
    )

    # Bot ham joylashuv yuboradi (Toshkent markazi)
    await m.answer_location(latitude=41.3111, longitude=69.2797)


# ─────────────────────────────────────────────────────────────────────
async def main():
    await bot.delete_webhook(drop_pending_updates=True)
    await dp.start_polling(bot)


if __name__ == "__main__":
    asyncio.run(main())
