TaskFlow — 6 bosqichda to'liq stack loyiha
1-Loyihalash

2-Backend API

3-React frontend

4-Autentifikatsiya

5-Qidiruv/filtr

6-Deploy

Bu kursda siz React va Node.js/Express kurslarida alohida o'rgangan hamma narsani bitta haqiqiy loyihada birlashtirasiz: TaskFlow — jamoaviy vazifalar boshqaruvchisi. Har bir dars — shu bitta loyihaning navbatdagi bosqichi, va har bir bosqich haqiqiy loyiha topshirig'i sifatida (GitHub repo + tavsif orqali) baholanadi.

🏆 5 daqiqada g'alaba
BLOKA 1 — repo tuzilmasi: monorepo
# TaskFlow uchun bitta repo ichida ikkita papka - "monorepo" yondashuvi
taskflow/
  backend/          # Express + PostgreSQL (2-4-darsda quriladi)
    package.json
    server.js
  frontend/          # React + Redux Toolkit (3-darsda quriladi)
    package.json
    src/
  README.md          # loyiha tavsifi, ishga tushirish yo'riqnomasi
  .gitignore          # node_modules, .env kabi fayllarni chiqarib tashlaydi

# Nega monorepo? Kichik jamoaviy loyihalarda frontend va backend'ni
# BIR joyda ko'rish, versiyalashni sinxronlashtirish osonroq bo'ladi.
BLOKA 2 — DB sxemasini KOD YOZISHDAN OLDIN loyihalash
# TaskFlow uchun asosiy jadvallar (ER diagramma darajasida, hali SQL emas):
#
# users        (id, ism, email, parol_hash, yaratilgan_vaqt)
# categories   (id, nomi, user_id -> users.id)
# tasks        (id, sarlavha, matn, bajarilgan, category_id -> categories.id,
#               user_id -> users.id, yaratilgan_vaqt)
#
# Bog'lanishlar:
# - Bitta user -> ko'p categories (1 ga ko'p)
# - Bitta user -> ko'p tasks (1 ga ko'p)
# - Bitta category -> ko'p tasks (1 ga ko'p)

# Bu sxema 2-darsda haqiqiy PostgreSQL jadvallariga aylantiriladi.
BLOKA 3 — README.md: loyihaning "eshigi"
# README.md
# TaskFlow

## Loyiha haqida
Jamoaviy vazifalar boshqaruvchisi - React + Node/Express + PostgreSQL.

## O'rnatish
1. `cd backend && npm install`
2. `.env` faylini yarating (`.env.example`dan nusxa oling)
3. `npm run dev`

## Texnologiyalar
- Backend: Node.js, Express, PostgreSQL
- Frontend: React, Redux Toolkit

## Holat
- [x] Loyihalash va repo skeleton
- [ ] Backend API
- [ ] React frontend
- [ ] Autentifikatsiya
- [ ] Qidiruv va filtrlash
- [ ] Deploy
🐛 Ataylab qiyin: DB sxemasisiz to'g'ridan-to'g'ri kod yozishga urinish
Ko'p boshlang'ich dasturchilar DB sxemasini loyihalashni "keyinroq qilaman" deb, darhol Express route'lari yoki React componentlarini yoza boshlaydi. Bu quyidagi muammoga olib keladi:

