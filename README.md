1. Muammoli holat va Premature Optimization xatosi
⚠️ Premature Optimization (Erta optimallashtirish) xatosi:
Ko'pchilik dasturchilar ilovani yozishni boshlaganoq har bir kichik funksiyani useCallback, har bir komponentni React.memo bilan o'rab chiqishadi. Bu noto'g'ri.

Nima uchun? Chunki har bir memo yoki useMemo o'z navbatida xotira va solishtirish uchun CPU resursini sarflaydi. Agar komponent oddiy bo'lsa, memo solishtirib o'tirguncha, React uni shunchaki qayta chizib qo'ygani tezroq ishlaydi.

Qoida: Avval kod yoziladi, React DevTools Profiler orqali haqiqiy qayerda "qotish" (bottleneck) borligi o'lchanadi, faqat shundan keyingina maqsadli optimallashtiriladi.

2. Kodni Qismlarga Bo'lish (React.lazy + Suspense)
Katta ro'yxat sahifasini dastlabki yuklanishda (bundle size) yengillashtirish uchun React.lazy yordamida dinamik import qilamiz.

JavaScript
import React, { useState, useTransition, lazy, Suspense } from 'react';

// Og'ir sahifani lazy yuklash
const HeavyListDashboard = lazy(() => import('./HeavyListDashboard'));

export default function App() {
  const [showDashboard, setShowDashboard] = useState(false);

  return (
    <div className="app">
      <h1>Katta Ma'lumotlar Boshqaruvi</h1>
      <button onClick={() => setShowDashboard(prev => !prev)}>
        {showDashboard ? 'Yopish' : 'Ro\'yxatni ochish'}
      </button>

      <Suspense fallback={<div className="loader">Yuklanmoqda... ⏳</div>}>
        {showDashboard && <HeavyListDashboard />}
      </Suspense>
    </div>
  );
}
3. Asosiy Optimallashtirilgan Komponent (HeavyListDashboard)
Quyida 1000+ elementli ro'yxat uchun Qidiruv, Filter, Sort, useMemo, useCallback va React.memo jamlangan to'liq mantiq keltirilgan.

JavaScript
import React, { useState, useMemo, useCallback } from 'react';

// 1. React.memo bilan o'ralgan Card komponenti
// Faqat o'zining props'lari o'zgandagina qayta chiziladi
const ProductCard = React.memo(({ item, onSelect }) => {
  console.log(`Render: ${item.name}`); // Profiler test qilish uchun
  return (
    <div className="product-card" onClick={() => onSelect(item.id)}>
      <h4>{item.name}</h4>
      <p>Kategoriya: {item.category}</p>
      <span>{item.price.toLocaleString()} so'm</span>
    </div>
  );
});

// 1000 ta elementdan iborat mock data yaratish
const generateMockData = () => {
  const categories = ['Elektronika', Kiyim, 'Oziq-ovqat', 'Kitoblar'];
  return Array.from({ length: 1500 }, (_, index) => ({
    id: index + 1,
    name: `Mahsulot #${index + 1}`,
    category: categories[index % categories.length],
    price: Math.floor(Math.random() * 500000) + 10000,
  }));
};

export default function HeavyListDashboard() {
  const [items] = useState(generateMockData);
  const [search, setSearch] = useState('');
  const [selectedCategory, setSelectedCategory] = useState('ALL');
  const [sortOrder, setSortOrder] = useState('asc');

  // 2. useMemo — Qimmat filter va sort amaliyotini keshlash
  // 1500 ta elementni har safar renderda qayta filter qilmaslik uchun
  const filteredAndSortedItems = useMemo(() => {
    console.log('⚡ Qimmat hisob-kitob bajarildi: Filter va Sort');
    
    let result = items.filter((item) => {
      const matchesSearch = item.name.toLowerCase().includes(search.toLowerCase());
      const matchesCategory = selectedCategory === 'ALL' || item.category === selectedCategory;
      return matchesSearch && matchesCategory;
    });

    return result.sort((a, b) => {
      if (sortOrder === 'asc') return a.price - b.price;
      return b.price - a.price;
    });
  }, [items, search, selectedCategory, sortOrder]);

  // 3. useCallback — Har birrenderda funksiya manzili o'zgarmasligini ta'minlash
  // Bu ProductCard'dagi React.memo behuda ishlamasligi uchun muhim
  const handleSelect = useCallback((id) => {
    console.log(`Tanlangan mahsulot ID: ${id}`);
  }, []);

  return (
    <div className="dashboard">
      <div className="controls">
        <input
          type="text"
          placeholder="Qidirish..."
          value={search}
          onChange={(e) => setSearch(e.target.value)}
        />

        <select 
          value={selectedCategory} 
          onChange={(e) => setSelectedCategory(e.target.value)}
        >
          <option value="ALL">Barcha kategoriyalar</option>
          <option value="Elektronika">Elektronika</option>
          <option value="Kiyim">Kiyim</option>
          <option value="Oziq-ovqat">Oziq-ovqat</option>
          <option value="Kitoblar">Kitoblar</option>
        </select>

        <select 
          value={sortOrder} 
          onChange={(e) => setSortOrder(e.target.value)}
        >
          <option value="asc">Narx: Arzonidan</option>
          <option value="desc">Narx: Qimmatidan</option>
        </select>
      </div>

      <div className="list-container">
        <p>Topildi: {filteredAndSortedItems.length} ta mahsulot</p>
        <div className="grid">
          {filteredAndSortedItems.map((item) => (
            <ProductCard 
              key={item.id} 
              item={item} 
              onSelect={handleSelect} 
            />
          ))}
        </div>
      </div>
    </div>
  );
}
