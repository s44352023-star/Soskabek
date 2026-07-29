1. Backend Qismi (Node.js + Express)
package.json uchun kerakli kutubxonalar
JSON
{
  "dependencies": {
    "bcrypt": "^5.1.1",
    "cors": "^2.8.5",
    "dotenv": "^16.4.5",
    "express": "^4.19.2",
    "jsonwebtoken": "^9.0.2",
    "pg": "^8.11.3"
  }
}
server.js (Asosiy backend fayli)
JavaScript
require('dotenv').config();
const express = require('express');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const { Pool } = require('pg');
const cors = require('cors');

const app = express();
app.use(express.json());
app.use(cors());

// PostgreSQL ulanishi (Ma'lumotlar bazasi sozlamalari)
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
});

// JWT Sirli kaliti
const JWT_SECRET = process.env.JWT_SECRET || 'super_secret_key_123';

// ----------------------------------------------------
// MIDDLEWARE: Autentifikatsiyani talab qilish
// ----------------------------------------------------
const autentifikatsiyaTalabQilish = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401.json({ error: 'Token taqdim etilmadi yoki noto‘g‘ri formatda' });
  }

  const token = authHeader.split(' ')[1];

  try {
    const decoded = jwt.verify(token, JWT_SECRET);
    req.user = decoded; // { userId: ... }
    next();
  } catch (err) {
    return res.status(403.json({ error: 'Token yaroqsiz yoki muddati o‘tgan' });
  }
};

// ----------------------------------------------------
// ENDPOINTLAR
// ----------------------------------------------------

// 1. POST /register — Ro'yxatdan o'tish
app.post('/register', async (req, res) => {
  try {
    const { username, password } = req.body;
    
    if (!username || !password) {
      return res.status(400).json({ error: 'Username va parol kiritilishi shart' });
    }

    // Foydalanuvchi mavjudligini tekshirish
    const userCheck = await pool.query('SELECT * FROM users WHERE username = $1', [username]);
    if (userCheck.rows.length > 0) {
      return res.status(400).json({ error: 'Bu foydalanuvchi allaqachon mavjud' });
    }

    // Parolni bcrypt orqali hash qilish
    const saltRounds = 10;
    const hashedPassword = await bcrypt.hash(password, saltRounds);

    // Foydalanuvchini bazaga saqlash
    const newUser = await pool.query(
      'INSERT INTO users (username, password) VALUES ($1, $2) RETURNING id, username',
      [username, hashedPassword]
    );

    res.status(201).json({
      message: 'Muvaffaqiyatli ro‘yxatdan o‘tdingiz',
      user: newUser.rows[0],
    });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Server xatoligi' });
  }
});

// 2. POST /login — Tizimga kirish
app.post('/login', async (req, res) => {
  try {
    const { username, password } = req.body;

    const userResult = await pool.query('SELECT * FROM users WHERE username = $1', [username]);
    if (userResult.rows.length === 0) {
      return res.status(400).json({ error: 'Foydalanuvchi topilmadi yoki parol xato' });
    }

    const user = userResult.rows[0];

    // Parolni solishtirish
    const isMatch = await bcrypt.compare(password, user.password);
    if (!isMatch) {
      return res.status(400).json({ error: 'Foydalanuvchi topilmadi yoki parol xato' });
    }

    // JWT token yaratish
    const token = jwt.sign({ userId: user.id, username: user.username }, JWT_SECRET, {
      expiresIn: '1h',
    });

    res.json({ message: 'Muvaffaqiyatli kirdingiz', token });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Server xatoligi' });
  }
});

