💻 Kod (HTML + CSS)
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sticky Nav & Tooltip Position Namunasi</title>
    <style>
        /* Umumiy sozlamalar va smooth scroll */
        html {
            scroll-behavior: smooth;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f4f6f9;
            color: #333;
        }

        /* 1. position: sticky bilan nav */
        .sticky-nav {
            position: sticky;
            top: 0;
            background-color: #2c3e50;
            color: #ffffff;
            padding: 15px 30px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            z-index: 1000; /* Boshqa elementlar ustida turishi uchun */
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.15);
        }

        .sticky-nav .logo {
            font-size: 20px;
            font-weight: bold;
            letter-spacing: 1px;
        }

        .sticky-nav ul {
            list-style: none;
            margin: 0;
            padding: 0;
            display: flex;
            gap: 20px;
        }

        .sticky-nav ul li a {
            color: #ffffff;
            text-decoration: none;
            font-size: 14px;
            transition: color 0.2s;
        }

        .sticky-nav ul li a:hover {
            color: #3498db;
        }

        /* Asosiy kontent konteyneri */
        .container {
            max-width: 1000px;
            margin: 40px auto;
            padding: 0 20px;
        }

        h2 {
            text-align: center;
            margin-bottom: 30px;
            color: #2c3e50;
        }

        /* Kartochkalar paneli */
        .cards-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 30px;
            margin-bottom: 50px;
        }

        /* 2. relative ota element */
        .card {
            position: relative; 
            background: #ffffff;
            border-radius: 10px;
            padding: 25px;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }

        .card:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 20px rgba(0, 0, 0, 0.1);
        }

        .card h3 {
            margin-top: 0;
            color: #2c3e50;
        }

        .card p {
            color: #666;
            font-size: 14px;
            line-height: 1.5;
        }

        /* 3. position: absolute bilan badge / tooltip */
        .card-tooltip {
            position: absolute;
            top: -12px;
            right: 15px;
            background-color: #e74c3c;
            color: white;
            padding: 5px 12px;
            font-size: 12px;
            font-weight: bold;
            border-radius: 20px;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
            z-index: 10; /* Kartochka ichidagi qatlamlar ustida turishi uchun */
        }

        .card-tooltip.pro {
            background-color: #3498db;
        }

        .card-tooltip.sale {
            background-color: #2ecc71;
        }

        /* Sahifani uzun qilish uchun qo'shimcha bo'lim */
        .content-section {
            background: #ffffff;
            padding: 30px;
            border-radius: 10px;
            margin-bottom: 50px;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
        }

        /* 4. position: fixed bilan floating 'Yuqoriga' tugmasi */
        .scroll-top-btn {
            position: fixed;
            bottom: 30px;
            right: 30px;
            background-color: #2c3e50;
            color: white;
            width: 45px;
            height: 45px;
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            text-decoration: none;
            font-size: 18px;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
            z-index: 999;
            transition: background-color 0.2s, transform 0.2s;
        }

        .scroll-top-btn:hover {
            background-color: #3498db;
            transform: scale(1.1);
        }
    </style>
</head>
<body id="top">

    <!-- Sticky Nav -->
    <nav class="sticky-nav">
        <div class="logo">PositionLab</div>
        <ul>
            <li><a href="#cards">Kartochkalar</a></li>
            <li><a href="#about">Haqida</a></li>
            <li><a href="#contact">Aloqa</a></li>
        </ul>
    </nav>

    <div class="container">
        <h2 id="cards">Position Xususiyatlari Namunasi</h2>

        <!-- 3 ta kartochka -->
        <div class="cards-grid">
            <!-- 1-kartochka -->
            <div class="card">
                <span class="card-tooltip">Yangi</span>
                <h3>Starter Paket</h3>
                <p>Boshlang'ich darajadagilar uchun maxsus imkoniyatlar va qulay shart-lar to'plami.</p>
            </div>

            <!-- 2-kartochka -->
            <div class="card">
                <span class="card-tooltip pro">Tavsiya etiladi</span>
                <h3>Pro Paket</h3>
                <p>Professional foydalanuvchilar uchun kengaytirilgan funksiyalar va tezkor yordam.</p>
            </div>

            <!-- 3-kartochka -->
            <div class="card">
                <span class="card-tooltip sale">Aksiya -20%</span>
                <h3>VIP Paket</h3>
                <p>Barcha imkoniyatlar jamlanmasi hamda shaxsiy menejer xizmati bilan ta'minlanadi.</p>
            </div>
        </div>

        <div class="content-section" id="about">
            <h3>Position haqida qisqacha</h3>
            <p><strong>Sticky:</strong> Sahifa skroll qilinganda element belgilangan nuqtaga (masalan, `top: 0`) kelgach yopishib qoladi.</p>
            <p><strong>Relative & Absolute:</strong> Ota elementga `position: relative` berilganda, uning ichidagi bola element `position: absolute` yordamida aynan ota element chegarasida aniq joylashadi.</p>
            <p><strong>Fixed:</strong> Sahifa skroll qilinishidan qat'i nazar, ekran oynasiga nisbatan doimiy joyda turadi (masalan, pastdagi tugma).</p>
        </div>

        <div class="content-section" id="contact">
            <h3>Aloqa ma'lumotlari</h3>
            <p>Savollar bo'yicha biz bilan bog'lanishingiz mumkin. Sahifani yuqoriga chiqarish uchun pastdagi tugmani bosing.</p>
        </div>
    </div>

    <!-- Floating 'Yuqoriga' tugmasi (Fixed) -->
    <a href="#top" class="scroll-top-btn" title="Yuqoriga qaytish">↑</a>

</body>
</html>
