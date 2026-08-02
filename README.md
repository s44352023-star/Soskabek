💻 SCSS / CSS Kod
SCSS
// ==========================================
// 1. PLACEHOLDERS (DRY Prinsipi uchun)
// ==========================================

// Asosiy kartochka stili uchun placeholder
%base-card {
    background-color: #ffffff;
    border-radius: 12px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.06);
    padding: 20px;
    border: 1px solid #e2e8f0;
}

// Elementlarni markazga keltirish uchun placeholder
%flex-center {
    display: flex;
    align-items: center;
    justify-content: center;
}


// ==========================================
// 2. ALERT KOMPONENTI (@extend bilan)
// ==========================================

.alert {
    @extend %base-card; // Asosiy xususiyatlarni meros qilib olamiz
    display: flex;
    align-items: center;
    gap: 12px;
    font-size: 15px;
    font-weight: 500;
    margin-bottom: 16px;

    // Variantlar
    &--success {
        background-color: #f0fdf4;
        border-color: #bbf7d0;
        color: #166534;
    }

    &--warning {
        background-color: #fefce8;
        border-color: #fef08a;
        color: #854d0e;
    }

    &--error {
        background-color: #fef2f2;
        border-color: #fecaca;
        color: #991b1b;
    }
}


// ==========================================
// 3. BADGE KOMPONENTI (4 ta variant)
// ==========================================

.badge {
    @extend %flex-center;
    display: inline-flex; // %flex-center dagi flex'ni qo'shimcha sozlash uchun
    padding: 6px 14px;
    border-radius: 20px;
    font-size: 12px;
    font-weight: 600;
    letter-spacing: 0.5px;
    text-transform: uppercase;

    // 1-variant: Primary
    &--primary {
        background-color: #eff6ff;
        color: #1d4ed8;
    }

    // 2-variant: Success
    &--success {
        background-color: #f0fdf4;
        color: #15803d;
    }

    // 3-variant: Warning
    &--warning {
        background-color: #fffbeb;
        color: #b45309;
    }

    // 4-variant: Danger
    &--danger {
        background-color: #fef2f2;
        color: #b91c1c;
    }
}


// ==========================================
// 4. CARD KOMPONENTI
// ==========================================

.card {
    @extend %base-card;
    max-width: 400px;

    &__header {
        font-size: 18px;
        font-weight: 700;
        color: #1e293b;
        margin-bottom: 8px;
    }

    &__body {
        font-size: 14px;
        color: #64748b;
        line-height: 1.5;
    }
}


// ==========================================
// 5. @extend VA @mixin FARQI HAQIDA IZOH
// ==========================================
/*
  ============================================================
  @extend va @mixin FARQI:
  ============================================================
  
  1. @extend (Meros olish):
     - Tanlangan selektorning barcha uslublarini boshqa selektorga ko'chiradi.
     - CSS kod hajmini tejaydi (kodni guruhlaydi: `.class1, .class2 { ... }`).
     - Parametrlar (argumentlar) qabul qila olmaydi.
     - Statik uslublarni takrorlamaslik uchun juda qulay.

  2. @mixin (Funksiya/Blok yaratish):
     - Xuddi funksiyaga o'xshaydi va o'ziga argumentlar (qiymatlar) qabul qilishi mumkin.
     - Har safar chaqirilganda, CSS kodini o'sha joyga to'g'ridan-to'g'ri nusxalab qo'yadi (duplicate code hosil qilishi mumkin).
     - Dinamik, qiymatlari o'zgarib turadigan uslublar uchun ishlatiladi.
  ============================================================
*/
