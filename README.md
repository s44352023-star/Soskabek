📁 Loyiha strukturasi
Plaintext
my-shop/
├── src/
│   ├── app/
│   │   ├── hooks.ts
│   │   └── store.ts
│   ├── features/
│   │   ├── api/
│   │   │   └── productsApi.ts
│   │   └── cart/
│   │       └── cartSlice.ts
│   ├── components/
│   │   ├── ProductList.tsx
│   │   └── Cart.tsx
│   ├── App.tsx
│   ├── main.tsx
│   └── vite-env.d.ts
├── src/__tests__/
│   └── shop.test.tsx
├── package.json
├── tsconfig.json
└── README.md
🛠 1. Redux va RTK Query sozlamalari
src/features/api/productsApi.ts
TypeScript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

// Generic/Utility type namunasi: Pick yordamida Product turidan kerakli qismini olish yoki moslash
export interface Product {
  id: number;
  title: string;
  price: number;
  description: string;
  category: string;
  image: string;
}

export type ProductSummary = Pick<Product, 'id' | 'title' | 'price'>;

export const productsApi = createApi({
  reducerPath: 'productsApi',
  baseQuery: fetchBaseQuery({ baseUrl: 'https://fakestoreapi.com' }),
  endpoints: (builder) => ({
    getProducts: builder.query<Product[], void>({
      query: () => '/products',
    }),
  }),
});

export const { useGetProductsQuery } = productsApi;
src/features/cart/cartSlice.ts
TypeScript
import { createSlice, PayloadAction } from '@reduxjs/toolkit';
import { Product } from '../api/productsApi';

export interface CartItem {
  product: Product;
  quantity: number;
}

interface CartState {
  items: CartItem[];
}

const initialState: CartState = {
  items: [],
};

export const cartSlice = createSlice({
  name: 'cart',
  initialState,
  reducers: {
    addToCart: (state, action: PayloadAction<Product>) => {
      const existingItem = state.items.find(
        (item) => item.product.id === action.payload.id
      );
      if (existingItem) {
        existingItem.quantity += 1;
      } else {
        state.items.push({ product: action.payload, quantity: 1 });
      }
    },
    removeFromCart: (state, action: PayloadAction<number>) => {
      state.items = state.items.filter(
        (item) => item.product.id !== action.payload
      );
    },
    updateQuantity: (
      state,
      action: PayloadAction<{ id: number; quantity: number }>
    ) => {
      const item = state.items.find(
        (i) => i.product.id === action.payload.id
      );
      if (item) {
        if (action.payload.quantity <= 0) {
          state.items = state.items.filter(
            (i) => i.product.id !== action.payload.id
          );
        } else {
          item.quantity = action.payload.quantity;
        }
      }
    },
  },
});

export const { addToCart, removeFromCart, updateQuantity } = cartSlice.actions;
export default cartSlice.reducer;
src/app/store.ts
TypeScript
import { configureStore } from '@reduxjs/toolkit';
import { productsApi } from '../features/api/productsApi';
import cartReducer from '../features/cart/cartSlice';

