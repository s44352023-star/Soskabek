// ════════════════════════════════════════════════════════════════════
// 2-BOSQICH: Backend API - tasks va categories uchun CRUD
// ════════════════════════════════════════════════════════════════════

const express = require('express');
const pool = require('./db');
const app = express();
app.use(express.json());

// ─────────────────────────────────────────────────────────────────────
// 1) GET /tasks - JOIN orqali category nomi bilan birga
// ─────────────────────────────────────────────────────────────────────

app.get('/tasks', async (req, res) => {
  const natija = await pool.query(`
    SELECT tasks.*, categories.nomi AS category_nomi
    FROM tasks
    JOIN categories ON tasks.category_id = categories.id
    ORDER BY tasks.yaratilgan_vaqt DESC
  `);
  res.json(natija.rows);
});

// ─────────────────────────────────────────────────────────────────────
// 2) POST /tasks - validatsiya + parametrlashtirilgan so'rov
// ─────────────────────────────────────────────────────────────────────

app.post('/tasks', async (req, res) => {
  const { sarlavha, matn, category_id } = req.body;
  if (!sarlavha || !category_id) {
    return res.status(400).json({ xato: "'sarlavha' va 'category_id' majburiy" });
  }
  const natija = await pool.query(
    'INSERT INTO tasks (sarlavha, matn, category_id) VALUES ($1, $2, $3) RETURNING *',
    [sarlavha, matn, category_id]
  );
  res.status(201).json(natija.rows[0]);
});

// ─────────────────────────────────────────────────────────────────────
// 3) DELETE /categories/:id - bog'liq tasks tekshiruvi bilan
// ─────────────────────────────────────────────────────────────────────

app.delete('/categories/:id', async (req, res) => {
  const { id } = req.params;

  const bogliqTasks = await pool.query(
    'SELECT COUNT(*) FROM tasks WHERE category_id = $1', [id]
  );
  if (Number(bogliqTasks.rows[0].count) > 0) {
    return res.status(400).json({
      xato: "Bu kategoriyada vazifalar bor, avval ularni o'chiring yoki boshqa kategoriyaga ko'chiring"
    });
  }

  await pool.query('DELETE FROM categories WHERE id = $1', [id]);
  res.status(204).send();
});

app.listen(3000, () => console.log('TaskFlow API: http://localhost:3000'));

// ─────────────────────────────────────────────────────────────────────
// 4) Ataylab xato - tekshiruvsiz o'chirish (izohda)
// ─────────────────────────────────────────────────────────────────────

// app.delete('/categories/:id', async (req, res) => {
//   const { id } = req.params;
//   await pool.query('DELETE FROM categories WHERE id = $1', [id]);   // tekshiruvsiz!
//   res.status(204).send();
// });
// ❌ Agar bog'liq tasks bo'lsa: foreign key constraint xatosi, 500 Internal Server Error
