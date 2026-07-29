1. useLocalStorage Custom Hook
Ma'lumotlarni localStorage bilan xavfsiz va reaktiv sinxronizatsiya qilish uchun maxsus hook.

JavaScript
import { useState, useEffect } from 'react';

export function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  useEffect(() => {
    try {
      if (storedValue === null) {
        window.localStorage.removeItem(key);
      } else {
        window.localStorage.setItem(key, JSON.stringify(storedValue));
      }
    } catch (error) {
      console.error(error);
    }
  }, [key, storedValue]);

  return [storedValue, setStoredValue];
}
2. AuthContext va JWT Pattern
Foydalanuvchi sessiyasini saqlash, kirish (login) va chiqish (logout) mantiqi.

JavaScript
import React, { createContext, useContext, useMemo, useState } from 'react';
import { useLocalStorage } from './useLocalStorage';

const AuthContext = createContext(null);

export const AuthProvider = ({ children }) => {
  // Sahifa qayta yuklanganda tizimda qolish uchun useLocalStorage ishlatamiz
  const [token, setToken] = useLocalStorage('jwt_token', null);
  const [user, setUser] = useLocalStorage('user_data', null);

  const login = async (credentials) => {
    // Bu yerda API so'rovi simulyatsiya qilinadi (JWT token qaytadi deb faraz qilamiz)
    if (credentials.email === 'test@example.com' && credentials.password === 'password123') {
      const fakeToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...fake-jwt-token';
      const fakeUser = { email: credentials.email, name: 'Azizbek' };

      setToken(fakeToken);
      setUser(fakeUser);
      return { success: true };
    }
    throw new Error('Email yoki parol noto\'g\'ri!');
  };

  const logout = () => {
    setToken(null);
    setUser(null);
  };

  const value = useMemo(
    () => ({
      token,
      user,
      isAuthenticated: !!token,
      login,
      logout,
    }),
    [token, user]
  );

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth hooki AuthProvider ichida ishlatilishi shart!');
  }
  return context;
};
3. Protected Wrapper Komponenti (Return-to Pattern bilan)
Foydalanuvchi tizimga kirmagan bo'lsa, uni login sahifasiga yo'naltiradi va qayerdan kelganini state={{ dan: ... }} orqali saqlaydi.

JavaScript
import React from 'react';
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from './AuthContext';

export const ProtectedRoute = ({ children }) => {
  const { isAuthenticated } = useAuth();
  const location = useLocation();

  if (!isAuthenticated) {
    // Qaysi sahifadan kelganini state orqali uzatamiz (Return-to pattern)
    return <Navigate to="/login" state={{ dan: location.pathname }} replace />;
  }

  return children;
};
4. Login Formasi (Validation, Loading, Error bilan)
JavaScript
import React, { useState } from 'react';
import { useNavigate, useLocation } from 'react-router-dom';
import { useAuth } from './AuthContext';

export const LoginPage = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);

  const { login } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();

  // Return-to pattern bo'yicha qayerga qaytish kerakligini aniqlaymiz
  const origin = location.state?.dan || '/profil';

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');

    // Oddiy validatsiya
    if (!email || !password) {
      setError('Barcha maydonlarni to\'ldiring!');
      return;
    }

    setLoading(true);
    try {
      await login({ email, password });
      navigate(origin, { replace: true });
    } catch (err) {
      setError(err.message || 'Xatolik yuz berdi');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="login-page">
      <h2>Tizimga kirish</h2>
      {error && <div className="error-alert">{error}</div>}
      
      <form onSubmit={handleSubmit}>
        <div>
          <label>Email:</label>
          <input
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            placeholder="test@example.com"
          />
        </div>
        <div>
          <label>Parol:</label>
          <input
            type="password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            placeholder="password123"
          />
        </div>
        <button type="submit" disabled={loading}>
          {loading ? 'Yuklanmoqda...' : 'Kirish'}
        </button>
      </form>
    </div>
  );
};
5. Header Komponenti (Auth State ga qarab ko'rinish)
JavaScript
import React from 'react';
import { Link, useNavigate } from 'react-router-dom';
import { useAuth } from './AuthContext';

export const Header = () => {
  const { isAuthenticated, user, logout } = useAuth();
  const navigate = useNavigate();

  const handleLogout = () => {
    logout();
    navigate('/');
  };

  return (
    <header className="header">
      <nav className="nav-links">
        <Link to="/">Bosh sahifa</Link>
        <Link to="/kurslar">Kurslar</Link>
        {isAuthenticated && (
          <>
            <Link to="/profil">Profil</Link>
            <Link to="/sozlamalar">Sozlamalar</Link>
          </>
        )}
      </nav>

      <div className="auth-section">
        {isAuthenticated ? (
          <div className="user-info">
            <span>Salom, {user?.name}</span>
            <button onClick={handleLogout}>Chiqish</button>
          </div>
        ) : (
          <Link to="/login" className="login-btn">Kirish</Link>
        )}
      </div>
    </header>
  );
};
6. Boshqa Sahifalar va 404
JavaScript
export const HomePage = () => <div><h1>Bosh sahifa (Public)</h1></div>;
export const CoursesPage = () => <div><h1>Kurslar ro'yxati (Public)</h1></div>;
export const ProfilePage = () => <div><h1>Foydalanuvchi Profili (Private)</h1></div>;
export const SettingsPage = () => <div><h1>Sozlamalar (Private)</h1></div>;
export const NotFoundPage = () => <div><h1>404 - Sahifa topilmadi</h1></div>;
7. Router Birlashtirilishi (App.jsx)
JavaScript
import React from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { AuthProvider } from './AuthContext';
import { ProtectedRoute } from './ProtectedRoute';
import { Header } from './Header';
import { 
  HomePage, 
  CoursesPage, 
  ProfilePage, 
  SettingsPage, 
  LoginPage, 
  NotFoundPage 
} from './pages';

export default function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <Header />
        <main className="main-content">
          <Routes>
            {/* Public sahifalar */}
            <Route path="/" element={<HomePage />} />
            <Route path="/kurslar" element={<CoursesPage />} />
            <Route path="/login" element={<LoginPage />} />

            {/* Private sahifalar (Protected wrapper bilan) */}
            <Route 
              path="/profil" 
              element={
                <ProtectedRoute>
                  <ProfilePage />
                </ProtectedRoute>
              } 
            />
            <Route 
              path="/sozlamalar" 
              element={
                <ProtectedRoute>
                  <SettingsPage />
                </ProtectedRoute>
              } 
            />

            {/* 404 sahifa */}
            <Route path="*" element={<NotFoundPage />} />
          </Routes>
        </main>
      </BrowserRouter>
    </AuthProvider>
  );
}
