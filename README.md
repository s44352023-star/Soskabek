1. Loyiha strukturasi (App.jsx)
Quyida barcha marshrutlar va komponentlar bog'liqligi keltirilgan:

JavaScript
import React, { useState, createContext, useContext } from 'react';
import {
  BrowserRouter,
  Routes,
  Route,
  NavLink,
  Link,
  useParams,
  useSearchParams,
  useNavigate,
  Outlet,
  Navigate
} from 'react-router-dom';

// --- Auth Context (Protected Route uchun imitatsiya) ---
const AuthContext = createContext(null);

function AuthProvider({ children }) {
  const [user, setUser] = useState(null); // null yoki { name: 'Admin' }

  const login = (callback) => {
    setUser({ name: 'Talaba' });
    callback();
  };

  const logout = (callback) => {
    setUser(null);
    callback();
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

function useAuth() {
  return useContext(AuthContext);
}

// --- Himoyalangan yo'l (Protected Route) ---
function ProtectedRoute({ children }) {
  const { user } = useAuth();
  if (!user) {
    return <Navigate to="/login" replace />;
  }
  return children;
}

// --- Asosiy Layout (Navbar va Footer) ---
function Layout() {
  const activeStyle = ({ isActive }) => ({
    color: isActive ? '#ff4757' : '#2f3542',
    fontWeight: isActive ? 'bold' : 'normal',
    textDecoration: 'none',
    marginRight: '15px'
  });

  return (
    <div style={{ fontFamily: 'Arial, sans-serif', padding: '20px' }}>
      <header style={{ marginBottom: '20px', borderBottom: '1px solid #ddd', paddingBottom: '10px' }}>
        <nav>
          <NavLink to="/" style={activeStyle}>Bosh sahifa</NavLink>
          <NavLink to="/kurslar" style={activeStyle}>Kurslar</NavLink>
          <NavLink to="/qidir" style={activeStyle}>Qidiruv</NavLink>
          <NavLink to="/profil" style={activeStyle}>Profil (Protected)</NavLink>
        </nav>
      </header>

      <main>
        <Outlet />
      </main>
    </div>
  );
}

// --- 1. Bosh sahifa (/) ---
function Home() {
  return (
    <div>
      <h2>Bosh sahifaga xush kelibsiz!</h2>
      <p>React Router imkoniyatlarini o'rganish uchun mo'ljallangan namuna.</p>
    </div>
  );
}

// --- 2. Kurslar ro'yxati (/kurslar) ---
function Kurslar() {
  const kurslarList = [
    { id: 1, title: 'React.js 0 dan boshlovchilarga' },
    { id: 2, title: 'Advanced JavaScript' },
    { id: 3, title: 'UI/UX Dizayn asoslari' },
  ];

  return (
    <div>
      <h2>Bizning kurslar</h2>
      <ul>
        {kurslarList.map(kurs => (
          <li key={kurs.id} style={{ margin: '10px 0' }}>
            <Link to={`/kurslar/${kurs.id}`}>{kurs.title}</Link>
          </li>
        ))}
      </ul>
    </div>
  );
}

// --- 3. Kurs tafsilotlari (/kurslar/:id) ---
function KursDetay() {
  const { id } = useParams();
  const navigate = useNavigate();

  return (
    <div>
      <h3>Kurs ID raqami: {id}</h3>
      <p>Bu yerda {id}-kurs haqida batafsil ma'lumotlar joylashgan.</p>
      <button 
        onClick={() => navigate(-1)} 
        style={{ padding: '8px 15px', cursor: 'pointer' }}
      >
        Orqaga qaytish
      </button>
    </div>
  );
}

// --- 4. Qidiruv sahifasi (?q=&p=) ---
function Qidiruv() {
  const [searchParams, setSearchParams] = useSearchParams();
  const query = searchParams.get('q') || '';
  const page = searchParams.get('p') || '1';

  const handleSearchChange = (e) => {
    const val = e.target.value;
    setSearchParams({ q: val, p: page });
  };

  const nextP = () => {
    setSearchParams({ q: query, p: Number(page) + 1 });
  };

  return (
    <div>
      <h2>Qidiruv sahifasi</h2>
      <input 
        type="text" 
        value={query} 
        onChange={handleSearchChange} 
        placeholder="Nimani qidiryapsiz?" 
        style={{ padding: '8px', width: '300px' }}
      />
      <p>Natijalar (Qidiruv so'zi: <b>{query}</b>, Sahifa: <b>{page}</b>)</p>
      <button onClick={nextP} style={{ padding: '5px 10px' }}>Keyingi sahifa</button>
    </div>
  );
}

// --- 5. Profil sahifasi (Protected) ---
function Profil() {
  const { user, logout } = useAuth();
  const navigate = useNavigate();

  return (
    <div>
      <h2>Foydalanuvchi profili</h2>
      <p>Xush kelibsiz, <b>{user?.name}</b>!</p>
      <button 
        onClick={() => logout(() => navigate('/'))}
        style={{ padding: '8px 15px', backgroundColor: '#ff4757', color: '#fff', border: 'none', cursor: 'pointer' }}
      >
        Chiqish (Logout)
      </button>
    </div>
  );
}

// --- 6. Login sahifasi ---
function Login() {
  const { login } = useAuth();
  const navigate = useNavigate();

  const handleLogin = () => {
    login(() => {
      navigate('/profil', { replace: true });
    });
  };

  return (
    <div style={{ padding: '20px' }}>
      <h2>Tizimga kirish</h2>
      <p>Profil sahifasini ko'rish uchun avval kiring.</p>
      <button 
        onClick={handleLogin}
        style={{ padding: '10px 20px', backgroundColor: '#2ed573', color: '#fff', border: 'none', cursor: 'pointer' }}
      >
        Kirish (Login)
      </button>
    </div>
  );
}

// --- 7. 404 Sahifasi ---
function NotFound() {
  return (
    <div style={{ textAlign: 'center', padding: '50px' }}>
      <h1>404</h1>
      <p>Sahifa topilmadi!</p>
      <Link to="/">Bosh sahifaga qaytish</Link>
    </div>
  );
}

// --- Asosiy App Komponenti ---
export default function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <Routes>
          {/* Layout va Nested Routes */}
          <Route path="/" element={<Layout />}>
            <Route index element={<Home />} />
            <Route path="kurslar" element={<Kurslar />} />
            <Route path="kurslar/:id" element={<KursDetay />} />
            <Route path="qidir" element={<Qidiruv />} />
            
            {/* Protected Route */}
            <Route 
              path="profil" 
              element={
                <ProtectedRoute>
                  <Profil />
                </ProtectedRoute>
              } 
            />
          </Route>

          <Route path="/login" element={<Login />} />

          {/* 404 Not Found */}
          <Route path="*" element={<NotFound />} />
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}
Asosiy xususiyatlar tushuntirishi:
NavLink va Active Style: Hozirgi ochilgan sahifa sarlavkasi avtomatik ravishda qizil rangda va qalin (bold) ko'rinishda belgilanadi.

useParams va Dinamik marshrut: /kurslar/:id orqali har bir kursning o'ziga xos ID raqami olinib, sahifada ko'rsatiladi.

useSearchParams: Qidiruv satridagi ?q=...&p=... parametrlari URL bilan to'g'ridan-to'g'ri sinxronizatsiya qilinadi.

useNavigate: Login tugmasi bosilgach, foydalanuvchi dasturiy ravishda /profil sahifasiga yo'naltiriladi. navigate(-1) esa brauzerdagi Back (orqaga) vazifasini bajaradi.

Browser Back/Forward: BrowserRouter barcha brauzer tarixini (history stack) avtomatik ravishda boshqaradi.
