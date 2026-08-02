💻 Kod (HTML + CSS)
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Developer Profil Kartochkalari</title>
    <style>
        /* Umumiy sahifa sozlamalari */
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f0f4f8;
            color: #333;
            margin: 0;
            padding: 40px 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
        }

        h1 {
            color: #1e293b;
            margin-bottom: 40px;
            font-size: 28px;
            text-align: center;
        }

        /* CSS Grid konteyneri */
        .developers-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
            gap: 30px;
            width: 100%;
            max-width: 1100px;
        }

        /* Kartochka dizayni */
        .dev-card {
            background: #ffffff;
            border-radius: 16px;
            box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
            padding: 30px 20px;
            text-align: center;
            display: flex;
            flex-direction: column;
            align-items: center;
            position: relative;
            overflow: hidden;
            border: 1px solid #e2e8f0;
            /* Silliq animatsiya uchun transition */
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }

        /* Hover effekti */
        .dev-card:hover {
            transform: translateY(-8px);
            box-shadow: 0 12px 30px rgba(0, 0, 0, 0.12);
        }

        /* Profil rasmi (Doira shaklida) */
        .dev-avatar {
            width: 100px;
            height: 100px;
            border-radius: 50%;
            object-fit: cover;
            margin-bottom: 20px;
            border: 3px solid #3b82f6;
            padding: 3px;
            background-color: #ffffff;
            transition: transform 0.3s ease;
        }

        .dev-card:hover .dev-avatar {
            transform: scale(1.05);
        }

        /* Ism va lavozim */
        .dev-name {
            font-size: 20px;
            font-weight: 700;
            color: #1e293b;
            margin: 0 0 8px 0;
        }

        .dev-role {
            font-size: 14px;
            color: #64748b;
            font-weight: 500;
            margin-bottom: 15px;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }

        .dev-bio {
            font-size: 13px;
            color: #475569;
            line-height: 1.5;
            margin-bottom: 20px;
            flex-grow: 1;
        }

        /* Ijtimoiy tarmoq havolalari / Tugmalar */
        .dev-socials {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
            width: 100%;
            justify-content: center;
        }

        .social-link {
            text-decoration: none;
            background-color: #f1f5f9;
            color: #475569;
            width: 36px;
            height: 36px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 14px;
            font-weight: bold;
            transition: background-color 0.2s, color 0.2s;
        }

        .social-link:hover {
            background-color: #3b82f6;
            color: #ffffff;
        }

        /* Asosiy tugma stillashtirish */
        .dev-btn {
            display: inline-block;
            width: 100%;
            padding: 10px 0;
            background-color: #3b82f6;
            color: #ffffff;
            text-decoration: none;
            border-radius: 8px;
            font-weight: 600;
            font-size: 14px;
            box-shadow: 0 4px 10px rgba(59, 130, 246, 0.3);
            transition: background-color 0.2s, box-shadow 0.2s;
        }

        .dev-btn:hover {
            background-color: #2563eb;
            box-shadow: 0 6px 15px rgba(59, 130, 246, 0.4);
        }
    </style>
</head>
<body>

    <h1>Bizning Dasturchilar Jamoasi</h1>

    <!-- CSS Grid konteyneri -->
    <div class="developers-grid">
        
        <!-- 1-Dasturchi -->
        <div class="dev-card">
            <img src="https://images.unsplash.com/photo-1534528741775-53994a69daeb?w=150&auto=format&fit=crop&q=80" alt="Profil rasmi" class="dev-avatar">
            <h3 class="dev-name">Malika Karimova</h3>
            <span class="dev-role">Frontend Developer</span>
            <p class="dev-bio">React va Vue.js texnologiyalari mutaxassisi. Foydalanuvchilar uchun qulay interfeyslar yaratadi.</p>
            <div class="dev-socials">
                <a href="#" class="social-link">GH</a>
                <a href="#" class="social-link">IN</a>
                <a href="#" class="social-link">TG</a>
            </div>
            <a href="#" class="dev-btn">Profilni ko'rish</a>
        </div>

        <!-- 2-Dasturchi -->
        <div class="dev-card">
            <img src="https://images.unsplash.com/photo-1507003211169-0a1dd7228f2d?w=150&auto=format&fit=crop&q=80" alt="Profil rasmi" class="dev-avatar">
            <h3 class="dev-name">Jasur Alimov</h3>
            <span class="dev-role">Backend Developer</span>
            <p class="dev-bio">Node.js va Python yordamida xavfsiz hamda yuqori unumdor server arxitekturasini tuzadi.</p>
            <div class="dev-socials">
                <a href="#" class="social-link">GH</a>
                <a href="#" class="social-link">IN</a>
                <a href="#" class="social-link">TG</a>
            </div>
            <a href="#" class="dev-btn">Profilni ko'rish</a>
        </div>

        <!-- 3-Dasturchi -->
        <div class="dev-card">
            <img src="https://images.unsplash.com/photo-1517841905240-472988babdf9?w=150&auto=format&fit=crop&q=80" alt="Profil rasmi" class="dev-avatar">
            <h3 class="dev-name">Zuxra Tursunova</h3>
            <span class="dev-role">UI/UX Designer</span>
            <p class="dev-bio">Figma va Adobe XD ustasi. Har bir mahsulot dizaynini mukammal darajada ishlab chiqadi.</p>
            <div class="dev-socials">
                <a href="#" class="social-link">BE</a>
                <a href="#" class="social-link">IN</a>
                <a href="#" class="social-link">TG</a>
            </div>
            <a href="#" class="dev-btn">Profilni ko'rish</a>
        </div>

        <!-- 4-Dasturchi -->
        <div class="dev-card">
            <img src="https://images.unsplash.com/photo-1500648767791-00dcc994a43e?w=150&auto=format&fit=crop&q=80" alt="Profil rasmi" class="dev-avatar">
            <h3 class="dev-name">Sardor Rahimov</h3>
            <span class="dev-role">Full Stack Developer</span>
            <p class="dev-bio">To'liq sikldagi loyihalar ustida ishlaydi. Ma'lumotlar bazasi va mijoz qismini birdek mukammal qoradi.</p>
            <div class="dev-socials">
                <a href="#" class="social-link">GH</a>
                <a href="#" class="social-link">IN</a>
                <a href="#" class="social-link">TG</a>
            </div>
            <a href="#" class="dev-btn">Profilni ko'rish</a>
        </div>

    </div>

</body>
</html>
