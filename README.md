1. Backend Qismini Yangilash (CORS sozlamasi)
Backend (server.js) fayliga cors middleware'ini qo'shamiz va .env orqali API manzillarini boshqaramiz.

.env (Backend)
Фрагмент кода
PORT=5000
DATABASE_URL=postgresql://postgres:password@localhost:5432/taskflow
FRONTEND_URL=http://localhost:3000
server.js ga qo'shiladigan o'zgarishlar:
JavaScript
const express = require('express');
const cors = require('cors'); // <-- CORS ni chaqiramiz
const pool = require('./db');
require('dotenv').config();

const app = express();

// CORS ni to'g'ri sozlash (Faqat ruxsat etilganfrontend URL'ga ruxsat berish)
const corsOptions = {
  origin: process.env.FRONTEND_URL || 'http://localhost:3000',
  credentials: true,
};

app.use(cors(corsOptions));
app.use(express.json());

// ... (qolgan CRUD marshrutlar o'zgarishsiz qoladi)
2. Frontend Fayllari Strukturasi (React + Redux Toolkit)
Plaintext
frontend/
├── .env
├── package.json
└── src/
    ├── app/
    │   └── store.js
    ├── features/
    │   └── tasksSlice.js
    ├── components/
    │   └── TaskList.jsx
    ├── App.js
    └── index.js
3. Frontend Konfiguratsiyalari va Kodlari
.env (Frontend - React loyihasining ildiz qismida)
Eslatma: React boshlang'ich muhitiga qarab (Create React App yoki Vite) o'zgaruvchi nomi farq qilishi mumkin. Create React App uchun REACT_APP_ prefiksi ishlatiladi.

Фрагмент кода
REACT_APP_API_URL=http://localhost:5000
src/app/store.js (Redux Store)
JavaScript
import { configureStore } from '@reduxjs/toolkit';
import tasksReducer from '../features/tasksSlice';

export const store = configureStore({
  reducer: {
    tasks: tasksReducer,
  },
});
src/features/tasksSlice.js (createAsyncThunk bilan tasksSlice)
JavaScript
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// API manzilini .env fayldan olish (kodga qattiq yozilmagan)
const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:5000';

// createAsyncThunk orqali GET /tasks so'rovini bajarish
export const tasklarniOlish = createAsyncThunk(
  'tasks/tasklarniOlish',
  async (_, { rejectWithValue }) => {
    try {
      const response = await fetch(`${API_URL}/tasks`);
      if (!response.ok) {
        throw new Error('Serverdan maʼlumotlarni olishda xatolik yuz berdi!');
      }
      const data = await response.json();
      return data;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

const tasksSlice = createSlice({
  name: 'tasks',
  initialState: {
    list: [],
    loading: false,
    error: null,
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      // Pending holati (yuklanmoqda)
      .cases([tasklarniOlish.pending], (state) => {
        state.loading = true;
        state.error = null;
      })
      // Fulfilled holati (muvaffaqiyatli yakunlandi)
      .addCase(tasklarniOlish.fulfilled, (state, action) => {
        state.loading = false;
        state.list = action.payload;
      })
      // Rejected holati (xato yuz berdi)
      .addCase(tasklarniOlish.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  },
});

export default tasksSlice.reducer;
src/components/TaskList.jsx (Tasklar ro'yxati komponenti)
JavaScript
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { tasklarniOlish } from '../features/tasksSlice';

const TaskList = () => {
  const dispatch = useDispatch();
  const { list: tasks, loading, error } = useSelector((state) => state.tasks);

  useEffect(() => {
    dispatch(tasklarniOlish());
  }, [dispatch]);

  if (loading) {
    return <div style={{ textAlign: 'center', padding: '20px' }}>⏳ Yuklanmoqda...</div>;
  }

  if (error) {
    return <div style={{ color: 'red', textAlign: 'center', padding: '20px' }}>❌ Xatolik: {error}</div>;
  }

  return (
    <div style={{ maxWidth: '600px', margin: '20px auto', fontFamily: 'Arial, sans-serif' }}>
      <h2>📋 Vazifalar Ro'yxati</h2>
      {tasks.length === 0 ? (
        <p>Hozircha vazifalar mavjud emas.</p>
      ) : (
        <ul style={{ listStyleType: 'none', padding: 0 }}>
          {tasks.map((task) => (
            <li
              key={task.id}
              style={{
                background: '#f9f9f9',
                border: '1px solid #ddd',
                padding: '12px 16px',
                marginBottom: '10px',
                borderRadius: '6px',
                display: 'flex',
                justifyContent: 'space-between',
                alignItems: 'center',
              }}
            >
              <div>
                <h4 style={{ margin: '0 0 5px 0', textDecoration: task.is_completed ? 'line-through' : 'none' }}>
                  {task.title}
                </h4>
                <p style={{ margin: 0, fontSize: '14px', color: '#666' }}>{task.description}</p>
              </div>
              <div style={{ textAlign: 'right' }}>
                {/* JOIN orqali kelgan category_nomi */}
                <span
                  style={{
                    background: '#e0e0e0',
                    padding: '4px 8px',
                    borderRadius: '4px',
                    fontSize: '12px',
                    fontWeight: 'bold',
                  }}
                >
                  {task.category_nomi || 'Kategoriyasiz'}
                </span>
                <div style={{ fontSize: '11px', marginTop: '5px', color: task.is_completed ? 'green' : 'orange' }}>
                  {task.is_completed ? 'Bajarilgan' : 'Bajarilmoqda'}
                </div>
              </div>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
};

export default TaskList;
src/App.js
JavaScript
import React from 'react';
import TaskList from './components/TaskList';

function App() {
  return (
    <div className="App">
      <header style={{ textAlign: 'center', padding: '20px', background: '#282c34', color: 'white' }}>
        <h1>TaskFlow — Boshqaruv Paneli</h1>
      </header>
      <main>
        <TaskList />
      </main>
    </div>
  );
}

export default App;
4. README.md (Holat Checklist'i Yangilanishi)
Markdown
# TaskFlow Loyihasi Holat Checklist'i

- [x] Backend API: `schema.sql`, `users`, `categories`, `tasks` jadvallari yaratildi
- [x] Backend API: Express va `pg` yordamida CRUD amallari bajarildi (`GET /tasks` da `category_nomi` qo'shildi)
- [x] Backend API: `DELETE /categories/:id` da bog'liq tasklar bor-yo'qligi tekshirilib, `400` xato qaytarish mexanizmi qo'shildi
- [x] Backend API: `cors` middleware to'g'ri origin bilan sozlandi
- [x] Frontend: `.env` orqali `REACT_APP_API_URL` manzili sozlandi
- [x] Frontend: Redux Toolkit va `createAsyncThunk` yordamida `tasksSlice` yaratildi (pending, fulfilled, rejected holatlari boshqarildi)
- [x] Frontend: Komponentlar orqali tasklar ro'yxati `category_nomi` bilan birgalikda ekr
