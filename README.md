1. Backend Qismi (Node.js + Express + PostgreSQL)
GET /tasks endpoint'ini query parametrlari (q, category_id, page, limit) bilan boyitamiz:

JavaScript
// GET /tasks — Qidiruv, filtr va sahifalash bilan
app.get('/tasks', autentifikatsiyaTalabQilish, async (req, res) => {
  try {
    const userId = req.user.userId;
    const { q, category_id, page = 1, limit = 10 } = req.query;

    const pageNum = parseInt(page, 10);
    const limitNum = parseInt(limit, 10);
    const offset = (pageNum - 1) * limitNum;

    // Bazaviy so'rov (faqat joriy foydalanuvchiga tegishli)
    let query = 'SELECT * FROM tasks WHERE user_id = $1';
    let queryParams = [userId];
    let paramIndex = 2;

    // 1. Qidiruv bo'yicha (ILIKE sarlavha bo'yicha)
    if (q) {
      query += ` AND title ILIKE $${paramIndex}`;
      queryParams.push(`%${q}%`);
      paramIndex++;
    }

    // 2. Category bo'yicha filtr
    if (category_id) {
      query += ` AND category_id = $${paramIndex}`;
      queryParams.push(category_id);
      paramIndex++;
    }

    // 3. Sahifalash (LIMIT / OFFSET) va tartiblash
    query += ` ORDER BY id DESC LIMIT $${paramIndex} OFFSET $${paramIndex + 1}`;
    queryParams.push(limitNum, offset);

    const tasks = await pool.query(query, queryParams);

    // Umumiy sonini olish (sahifalash uchun foydali)
    // Shu shartlar asosida umumiy elementlar sonini hisoblash ham mumkin
    res.json(tasks.rows);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Server xatoligi' });
  }
});
2. Frontend Qismi (React + Redux Toolkit)
Redux Slice (taskSlice.js)
JavaScript
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

export const fetchTasks = createAsyncThunk(
  'tasks/fetchTasks',
  async ({ search, categoryId, page }, thunkAPI) => {
    const token = localStorage.getItem('token');
    const params = new URLSearchParams();
    
    if (search) params.append('q', search);
    if (categoryId) params.append('category_id', categoryId);
    if (page) params.append('page', page);
    params.append('limit', '5');

    const response = await fetch(`http://localhost:5000/tasks?${params.toString()}`, {
      headers: { 'Authorization': `Bearer ${token}` }
    });
    
    const data = await response.json();
    if (!response.ok) throw new Error(data.error);
    return data;
  }
);

const taskSlice = createSlice({
  name: 'tasks',
  initialState: {
    items: [],
    status: 'idle',
    search: '',
    categoryId: '',
    page: 1,
    error: null,
  },
  reducers: {
    setSearch: (state, action) => {
      state.search = action.payload;
      state.page = 1; // Qidirganda 1-sahifaga qaytish
    },
    setCategory: (state, action) => {
      state.categoryId = action.payload;
      state.page = 1; // Filtr o'zgarganda 1-sahifaga qaytish
    },
    setPage: (state, action) => {
      state.page = action.payload;
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchTasks.pending, (state) => { state.status = 'loading'; })
      .addCase(fetchTasks.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.items = action.payload;
      })
      .addCase(fetchTasks.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.error.message;
      });
  }
});

export const { setSearch, setCategory, setPage } = taskSlice.actions;
export default taskSlice.reducer;
Debounce va Filtr komponenti (TaskList.jsx)
JavaScript
import React, { useEffect, useState } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchTasks, setSearch, setCategory, setPage } from './taskSlice';

function TaskList() {
  const dispatch = useDispatch();
  const { items, search, categoryId, page, status } = useSelector((state) => state.tasks);
  
  // Debounce uchun local state
  const [inputValue, setInputValue] = useState(search);

  // 400ms Debounce mexanizmi
  useEffect(() => {
    const handler = setTimeout(() => {
      dispatch(setSearch(inputValue));
    }, 400);

    return () => {
      clearTimeout(handler);
    };
  }, [inputValue, dispatch]);

  // Parametrlar o'zgarganda ma'lumotni qayta yuklash
  useEffect(() => {
    dispatch(fetchTasks({ search, categoryId, page }));
  }, [dispatch, search, categoryId, page]);

  return (
    <div style={{ maxWidth: '600px', margin: '30px auto', padding: '20px' }}>
      <h2>Vazifalar ro'yxati</h2>

      {/* Qidiruv va Filtr boshqaruvlari */}
      <div style={{ display: 'flex', gap: '10px', marginBottom: '20px' }}>
        <input
          type="text"
          placeholder="Sarlavha bo'yicha qidirish..."
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          style={{ flex: 1, padding: '8px' }}
        />

        <select
          value={categoryId}
          onChange={(e) => dispatch(setCategory(e.target.value))}
          style={{ padding: '8px' }}
        >
          <option value="">Barcha kategoriyalar</option>
          <option value="1">Ish (Work)</option>
          <option value="2">Shaxsiy (Personal)</option>
        </select>
      </div>

      {status === 'loading' && <p>Yuklanmoqda...</p>}

      {/* Ro'yxat */}
      <ul>
        {items.map((task) => (
          <li key={task.id} style={{ padding: '10px', borderBottom: '1px solid #ddd' }}>
            <strong>{task.title}</strong> — {task.description}
          </li>
        ))}
      </ul>

      {/* Sahifalash (Pagination) tugmalari */}
      <div style={{ marginTop: '20px', display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
        <button 
          disabled={page === 1} 
          onClick={() => dispatch(setPage(page - 1))}
          style={{ padding: '5px 15px' }}
        >
          Oldingi
        </button>
        <span>Sahifa: {page}</span>
        <button 
          onClick={() => dispatch(setPage(page + 1))}
          style={{ padding: '5px 15px' }}
        >
          Keyingi
        </button>
      </div>
    </div>
  );
}

export default TaskList;
3. README.md (Holat checklist'i)
Markdown
# TaskFlow Loyihasi - Qidiruv va Sahifalash

## Holat Checklist'i (Status Checklist)
- [x] **GET /tasks** — `?q=...&category_id=...&page=...` query parametrlari qo'llab-quvvatlandi
- [x] **ILIKE qidiruvi** — Katta-kichik harfga sezgir bo'lmagan sarlavha bo'yicha qidiruv ishlaydi
- [x] **LIMIT / OFFSET** — PostgreSQL yordamida ma'lumotlarni sahifalash amalga oshirildi
- [x] **Frontend Debounce** — Qidiruv maydoni 400ms kechikish (debounce) bilan ishlaydi
- [x] **Category Dropdown** — Kategoriya bo'yicha filtr tanlash imkoniyati yaratildi
- [x] **Redux Toolkit** — Holat (state) Redux yordamida boshqarildi