// 2-darsda backend yozishni boshlaganingizda:
app.post('/tasks', async (req, res) => {
  // savol: task qaysi user'ga tegishli? category kerakmi?
  // Agar sxema oldindan aniq bo'lmasa, bu yerda IKKILANISH boshlanadi,
  // va keyinchalik jadval tuzilishini o'zgartirish (migratsiya) kerak bo'ladi
});
Natija: DB sxemasi (jadvallar, ustunlar, bog'lanishlar) oldindan aniq bo'lmasa, backend kodini yozish paytida doimiy "bu maydon kerakmi?", "bu qanday bog'lanadi?" kabi savollar tug'iladi — bu vaqtni behuda sarflaydi va ko'pincha keyinroq qayta migratsiya qilishga majbur qiladi. To'g'ri tartib: avval sxemani qog'ozda (yoki diagram sifatida) chizib, keyin kodni yozish.

Endi tushuntiramiz
1. Nega monorepo (bitta repo, ikkita papka) tanlandi?
Kichik, bitta dasturchi (yoki kichik jamoa) tomonidan qurilayotgan to'liq stack loyihalar uchun monorepo qulay: frontend va backend o'zgarishlarini bitta joyda kuzatish, versiyalashni sinxronlashtirish osonroq. Katta kompaniyalarda ko'pincha alohida repo'lar ishlatiladi, lekin bu boshqa masala.

2. Nega DB sxemasi eng birinchi loyihalanadi?
Deyarli hamma narsa — backend endpoint'lari, frontend'dagi ma'lumot shakli, autentifikatsiya — DB sxemasiga bog'liq. Sxema noaniq bo'lsa, keyingi bosqichlarning har birida qayta-qayta qaror qabul qilishga to'g'ri keladi. Sxemani oldindan loyihalash — keyingi bosqichlarni tezlashtiradi.

3. README.md nima uchun muhim?
README — loyihaning "eshigi": boshqa dasturchi (yoki baholovchi) loyihani birinchi marta ko'rganda, uni qanday ishga tushirish, qaysi texnologiyalar ishlatilgani va joriy holatni shu yerdan biladi. Bu 6-darsda deploy qilinganda ham juda muhim bo'ladi.

4. Bu kursda "topshiriq" oldingi kurslardan nima bilan farq qiladi?
Oldingi kurslarda har bir dars mustaqil mavzu edi. Bu yerda har bir dars bitta, davom etayotgan loyihaning keyingi bosqichi — siz har safar bir xil GitHub repo'ga (yangilangan holda) havola yuborasiz, va loyiha 6-darsning oxirida to'liq, deploy qilingan ilova bo'lishi kerak.

5. .gitignore nima uchun kerak?
.gitignore — node_modules (juda katta, qayta o'rnatish mumkin) va .env (maxfiy kalitlar) kabi fayllarni repo'ga tushmasligi uchun belgilaydi. Bularni repo'ga qo'shish — repo hajmini keraksiz kattalashtiradi va (agar .env bo'lsa) maxfiy ma'lumotlarni oshkor qilish xavfi tug'diradi.

📌 Bu darsdan keyin siz bilasizki
✅ Monorepo — kichik to'liq stack loyihalar uchun frontend+backend'ni bitta repo'da saqlash
✅ DB sxemasi kod yozishdan oldin loyihalanishi kerak
✅ README.md — loyihaning ishga tushirish yo'riqnomasi va joriy holatini ko'rsatadi
✅ Bu kursda har bir dars — bitta davom etayotgan loyihaning bosqichi, mustaqil mavzu emas
✅ .gitignore — node_modules/.env kabi fayllarni repo'dan chiqarib tashlaydi
💻
Код
Код
#2
javascript
 Копировать
// ════════════════════════════════════════════════════════════════════
// 1-BOSQICH: Loyihalash va repo skeleton
// ════════════════════════════════════════════════════════════════════

// Bu dars kod yozishdan ko'ra REJALASHTIRISHGA bag'ishlangan.
// Quyida - TaskFlow uchun DB sxemasining JavaScript obyekt shaklidagi
// "qog'ozdagi" tasviri (hali haqiqiy SQL/migratsiya emas - bu 2-darsda bo'ladi):

const dbSxemasi = {
  users: {
    id: 'SERIAL PRIMARY KEY',
    ism: 'VARCHAR(100)',
    email: 'VARCHAR(255) UNIQUE',
    parol_hash: 'VARCHAR(255)',
    yaratilgan_vaqt: 'TIMESTAMP DEFAULT NOW()',
  },
  categories: {
    id: 'SERIAL PRIMARY KEY',
    nomi: 'VARCHAR(100)',
    user_id: 'INTEGER REFERENCES users(id)',
  },
  tasks: {
    id: 'SERIAL PRIMARY KEY',
    sarlavha: 'VARCHAR(200)',
    matn: 'TEXT',
    bajarilgan: 'BOOLEAN DEFAULT false',
    category_id: 'INTEGER REFERENCES categories(id)',
    user_id: 'INTEGER REFERENCES users(id)',
    yaratilgan_vaqt: 'TIMESTAMP DEFAULT NOW()',
  },
};

console.log(dbSxemasi);

// ─────────────────────────────────────────────────────────────────────
// Repo tuzilmasi (izohda - papka/fayl tuzilmasi, kod emas)
// ─────────────────────────────────────────────────────────────────────

// taskflow/
//   backend/
//   frontend/
//   README.md
//   .gitignore
