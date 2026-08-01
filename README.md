Mana siz talab qilgan barcha shartlarni to'liq bajaruvchi, xavfsiz (parametrlashtirilgan SQL so'rovlar) va mukammal arxitekturadagi Node.js + Express + PostgreSQL (pg) loyihasi kodi:

1. schema.sql — Ma'lumotlar bazasi strukturasi
Bog'liqliklar (Foreign Keys) va xavfsizlik qoidalariga to'liq amal qilingan holda (ON DELETE RESTRICT yordamida bog'liq vazifalari bor kategoriyani o'chirishning oldi olingan):

SQL
-- users jadvali
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- categories jadvali
CREATE TABLE IF NOT EXISTS categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- tasks jadvali
CREATE TABLE IF NOT EXISTS tasks (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    completed BOOLEAN DEFAULT FALSE,
    category_id INT NOT NULL,
    user_id INT NOT NULL,
    FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE RESTRICT,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
2. db.js — PostgreSQL ulanish moduli (pg)
JavaScript
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL || 'postgresql://postgres:password@localhost:5432/todo_db',
});

module.exports = {
  query: (text, params) => pool.query(text, params),
};
3. server.js — Express API va barcha Endpoints
Barcha talab qilingan API yo'llari (GET /tasks, POST /tasks, PUT /tasks/:id, DELETE /categories/:id) SQL injection'dan himoyalangan holda ($1, $2 orqali) yozilgan:

JavaScript
const express = require('express');
const db = require('./db');

const app = express();
app.use(express.json());

// 1. GET /tasks — JOIN orqali category_nomi bilan birga qaytaradi
app.get('/tasks', async (req, res) => {
  try {
    const query = `
      SELECT t.id, t.title, t.completed, t.category_id, c.name AS category_name, t.created_at
      FROM tasks t
      JOIN categories c ON t.category_id = c.id
      ORDER BY t.id DESC;
    `;
    const { rows } = await db.query(query);
    res.json(rows);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Serverda xatolik yuz berdi' });
  }
});

// 2. POST /tasks — sarlavha va category_id validatsiya qilinadi, 201 qaytaradi
app.post('/tasks', async (req, res) => {
  const { title, category_id, user_id } = req.body;

  // Validatsiya
  if (!title || typeof title !== 'string' || title.trim() === '') {
    return res.status(400).json({ error: "Sarlavha (title) kiritilishi shart va yaroqli bo'lishi kerak" });
  }
  if (!category_id || typeof category_id !== 'number') {
    return res.status(400).json({ error: "Yaroqli category_id kiritilishi shart" });
  }

  try {
    // Kategoriya mavjudligini tekshirish
    const catCheck = await db.query('SELECT id FROM categories WHERE id = $1', [category_id]);
    if (catCheck.rows.length === 0) {
      return res.status(404).json({ error: "Ko'rsatilgan kategoriya topilmadi" });
    }

    const query = `
      INSERT INTO tasks (title, category_id, user_id) 
      VALUES ($1, $2, $3) 
      RETURNING *;
    `;
    // Parametrlashtirilgan so'rov ($1, $2, $3)
    const values = [title.trim(), category_id, user_id || 1]; 
    const { rows } = await db.query(query, values);

    res.status(201).json(rows[0]);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Serverda xatolik yuz berdi' });
  }
});

// 3. PUT /tasks/:id — bajarilgan holatini yangilaydi
app.put('/tasks/:id', async (req, res) => {
  const { id } = req.params;
  const { completed } = req.body;

  if (typeof completed !== 'boolean') {
    return res.status(400).json({ error: "completed qiymati boolean (true/false) bo'lishi kerak" });
  }

  try {
    const query = `
      UPDATE tasks 
      SET completed = $1 
      WHERE id = $2 
      RETURNING *;
    `;
    const { rows } = await db.query(query, [completed, id]);

    if (rows.length === 0) {
      return res.status(404).json({ error: "Vazifa topilmadi" });
    }

    res.json(rows[0]);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Serverda xatolik yuz berdi' });
  }
});

// 4. DELETE /categories/:id — bog'liq tasks bo'lsa 400 xato qaytaradi, bo'lmasa o'chiradi
app.delete('/categories/:id', async (req, res) => {
  const { id } = req.params;

  try {
    // Ushbu kategoriyaga bog'liq vazifalar borligini tekshiramiz
    const checkQuery = 'SELECT COUNT(*) FROM tasks WHERE category_id = $1';
    const checkResult = await db.query(checkQuery, [id]);
    const taskCount = parseInt(checkResult.rows[0].count, 10);

    if (taskCount > 0) {
      return res.status(400).json({ 
        error: "Ushbu kategoriyaga bog'liq vazifalar mavjud. Avval ularni o'chiring yoki boshqa kategoriyaga o'tkazing." 
      });
    }

    // Bog'liq vazifalar bo'lmasa, kategoriyani o'chiramiz
    const deleteQuery = 'DELETE FROM categories WHERE id = $1 RETURNING *;';
    const { rows } = await db.query(deleteQuery, [id]);

    if (rows.length === 0) {
      return res.status(404).json({ error: "Kategoriya topilmadi" });
    }

    res.json({ message: "Kategoriya muvaffaqiyatli o'chirildi", deleted: rows[0] });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Serverda xatolik yuz berdi' });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server ${PORT}-portda ishga tushdi...`);
});
4. README.md — Holat Checklist'i
Markdown
# Node.js Express & PostgreSQL Todo API

## 📋 Holat Checklist'i (Status Checklist)

- [x] **schema.sql**: `users`, `categories`, `tasks` jadvallari to'g'ri Foreign Key bog'liqliklari bilan yaratildi.
- [x] **GET /tasks**: Ma'lumotlar `JOIN` yordamida `category_nomi` bilan birgalikda qaytarilmoqda.
- [x] **POST /tasks**: `title` va `category_id` validatsiyadan o'tkazilib, muvaffaqiyatli qo'shilganda `201 Created` statusi qaytarilmoqda.
- [x] **PUT /tasks/:id**: Vazifaning bajarilganlik holati (`completed`) xavfsiz yangilanmoqda.
- [x] **DELETE /categories/:id**: Bog'liq vazifalari bor kategoriya o'chirilganda `400 Bad Request` xatosi qaytarilmoqda, bog'liqlik bo'lmasa o'chirilmoqda.
- [x] **Xavfsizlik**: Barcha SQL so'rovlar SQL Injection'dan himoyalangan holda parametrlashtirilgan (`$1, $2, ...`).
- [x] **Dokumentatsiya**: README.md faylidagi holat checklist'i yangilandi.