export const store = configureStore({
  reducer: {
    [productsApi.reducerPath]: productsApi.reducer,
    cart: cartReducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(productsApi.middleware),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
src/app/hooks.ts
TypeScript
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
🎨 2. Komponentlar
src/components/ProductList.tsx
TypeScript
import React from 'react';
import { useGetProductsQuery } from '../features/api/productsApi';
import { useAppDispatch } from '../app/hooks';
import { addToCart } from '../features/cart/cartSlice';

export const ProductList: React.FC = () => {
  const { data: products, isLoading, error } = useGetProductsQuery();
  const dispatch = useAppDispatch();

  if (isLoading) return <div>Yuklanmoqda...</div>;
  if (error) return <div>Xatolik yuz berdi!</div>;

  return (
    <div>
      <h2>Mahsulotlar</h2>
      <div className="product-grid">
        {products?.map((product) => (
          <div key={product.id} data-testid="product-item">
            <h3>{product.title}</h3>
            <p>${product.price}</p>
            <button onClick={() => dispatch(addToCart(product))}>
              Savatga qo'shish
            </button>
          </div>
        ))}
      </div>
    </div>
  );
};
src/components/Cart.tsx
TypeScript
import React from 'react';
import { useAppSelector, useAppDispatch } from '../app/hooks';
import { removeFromCart, updateQuantity } from '../features/cart/cartSlice';

export const Cart: React.FC = () => {
  const cartItems = useAppSelector((state) => state.cart.items);
  const dispatch = useAppDispatch();

  const totalSum = cartItems.reduce(
    (sum, item) => sum + item.product.price * item.quantity,
    0
  );

  return (
    <div>
      <h2>Savat</h2>
      {cartItems.length === 0 ? (
        <p>Savat bo'sh</p>
      ) : (
        <div>
          {cartItems.map((item) => (
            <div key={item.product.id} data-testid="cart-item">
              <span>{item.product.title}</span>
              <span>${item.product.price}</span>
              <input
                type="number"
                value={item.quantity}
                onChange={(e) =>
                  dispatch(
                    updateQuantity({
                      id: item.product.id,
                      quantity: Number(e.target.value),
                    })
                  )
                }
              />
              <button onClick={() => dispatch(removeFromCart(item.product.id))}>
                O'chirish
              </button>
            </div>
          ))}
          <h3 data-testid="total-sum">Jami: ${totalSum.toFixed(2)}</h3>
        </div>
      )}
    </div>
  );
};
🧪 3. Testlar (Vitest + React Testing Library)
src/__tests__/shop.test.tsx
TypeScript
import React from 'react';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Provider } from 'react-redux';
import { configureStore } from '@reduxjs/toolkit';
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';
import { describe, it, expect, beforeAll, afterEach, afterAll } from 'vitest';

import cartReducer from '../features/cart/cartSlice';
import { productsApi } from '../features/api/productsApi';
import { ProductList } from '../components/ProductList';
import { Cart } from '../components/Cart';

// MSW Serverni Sozlash
const server = setupServer(
  http.get('https://fakestoreapi.com/products', () => {
    return HttpResponse.json([
      { id: 1, title: 'Test Product 1', price: 100, description: 'Desc', category: 'cat', image: 'img' },
      { id: 2, title: 'Test Product 2', price: 200, description: 'Desc', category: 'cat', image: 'img' },
    ]);
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

const renderWithStore = (component: React.ReactElement) => {
  const store = configureStore({
    reducer: {
      [productsApi.reducerPath]: productsApi.reducer,
      cart: cartReducer,
    },
    middleware: (getDefaultMiddleware) =>
      getDefaultMiddleware().concat(productsApi.middleware),
  });
  return { ...render(<Provider store={store}>{component}</Provider>) };
};

describe('Shop App Tests', () => {
  // 1. Komponent render bo'lishini tekshiruvchi test
  it('ProductList renders correctly with loading state initially', () => {
    renderWithStore(<ProductList />);
    expect(screen.getByText(/yuklanmoqda.../i)).toBeDefined();
  });

  // 2. Mock qilingan fetch bilan async yuklashni tekshiruvchi test (findBy)
  it('loads and displays products asynchronously', async () => {
    renderWithStore(<ProductList />);
    const productTitle = await screen.findByText('Test Product 1');
    expect(productTitle).toBeDefined();
  });

  // 3. userEvent bilan "savatga qo'shish" tugmasini bosish testi
  it('adds product to cart when button is clicked', async () => {
    renderWithStore(
      <div>
        <ProductList />
        <Cart />
      </div>
    );

    const addToCartButtons = await screen.findAllByText("Savatga qo'shish");
    await userEvent.click(addToCartButtons[0]);

    const cartItem = screen.getByTestId('cart-item');
    expect(cartItem).toBeDefined();
    expect(screen.getByText('Test Product 1')).toBeDefined();
  });

  // 4. Savat jami summasi to'g'ri hisoblanishini tekshiruvchi test
  it('calculates the total sum of the cart correctly', async () => {
    renderWithStore(
      <div>
        <ProductList />
        <Cart />
      </div>
    );

    const addToCartButtons = await screen.findAllByText("Savatga qo'shish");
    await userEvent.click(addToCartButtons[0]); // 100
    await userEvent.click(addToCartButtons[1]); // 200

    const totalSumElement = screen.getByTestId('total-sum');
    expect(totalSumElement.textContent).toContain('300');
  });

  // 5. Xato holatini tekshiruvchi test (server 500 qaytarsa)
  it('handles server error (500) gracefully', async () => {
    server.use(
      http.get('https://fakestoreapi.com/products', () => {
        return new HttpResponse(null, { status: 500 });
      })
    );

    renderWithStore(<ProductList />);
    const errorMessage = await screen.findByText(/xatolik yuz berdi!/i);
    expect(errorMessage).toBeDefined();
  });
});
📖 4. README.md
Markdown
# React + TypeScript E-Commerce Loyihasi

Ushbu loyiha React, TypeScript, Redux Toolkit (RTK Query) va Vitest texnologiyalari yordamida qurilgan bo'lib, to'liq tiplangan va test qilingan savat hamda mahsulotlar ro'yxati funksionalligini o'z ichiga oladi.

## 🛠 Texnologiyalar
- **React** (.tsx)
- **TypeScript**
- **Redux Toolkit & RTK Query**
- **Vitest & React Testing Library** (MSW bilan birga)

---

## 🚀 Loyihani ishga tushirish

1. **Bog'liqliklarni o'rnatish:**
   ```bash
   npm install
Loyihani lokal serverda ishga tushirish:

Bash
npm run dev
🧪 Testlarni ishga tushirish
Barcha yozilgan 5 ta testni (render, async yuklash, userEvent, jami summa va 500 server xatosi) ishga tushirish uchun:

Bash
npm run test
