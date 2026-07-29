🛠 1. Loyiha Arxitekturasi va Tuzilmasi
Frontend (React + Vite)
Loyihani quyidagi papkalar tuzilmasi asosida tashkil qilish kodni tartibli saqlashga yordam beradi:

Plaintext
src/
├── assets/          # Rasmlar, ikonka va fontlar
├── components/      # Umumiy komponentlar (Navbar, Footer, Modal, Button)
├── context/         # AuthContext, ThemeContext (Dark Mode)
├── hooks/           # useFetch, useDebounce, useLocalStorage
├── pages/           # 7+ sahifa (Home, Recipes, Detail, Favorites, Profile, Login, Register, NotFound)
├── router/          # ProtectedRoute va AppRouter
├── services/        # API so'rovlar uchun alohida modullar (axios yoki fetch wrapper)
├── utils/           # Yordamchi funksiyalar
└── App.jsx
Backend (Flask + SQLAlchemy)
Flask uchun Application Factory patternidan foydalanish eng to‘g‘ri yondashuv hisoblanadi:

Plaintext
backend/
├── app/
│   ├── __init__.py      # Flask app yaratish va CORS, DB ulash
│   ├── models.py        # SQLAlchemy modellar (User, Recipe, Favorite, Comment)
│   ├── routes/          # Blueprintlar (auth, recipes, favorites, comments)
│   └── utils.py         # JWT va yordamchi funksiyalar
├── requirements.txt
└── run.py               # Loyihani ishga tushiruvchi fayl
