1. Backend Sozlamalari (Render / Railway)
Production muhitida backend xavfsiz va to'g'ri ishlashi uchun CORS va Environment variables quyidagicha sozlanishi shart:

server.js (CORS'ni production frontend domeniga moslash)
JavaScript
const express = require('express');
const cors = require('cors');
const app = express();

// Ruxsat etilgan domenlar ro'yxati (Frontend qayerga deploy qilingan bo'lsa o'sha domen yoziladi)
const allowedOrigins = [
  'http://localhost:3000', // Local dev uchun
  'https://sening-frontend-domen.vercel.app' // Vercel'dagi production domeningiz
];

app.use(cors({
  origin: function (origin, callback) {
    // Agar origin yo'q bo'lsa (masalan Postman yoki server-to-server so'rovlar) yoki ruxsat etilganlar ro'yxatida bo'lsa
    if (!origin || allowedOrigins.indexOf(origin) !== -1) {
      callback(null, true);
    } else {
      callback(new Error('CORS siyosati bu domenga ruxsat bermaydi'));
    }
  },
  credentials: true
}));

app.use(express.json());

// ... qolgan endpoint'lar (register, login, tasks va h.k.)

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server ${PORT}-portda ishlamoqda`));
Hostingda (Render/Railway) o'rnatiladigan Environment Variables:
DATABASE_URL — PostgreSQL bazasining tashqi ulanish havolasi (External Connection String)

JWT_SECRET — Maxfiy kalit (string)

PORT — Render yoki Railway avtomatik beradi (odatda kiritish shart emas, lekin process.env.PORT ishlatilishi shart)

2. Frontend Sozlamalari (Vercel / Netlify)
Frontend qismida API bazaviy manzilini qattiq kodlash (hardcode) o'rniga Environment variable orqali olish kerak.

API chaqiruvlarini sozlash (src/api.js yoki config.js)
JavaScript
// Environment variable orqali backend manzilini olish
// Vercel uchun: REACT_APP_API_URL (React uchun) yoki VITE_API_URL (Vite uchun)
const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:5000';

export default API_URL;
So'rov yuborishda:

JavaScript
import API_URL from './api';

const response = await fetch(`${API_URL}/tasks`, {
  headers: {
    'Authorization': `Bearer ${token}`
  }
});
Hostingda (Vercel) o'rnatiladigan Environment Variable:
REACT_APP_API_URL (yoki Vite ishlatsangiz VITE_API_URL) = [https://sening-backend-domen.onrender.com](https://sening-backend-domen.onrender.com)

3. Yakuniy README.md Shabloni
GitHub repository'ingizning asosiy sahifasidagi README.md faylini quyidagi ko'rinishda yangilang:

Markdown
# TaskFlow — Full-Stack Task Management Application

TaskFlow bu foydalanuvchilarga o'z vazifalarini xavfsiz boshqarish, qidirish va filtrlash imkonini beruvchi zamonaviy veb-ilova. Loyiha to'liq to'liq (Full-Stack) tarzda ishlab chiqildi va production muhitiga deploy qilindi.

## 🚀 Jonli Havolalar (Live Demos)
- **Frontend (Vercel):** [https://taskflow-frontend.vercel.app](https://sening-frontend-domen.vercel.app)
- **Backend (Render):** [https://taskflow-backend.onrender.com](https://sening-backend-domen.onrender.com)

---

## 🛠 Seksiyalar va Texnologiyalar (Tech Stack)
* **Backend:** Node.js, Express.js, PostgreSQL, JWT (jsonwebtoken), bcrypt
* **Frontend:** React, Redux Toolkit, CSS / UI Components
* **Hosting:** 
  * Backend: Render
  * Frontend: Vercel
  * Database: PostgreSQL (Neon / Render)

---

## ✅ Loyiha Holat Checklist'i (6/6 Bosqich)
- [x] **1-bosqich:** Backend arxitekturasi va PostgreSQL baza ulanishi
- [x] **2-bosqich:** JWT autentifikatsiya, bcrypt parol xavfsizligi va himoyalangan route'lar
- [x] **3-bosqich:** Qidiruv (`ILIKE`), filtr (`category_id`) va sahifalash (`LIMIT / OFFSET`)
- [x] **4-bosqich:** Frontend qismi, Login/Register formalari va `localStorage` boshqaruvi
- [x] **5-bosqich:** Redux Toolkit yordamida global holat (state) va **400ms Debounce** qidiruv tizimi
- [x] **6-bosqich:** Production deploy (Render + Vercel), CORS to'g'ri sozlanishi va jonli havolalar integratsiyasi

---

## ⚙️ Mahalliy ishga tushirish (Local Installation)

Loyihani o'z kompyuteringizda ishga tushirish uchun:

1. Repozitoriyani klonlang:
   ```bash
   git clone [https://github.com/username/TaskFlow.git](https://github.com/username/TaskFlow.git)
Backend va Frontend papkalariga o'tib bog'liqliklarni o'rnating:

Bash
cd backend && npm install
cd ../frontend && npm install
.env fayllarini to'ldiring va serverlarni ishga tushiring.
