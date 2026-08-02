💻 Kod (HTML + CSS)
HTML
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS orqali Dark/Light Tema Almashtirish</title>
    <style>
        /* 1. :root da barcha ranglar va o'lchamlar CSS o'zgaruvchi sifatida */
        :root {
            --bg-color: #f4f6f9;
            --card-bg: #ffffff;
            --text-color: #333333;
            --text-muted: #666666;
            --primary-color: #3b82f6;
            --primary-hover: #2563eb;
            --border-color: #e2e8f0;
            --shadow: 0 4px 15px rgba(0, 0, 0, 0.05);
            
            --spacing-sm: 8px;
            --spacing-md: 16px;
            --spacing-lg: 24px;
            --border-radius: 12px;
        }

        /* 2. Checkbox tanlanganda (checked bo'lganda) o'zgaruvchilar qiymati qorong'u tema uchun o'zgaradi */
        body:has(#theme-toggle:checked) {
            --bg-color: #0f172a;
            --card-bg: #1e293b;
            --text-color: #f8fafc;
            --text-muted: #94a3b8;
            --primary-color: #38bdf8;
            --primary-hover: #0ea5e9;
            --border-color: #334155;
            --shadow: 0 4px 20px rgba(0, 0, 0, 0.4);
        }

        /* Umumiy sahifa sozlamalari */
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            margin: 0;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            transition: background-color 0.3s ease, color 0.3s ease;
        }

        /* Yashirin checkbox */
        #theme-toggle {
            display: none;
        }

        /* Asosiy konteyner (Kartochka) */
        .wrapper {
            background-color: var(--card-bg);
            border: 1px solid var(--border-color);
            border-radius: var(--border-radius);
            box-shadow: var(--shadow);
            padding: 40px;
            width: 100%;
            max-width: 400px;
            box-sizing: border-box;
            transition: background-color 0.3s ease, border-color 0.3s ease, box-shadow 0.3s ease;
        }

        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: var(--spacing-lg);
        }

        h2 {
            margin: 0;
            font-size: 22px;
        }

        /* Switch Label dizayni */
        .theme-switch-label {
            cursor: pointer;
            background-color: var(--border-color);
            border: 2px solid var(--border-color);
            padding: 6px 14px;
            border-radius: 20px;
            font-size: 14px;
            font-weight: 600;
            display: inline-flex;
            align-items: center;
            gap: 6px;
            transition: background-color 0.2s, border-color 0.2s;
            user-select: none;
        }

        .theme-switch-label:hover {
            border-color: var(--primary-color);
        }

        /* Matnlar */
        p {
            color: var(--text-muted);
            line-height: 1.6;
            margin-bottom: var(--spacing-lg);
            font-size: 15px;
        }

        /* Tugma */
        .btn {
            display: block;
            width: 100%;
            background-color: var(--primary-color);
            color: #ffffff;
            border: none;
            padding: 12px;
            border-radius: var(--spacing-sm);
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            text-align: center;
            text-decoration: none;
            box-sizing: border-box;
            transition: background-color 0.2s;
        }

        .btn:hover {
            background-color: var(--primary-hover);
        }

        /* Dinamik matn almashtirish (Yorug' / Qorong'u holatiga qarab label ichidagi matn o'zgaradi) */
        .theme-switch-label .dark-text {
            display: inline;
        }
        .theme-switch-label .light-text {
            display: none;
        }

        /* Checkbox yoqilganda label ichidagi matn va ikonkalarni o'zgartirish */
        body:has(#theme-toggle:checked) .theme-switch-label .dark-text {
            display: none;
        }
        body:has(#theme-toggle:checked) .theme-switch-label .light-text {
            display: inline;
        }
    </style>
</head>
<body>

    <!-- Yashirin Checkbox -->
    <input type="checkbox" id="theme-toggle">

    <div class="wrapper">
        <div class="header">
            <h2>Tema sozlamasi</h2>
            <!-- Label checkboxni boshqaradi -->
            <label for="theme-toggle" class="theme-switch-label">
                <span class="dark-text">🌙 Qorong'u</span>
                <span class="light-text">☀️ Yorug'</span>
            </label>
        </div>

        <p>
            Ushbu sahifa to'liq CSS texnologiyalari yordamida yaratilgan. Hech qanday JavaScript kodidan foydalanilmagan. Ranglar va o'lchamlar faqatgina CSS o'zgaruvchilari orqali boshqariladi.
        </p>

        <a href="#" class="btn">Tugmani sinash</a>
    </div>

</body>
</html>