// 3. GET /tasks — Joriy foydalanuvchining vazifalarini olish (Himoyalangan)
app.get('/tasks', autentifikatsiyaTalabQilish, async (req, res) => {
  try {
    const userId = req.user.userId;

    // Faqat joriy foydalanuvchiga tegishli tasklarni qaytarish
    const tasks = await pool.query('SELECT * FROM tasks WHERE user_id = $1 ORDER BY id DESC', [userId]);
    
    res.json(tasks.rows);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Server xatoligi' });
  }
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server ${PORT}-portda ishga tushdi`));
2. Frontend Qismi (React)
Auth.jsx (Login va Register formalari)
JavaScript
import React, { useState } from 'react';

function Auth({ onLoginSuccess }) {
  const [isLogin, setIsLogin] = useState(true);
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');

    const endpoint = isLogin ? '/login' : '/register';

    try {
      const response = await fetch(`http://localhost:5000${endpoint}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ username, password }),
      });

      const data = await response.json();

      if (!response.ok) {
        throw new Error(data.error || 'Xatolik yuz berdi');
      }

      if (isLogin) {
        // Tokenni localStorage'da saqlash
        localStorage.setItem('token', data.token);
        onLoginSuccess();
      } else {
        alert('Ro‘yxatdan o‘tdingiz! Endi tizimga kiring.');
        setIsLogin(true);
      }
    } catch (err) {
      setError(err.message);
    }
  };

  return (
    <div style={{ maxWidth: '400px', margin: '50px auto', padding: '20px', border: '1px solid #ccc' }}>
      <h2>{isLogin ? 'Tizimga kirish' : 'Ro‘yxatdan o‘tish'}</h2>
      {error && <p style={{ color: 'red' }}>{error}</p>}
      
      <form onSubmit={handleSubmit}>
        <div style={{ marginBottom: '10px' }}>
          <label>Username:</label><br />
          <input
            type="text"
            value={username}
            onChange={(e) => setUsername(e.target.value)}
            required
            style={{ width: '100%', padding: '8px' }}
          />
        </div>
        
        <div style={{ marginBottom: '15px' }}>
          <label>Parol:</label><br />
          <input
            type="password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            required
            style={{ width: '100%', padding: '8px' }}
          />
        </div>

        <button type="submit" style={{ width: '100%', padding: '10px', background: '#007BFF', color: '#fff', border: 'none' }}>
          {isLogin ? 'Kirish' : 'Ro‘yxatdan o‘tish'}
        </button>
      </form>

      <p style={{ marginTop: '15px', textAlign: 'center' }}>
        {isLogin ? 'Hisobingiz yo‘qmi?' : 'Hisobingiz bormi?'}{' '}
        <span
          onClick={() => setIsLogin(!isLogin)}
          style={{ color: 'blue', cursor: 'pointer', textDecoration: 'underline' }}
        >
          {isLogin ? 'Ro‘yxatdan o‘ting' : 'Tizimga kiring'}
        </span>
      </p>
    </div>
  );
}

export default Auth;
TaskList.jsx (Himoyalangan so'rovlar orqali Tasklarni olish)
JavaScript
import React, { useEffect, useState } from 'react';

function TaskList({ onLogout }) {
  const [tasks, setTasks] = useState([]);
  const [error, setError] = useState('');

  useEffect(() => {
    const fetchTasks = async () => {
      const token = localStorage.getItem('token');

      try {
        const response = await fetch('http://localhost:5000/tasks', {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${token}`, // Tokenni har bir himoyalangan so'rovga qo'shish
          },
        });

        if (response.status === 401 || response.status === 403) {
          onLogout(); // Token yaroqsiz bo'lsa chiqib ketish
          return;
        }

        const data = await response.json();
        if (!response.ok) throw new Error(data.error);

        setTasks(data);
      } catch (err) {
        setError(err.message);
      }
    };

    fetchTasks();
  }, [onLogout]);

  return (
    <div style={{ maxWidth: '600px', margin: '50px auto', padding: '20px' }}>
      <h2>Mening Vazifalarim</h2>
      <button onClick={onLogout} style={{ float: 'right', padding: '5px 10px', background: 'red', color: '#fff', border: 'none' }}>
        Chiqish
      </button>
      
      {error && <p style={{ color: 'red' }}>{error}</p>}
      
      <ul>
        {tasks.length === 0 ? (
          <p>Hozircha vazifalar yo'q.</p>
        ) : (
          tasks.map((task) => (
            <li key={task.id} style={{ margin: '10px 0', padding: '10px', background: '#f9f9f9', border: '1px solid #ddd' }}>
              <strong>{task.title}</strong> — {task.description}
            </li>
          ))
        )}
      </ul>
    </div>
  );
}

export default TaskList;
3. README.md (Holat checklist'i)
Markdown
# TaskFlow Loyihasi

## Holat Checklist'i (Status Checklist)
- [x] **POST /register** — Parolni `bcrypt.hash()` yordamida xavfsiz saqlash
- [x] **POST /login** — `bcrypt.compare()` orqali parolni tekshirish va JWT token qaytarish
- [x] **autentifikatsiyaTalabQilish** — `Authorization: Bearer <token>` header'ini tekshiruvchi middleware
- [x] **GET /tasks** — `WHERE user_id = $1` orqali faqat joriy foydalanuvchi ma'lumotlarini qaytarish
- [x] **Frontend Login/Register formalari** — React yordamida interfeys yaratish va token'ni `localStorage`'da saqlash
- [x] **Himoyalangan so'rovlar** — Barcha so'rovlarga `Authorization` header'ini avtomatik
