## Лекция 13: Управление глобальным состоянием с помощью Zustand

* Длительность: 45 минут
* Сложность: Средняя
* Цель: Научиться создавать централизованные хранилища данных (Stores), изменять состояние без мутаций и точечно подписывать компоненты на обновления для исключения лишних ререндеров.

------------------------------
## 1. Теоретический блок (25 минут)
Для начала установим библиотеку:
```bash
npm install zustand
```
## Почему Zustand, а не Context или Redux?

* Против Context API: Контекст при изменении данных ререндерит всех своих потребителей. Zustand позволяет компоненту подписаться на конкретное поле (селектор). Компонент перерисуется только тогда, когда изменится именно это поле.
* Против Redux: Для Redux нужно создавать экшены, редюсеры, типы и провайдеры. В Zustand всё хранилище — это одна простая функция. Вам больше не нужно оборачивать все приложение в ```<Provider>```.

## Основные концепции
Вся магия Zustand строится вокруг функции ```create```, которая принимает колбэк с двумя главными инструментами:

   1. ```set```: Функция для обновления состояния. Она автоматически делает поверхностное слияние (Shallow Merge) для объектов, поэтому вам не нужно постоянно писать ```...state``` для сохранения остальных полей первого уровня.
   2. ```get```: Функция (необязательная), которая позволяет прочитать текущее состояние прямо внутри экшена (например, для проверки условий перед запросом).

## Правило иммутабельности
Хотя ```set``` делает слияние объектов верхнего уровня за вас, вложенные массивы и объекты всё еще нельзя мутировать напрямую (принцип чистых функций).

* Неправильно: ```set((state) => { state.todos.push(newTodo) })```
* Правильно: ```set((state) => ({ todos: [...state.todos, newTodo] }))```

------------------------------
## 2. Практический блок (15 минут)
Создадим глобальное хранилище для корзины интернет-магазина. Любой компонент на любой глубине сможет положить товар в корзину или узнать общее количество товаров.

## Шаг 1. Создаем глобальное хранилище ```src/store/useCartStore.js```
Создайте файл (обратите внимание, расширение ```.js``` или ```.ts```, так как JSX разметки тут нет):
```javascript
import { create } from 'zustand';
// Создаем хук-хранилище с помощью функции createexport const useCartStore = create((set) => ({
  // 1. Исходное состояние (State)
  cartItems: [],

  // 2. Действия для изменения состояния (Actions)
  addToCart: (product) => set((state) => {
    // Проверяем, есть ли уже такой товар в корзине
    const exists = state.cartItems.find(item => item.id === product.id);

    if (exists) {
      // Если есть, увеличиваем его количество (сохраняя иммутабельность)
      return {
        cartItems: state.cartItems.map(item =>
          item.id === product.id ? { ...item, quantity: item.quantity + 1 } : item
        )
      };
    }

    // Если товара нет, добавляем его с количеством 1
    return { cartItems: [...state.cartItems, { ...product, quantity: 1 }] };
  }),

  removeFromCart: (productId) => set((state) => ({
    cartItems: state.cartItems.filter(item => item.id !== productId)
  })),

  clearCart: () => set({ cartItems: [] }) // Просто передаем объект для полного сброса
}));
```
## Шаг 2. Создаем компонент списка товаров ```src/components/ProductList.jsx```
Компонент просто забирает экшен ```addToCart``` из хранилища и вызывает его при клике:
```jsx
import { useCartStore } from '../store/useCartStore';

const FAKE_PRODUCTS = [
  { id: 201, title: 'Кроссовки беговые', price: 5500 },
  { id: 202, title: 'Спортивная бутылка', price: 800 },
  { id: 203, title: 'Умные часы', price: 12000 }
];

function ProductList() {
  // Забираем только нужную функцию. Пишем селектор: state => state.addToCart
  const addToCart = useCartStore(state => state.addToCart);

  return (
    <div>
      <h3>👟 Витрина товаров</h3>
      <div style={{ display: 'flex', flexDirection: 'column', gap: '10px' }}>
        {FAKE_PRODUCTS.map(product => (
          <div key={product.id} style={{ border: '1px solid #eee', padding: '10px', display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
            <span>{product.title} ({product.price} руб.)</span>
            <button onClick={() => addToCart(product)}>В корзину</button>
          </div>
        ))}
      </div>
    </div>
  );
}

export default ProductList;
```
## Шаг 3. Создаем компонент виджета корзины ```src/components/CartWidget.jsx```
Этот компонент будет выводить список добавленных товаров и считать общую сумму:
```jsx
import { useCartStore } from '../store/useCartStore';

function CartWidget() {
  // Точечно подписываемся на массив cartItems. 
  // Компонент перерисуется ТОЛЬКО когда изменится этот массив.
  const cartItems = useCartStore(state => state.cartItems);
  const clearCart = useCartStore(state => state.clearCart);

  const totalPrice = cartItems.reduce((sum, item) => sum + (item.price * item.quantity), 0);

  return (
    <div style={{ background: '#f5f5f5', padding: '15px', borderRadius: '6px', marginTop: '20px' }}>
      <h3>🛒 Ваша корзина</h3>
      {cartItems.length === 0 ? (
        <p style={{ color: 'gray' }}>Корзина пуста</p>
      ) : (
        <>
          <ul style={{ paddingLeft: '20px' }}>
            {cartItems.map(item => (
              <li key={item.id} style={{ marginBottom: '5px' }}>
                {item.title} — {item.quantity} шт. х {item.price} руб.
              </li>
            ))}
          </ul>
          <p><strong>Итоговая сумма: {totalPrice} руб.</strong></p>
          <button onClick={clearCart} style={{ background: '#ff4d4f', color: '#fff', border: 'none', padding: '6px 12px', borderRadius: '4px', cursor: 'pointer' }}>
            Очистить корзину
          </button>
        </>
      )}
    </div>
  );
}

export default CartWidget;
```
## Шаг 4. Подключаем всё в ```src/App.jsx```
Никаких провайдеров не нужно. Просто импортируем и рендерим компоненты:
```jsx
import ProductList from './components/ProductList';
import CartWidget from './components/CartWidget';

function App() {
  return (
    <div style={{ maxWidth: '450px', margin: '40px auto', fontFamily: 'sans-serif' }}>
      <h2>Магазин на Zustand</h2>
      <hr style={{ margin: '20px 0' }} />
      <ProductList />
      <CartWidget />
    </div>
  );
}

export default App;
```
------------------------------
## 3. Чек-лист для самопроверки (3 минуты)

* Понимаете, почему для Zustand не нужно оборачивать приложение в ```<CartStore.Provider>```.
* Знаете, зачем в вызове ```useCartStore(state => state.cartItems)``` нужна стрелочная функция (селектор).
* Помните, делает ли функция ```set``` глубокое (вложенное) автоматическое слияние объектов.
* Понимаете, почему Zustand производительнее стандартного React Context.

------------------------------
## 4. Домашнее задание (2 минуты)

   1. Добавить функцию удаления: В файл ```useCartStore.js``` добавить экшен ```decreaseQuantity: (productId) => set(...)```.
   2. Внедрить логику:
      * Функция должна находить товар в корзине.
      * Если его ```quantity > 1```, она должна уменьшать количество на 1.
      * Если ```quantity === 1```, она должна полностью удалять товар из массива (как экшен ```removeFromCart```).
   3. Вывести в UI: Добавить кнопку [-] рядом с каждым товаром в корзине и привязать к ней созданный экшен.
