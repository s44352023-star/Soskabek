1. Context'larni yaratish va Custom Hook'lar
Barcha Context'larni bir faylda yoki alohida modullarda yozish mumkin. Quyida barchasi bitta tuzilmada jamlangan.

JavaScript
import React, { createContext, useContext, useState, useMemo, useEffect } from 'react';

// ==========================================
// 1. USER CONTEXT
// ==========================================
const UserContext = createContext(null);

export const UserProvider = ({ children }) => {
  const [user, setUser] = useState(() => {
    const saved = localStorage.getItem('user');
    return saved ? JSON.parse(saved) : null;
  });

  useEffect(() => {
    if (user) {
      localStorage.setItem('user', JSON.stringify(user));
    } else {
      localStorage.removeItem('user');
    }
  }, [user]);

  const login = (userData) => setUser(userData);
  const logout = () => setUser(null);

  const value = useMemo(() => ({ user, login, logout }), [user]);

  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
};

export const useUser = () => {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser hooki UserProvider ichida ishlatilishi shart!');
  }
  return context;
};

// ==========================================
// 2. THEME CONTEXT
// ==========================================
const ThemeContext = createContext(null);

export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState(() => {
    return localStorage.getItem('theme') || 'light';
  });

  useEffect(() => {
    localStorage.setItem('theme', theme);
    document.documentElement.setAttribute('data-theme', theme);
  }, [theme]);

  const toggleTheme = () => {
    setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));
  };

  const value = useMemo(() => ({ theme, toggleTheme }), [theme]);

  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
};

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme hooki ThemeProvider ichida ishlatilishi shart!');
  }
  return context;
};

// ==========================================
// 3. CART CONTEXT
// ==========================================
const CartContext = createContext(null);

export const CartProvider = ({ children }) => {
  const [cart, setCart] = useState([]);

  const addToCart = (product) => {
    setCart((prev) => {
      const existing = prev.find((item) => item.id === product.id);
      if (existing) {
        return prev.map((item) =>
          item.id === product.id ? { ...item, quantity: item.quantity + 1 } : item
        );
      }
      return [...prev, { ...product, quantity: 1 }];
    });
  };

  const removeFromCart = (productId) => {
    setCart((prev) => prev.filter((item) => item.id !== productId));
  };

  const clearCart = () => setCart([]);

  const total = useMemo(() => {
    return cart.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }, [cart]);

  const value = useMemo(
    () => ({ cart, addToCart, removeFromCart, clearCart, total }),
    [cart, total]
  );

  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
};

export const useCart = () => {
  const context = useContext(CartContext);
  if (!context) {
    throw new Error('useCart hooki CartProvider ichida ishlatilishi shart!');
  }
  return context;
};
2. UI Komponentlar
Header Komponenti
Bevosita useUser, useTheme va useCart ma'lumotlarini olib ko'rsatadi.

JavaScript
import React from 'react';
import { useUser, useTheme, useCart } from './contextPath';

export const Header = () => {
  const { user, login, logout } = useUser();
  const { theme, toggleTheme } = useTheme();
  const { cart } = useCart();

  const totalItems = cart.reduce((sum, item) => sum + item.quantity, 0);

  return (
    <header className={`header ${theme}`}>
      <h1>Mening Do'konim</h1>
      
      <nav>
        <span>Savatcha: {totalItems} ta mahsulot</span>
        <button onClick={toggleTheme}>
          Mavzu: {theme === 'light' ? '🌞 Yorug\'' : '🌙 Qorong\'i'}
        </button>

        {user ? (
          <div>
            <span>Salom, {user.name}</span>
            <button onClick={logout}>Chiqish</button>
          </div>
        ) : (
          <button onClick={() => login({ name: 'Ali' })}>Kirish</button>
        )}
      </nav>
    </header>
  );
};
Mahsulot Kartochkasi
Bevosita useCart orqali ishlaydi.

JavaScript
import React from 'react';
import { useCart } from './contextPath';

export const ProductCard = ({ product }) => {
  const { addToCart } = useCart();

  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>{product.price.toLocaleString()} so'm</p>
      <button onClick={() => addToCart(product)}>Savatchaga qo'shish</button>
    </div>
  );
};
Savatcha Sahifasi
Mahsulotlar ro'yxati va umumiy summani ko'rsatadi.

JavaScript
import React from 'react';
import { useCart } from './contextPath';

export const CartPage = () => {
  const { cart, removeFromCart, clearCart, total } = useCart();

  if (cart.length === 0) {
    return <h2>Savatchangiz bo'sh</h2>;
  }

  return (
    <div className="cart-page">
      <h2>Savatcha</h2>
      <ul>
        {cart.map((item) => (
          <li key={item.id}>
            <span>{item.name} ({item.quantity} dona)</span>
            <span>{(item.price * item.quantity).toLocaleString()} so'm</span>
            <button onClick={() => removeFromCart(item.id)}>O'chirish</button>
          </li>
        ))}
      </ul>
      
      <div className="cart-summary">
        <h3>Jami: {total.toLocaleString()} so'm</h3>
        <button onClick={clearCart}>Savatchani tozalash</button>
      </div>
    </div>
  );
};
3. Ilovani Birlashtirish (App.jsx)
Barcha Provider'larni to'g'ri tartibda o'rash muhim:

JavaScript
import React from 'react';
import { UserProvider, ThemeProvider, CartProvider } from './contextPath';
import { Header } from './Header';
import { ProductCard } from './ProductCard';
import { CartPage } from './CartPage';

const sampleProduct = { id: 1, name: 'Smartfon', price: 2500000 };

export default function App() {
  return (
    <ThemeProvider>
      <UserProvider>
        <CartProvider>
          <div className="app">
            <Header />
            <main>
              <ProductCard product={sampleProduct} />
              <CartPage />
            </main>
          </div>
        </CartProvider>
      </UserProvider>
    </ThemeProvider>
  );
}
4. CSS Variables (Mavzular uchun)
CSS
:root[data-theme='light'] {
  --bg-color: #ffffff;
  --text-color: #333333;
}

:root[data-theme='dark'] {
  --bg-color: #1a1a1a;
  --text-color: #f0f0f0;
}

body {
  background-color: var(--bg-color);
  color: var(--text-color);
  transition: background-color 0.3s, color 0.3s;
}
