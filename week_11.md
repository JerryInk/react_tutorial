## Лекция 11.1: Маршрутизация с React Router DOM (v6+)

* Длительность: 50 минут
* Сложность: Средняя
* Цель: Научиться настраивать навигацию, работать с динамическими параметрами URL (например, id товара) и защищать приватные страницы (Guards).

------------------------------
## 1. Теоретический блок (25 минут)
В Angular роутер встроен в ядро фреймворка. В React мы устанавливаем внешнюю библиотеку:
```bash
npm install react-router-dom
```
В современной версии React Router (v6+) используется декларативный подход на базе объектов или компонентов для описания карты путей.
## Основные компоненты и понятия:

   1. ```BrowserRouter```: Обертка (провайдер), которая подключает приложение к истории браузера.
   2. ```Routes``` и ```Route```: Контейнер и описание конкретного маршрута. Route связывает текстовый путь (```path="/about"```) с конкретным компонентом (```element={<About />}```).
   3. ```Link```: Компонент для навигации. Заменяет стандартный тег ```<a href="...">```. При клике страница не перезагружается — React Router просто подменяет компонент на экране.
   4. Динамические параметры (```:id```): Позволяют создавать универсальные страницы (например, ```/user/:id```). Считать этот ```id``` внутри компонента можно с помощью хука ```useParams```.
   5. Программный переход (```useNavigate```): Хук, который возвращает функцию для перенаправления пользователя через код (например, после успешной отправки формы).

------------------------------
## 2. Практический блок (20 минут)
Создадим приложение с тремя страницами: Главная, Профиль пользователя (динамическая) и Панель админа (приватная).
## Шаг 1. Создаем страницы приложения
Создайте три простых компонента в папке ```src/pages/```:
```src/pages/Home.jsx```:
```jsx
function Home() {
  return <h2>🏠 Главная страница приложения</h2>;
}
export default Home;
```
```src/pages/UserProfile.jsx``` (Динамическая страница):
```jsx
import { useParams } from 'react-router-dom';

function UserProfile() {
  // Хук достает двоеточие из пути (из :id)
  const { id } = useParams();

  return (
    <div>
      <h2>👤 Профиль пользователя</h2>
      <p>Вы просматриваете данные пользователя с ID: <strong>{id}</strong></p>
    </div>
  );
}
export default UserProfile;
```
```src/pages/Admin.jsx``` (Приватная страница):
```jsx
function Admin() {
  return <h2 style={{ color: 'red' }}>🔒 Секретная панель администратора</h2>;
}
export default Admin;
```
## Шаг 2. Создаем защитник маршрутов (Route Guard)
В React нет специального интерфейса ```CanActivate```, как в Angular. Защита страниц пишется как обычный компонент-обертка:
Создайте файл ```src/components/ProtectedRoute.jsx```:
```jsx
import { Navigate } from 'react-router-dom';

function ProtectedRoute({ isAllowed, children }) {
  // Если доступа нет, принудительно перенаправляем на главную
  if (!isAllowed) {
    return <Navigate to="/" replace />;
  }

  // Если доступ есть, рендерим защищенную страницу
  return children;
}
export default ProtectedRoute;
```
## Шаг 3. Настраиваем роутер в ```src/App.jsx```
Собираем все страницы вместе, настраиваем навигацию и защиту:
```jsx
import { BrowserRouter, Routes, Route, Link, useNavigate } from 'react-router-dom';
import { useState } from 'react';
import Home from './pages/Home';
import UserProfile from './pages/UserProfile';
import Admin from './pages/Admin';
import ProtectedRoute from './components/ProtectedRoute';

function App() {
  // Имитируем стейт авторизации админа
  const [isAdmin, setIsAdmin] = useState(false);
  
  return (
    <BrowserRouter>
      <div style={{ maxWidth: '600px', margin: '30px auto', fontFamily: 'sans-serif' }}>
        
        {/* Шапка с навигационными ссылками */}
        <nav style={{ display: 'flex', gap: '15px', marginBottom: '20px' }}>
          <Link to="/">Главная</Link>
          <Link to="/user/42">Профиль №42</Link>
          <Link to="/user/107">Профиль №107</Link>
          <Link to="/admin">Админка</Link>
        </nav>

        {/* Кнопка изменения прав прямо в UI */}
        <button onClick={() => setIsAdmin(!isAdmin)} style={{ marginBottom: '20px' }}>
          {isAdmin ? '🔴 Выйти из админа' : '🟢 Войти как админ'}
        </button>

        <hr />

        {/* Карта маршрутов */}
        <Routes>
          <Route path="/" element={<Home />} />
          
          {/* Динамический роутинг */}
          <Route path="/user/:id" element={<UserProfile />} />
          
          {/* Защищенный роутинг */}
          <Route path="/admin" element={
            <ProtectedRoute isAllowed={isAdmin}>
              <Admin />
            </ProtectedRoute>
          } />

          {/* 404 Страница (если путь не найден) */}
          <Route path="*" element={<h2>⚠️ Страница не найдена (404)</h2>} />
        </Routes>

        <hr style={{ marginTop: '30px' }} />
        <NavigationHelper />
      </div>
    </BrowserRouter>
  );
}

// Вспомогательный компонент для демонстрации хука useNavigate
function NavigationHelper() {
  const navigate = useNavigate();

  const handleRandomRedirect = () => {
    const randomId = Math.floor(Math.random() * 1000);
    // Программный переход в коде
    navigate(`/user/${randomId}`);
  };

  return (
    <button onClick={handleRandomRedirect} style={{ background: '#ddd', padding: '10px' }}>
      🎲 Перейти на случайного пользователя
    </button>
  );
}

export default App;
```
Обратите внимание: Компонент ```NavigationHelper``` вынесен отдельно, так как хук ```useNavigate``` можно вызывать только внутри компонента, который физически находится под оберткой ```<BrowserRouter>```!

------------------------------
## 3. Чек-лист для самопроверки (3 минуты)

* Понимаете, почему стандартный тег ```<a href="...">``` ломает концепцию SPA в React-приложении.
* Знаете, какой хук отвечает за чтение параметров из строки вида ```/product/:productId```.
* Можете объяснить, как работает программный переход с помощью ```useNavigate```.
* Понимаете, как символ ```*``` в путях помогает обрабатывать ошибку 404.

------------------------------
## 4. Домашнее задание (2 минуты)

   1. Создать страницу «Каталог» (```/catalog```) со списком из 3-4 товаров.
   2. Настроить ссылки: Каждый товар в списке должен быть обернут в ```<Link>```, ведущий на страницу ```/product/:id```.
   3. Создать страницу товара: Написать компонент ```ProductPage.jsx```, который считывает ```id``` из URL через ```useParams``` и выводит текст: ```«Вы смотрите товар с артикулом Х»```.
   4. Добавить кнопку «Назад»: Использовать ```useNavigate``` со значением ```-1``` (```navigate(-1)```), чтобы при клике пользователя возвращало на предыдущую страницу истории браузера.



------------------------------
## Лекция 11.2: Работа с Query-параметрами через ```useSearchParams```

* Длительность: 40 минут
* Сложность: Средняя
* Цель: Научиться читать, изменять и удалять параметры из строки запроса (```?key=value```) без перезагрузки страницы.

------------------------------
## 1. Теоретический блок (20 минут)
В отличие от динамических параметров пути (```/user/:id```), которые жестко зашиты в карту маршрутов Route, Query-параметры являются необязательными и могут быть абсолютно любыми.
Для работы с ними в react-router-dom есть специальный хук — ```useSearchParams```. По своей механике он очень похож на стандартный ```useState```, так как возвращает массив из двух элементов:
```jsx
const [searchParams, setSearchParams] = useSearchParams();
```

   1. ```searchParams``` — это объект (экземпляр встроенного в браузер класса ```URLSearchParams```), из которого можно читать данные с помощью метода ```.get('ключ')```.
   2. ```setSearchParams``` — функция, которая обновляет строку запроса в URL-адресе браузера, что автоматически запускает ререндер компонента с новыми значениями.

## Важные нюансы:

* Метод ```searchParams.get('key')``` всегда возвращает строку или ```null```, если параметра нет. Если вам нужно число или булево значение, их придется приводить к нужному типу вручную (```Number()```, ```=== 'true'```).
* Функция ```setSearchParams``` полностью перезаписывает строку URL. Если у вас в строке было три параметра, а вы передали один: ```setSearchParams({ sort: 'desc' })```, то остальные два параметра сотрутся. Чтобы этого избежать, нужно использовать копирование текущих параметров.

------------------------------
## 2. Практический блок (15 минут)
Создадим страницу каталога товаров с живым поиском и сортировкой по цене, которые полностью синхронизированы с адресной строкой браузера.
## Шаг 1. Создаем компонент каталога ```src/pages/Catalog.jsx```
Создайте файл и вставьте следующий код. Обратите внимание на то, как мы соединяем инпуты с URL:
```jsx
import { useSearchParams } from 'react-router-dom';

// Фейковый массив товаров для демонстрации
const PRODUCTS = [
  { id: 1, title: 'iPhone 15 Pro', price: 120000 },
  { id: 2, title: 'Samsung Galaxy S24', price: 95000 },
  { id: 3, title: 'Xiaomi 14 Ultra', price: 85000 },
  { id: 4, title: 'iPhone 13', price: 60000 },
];

function Catalog() {
  // Инициализируем хук для работы с Query-параметрами
  const [searchParams, setSearchParams] = useSearchParams();

  // 1. Читаем текущие значения из URL (если их нет, ставим дефолты)
  const searchQuery = searchParams.get('search') || '';
  const sortOrder = searchParams.get('sort') || 'asc'; // 'asc' (по возрастанию) или 'desc' (по убыванию)

  // 2. Функция для обновления конкретного параметра без потери остальных
  const updateQueryParam = (key, value) => {
    // Создаем копию текущих параметров на основе существующего URL
    const newParams = new URLSearchParams(searchParams);
    
    if (value) {
      newParams.set(key, value); // Добавляем или обновляем параметр
    } else {
      newParams.delete(key); // Если значение пустое (очистили инпут), удаляем ключ из URL
    }
    
    setSearchParams(newParams); // Обновляем строку браузера
  };

  // 3. Логика фильтрации и сортировки массива на основе данных из URL
  // (В реальном проекте здесь мог бы быть useMemo из Лекции 7, но пишем "в лоб" для простоты)
  const filteredProducts = PRODUCTS.filter(product =>
    product.title.toLowerCase().includes(searchQuery.toLowerCase())
  ).sort((a, b) => {
    return sortOrder === 'asc' ? a.price - b.price : b.price - a.price;
  });

  return (
    <div style={{ padding: '15px', border: '1px solid #ddd', borderRadius: '8px' }}>
      <h2>📦 Каталог товаров с Query-фильтрами</h2>

      {/* Блок управления фильтрами */}
      <div style={{ display: 'flex', gap: '15px', marginBottom: '20px' }}>
        
        {/* Инпут поиска напрямую меняет URL при вводе */}
        <input
          type="text"
          value={searchQuery}
          onChange={(e) => updateQueryParam('search', e.target.value)}
          placeholder="Поиск товара..."
          style={{ padding: '6px', flexGrow: 1 }}
        />

        {/* Селект сортировки напрямую меняет URL */}
        <select 
          value={sortOrder} 
          onChange={(e) => updateQueryParam('sort', e.target.value)}
          style={{ padding: '6px' }}
        >
          <option value="asc">Сначала дешевые</option>
          <option value="desc">Сначала дорогие</option>
        </select>
      </div>

      {/* Вывод результатов */}
      {filteredProducts.length > 0 ? (
        <ul style={{ listStyle: 'none', padding: 0 }}>
          {filteredProducts.map(product => (
            <li key={product.id} style={{ padding: '10px 0', borderBottom: '1px solid #eee' }}>
              <strong>{product.title}</strong> — {product.price.toLocaleString()} руб.
            </li>
          ))}
        </ul>
      ) : (
        <p style={{ color: 'gray' }}>Товары не найдены.</p>
      )}
    </div>
  );
}

export default Catalog;
```
## Шаг 2. Подключаем страницу в ```src/App.jsx```
Добавим новый маршрут для нашего каталога:
```jsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';
import Catalog from './pages/Catalog';

function App() {
  return (
    <BrowserRouter>
      <div style={{ maxWidth: '500px', margin: '40px auto', fontFamily: 'sans-serif' }}>
        <nav style={{ marginBottom: '20px', display: 'flex', gap: '15px' }}>
          <Link to="/">На главную</Link>
          {/* Ссылка сразу с предустановленными фильтрами для проверки */}
          <Link to="/catalog?search=iphone&sort=desc">Раздел iPhone (Дорогие)</Link>
          <Link to="/catalog">Чистый каталог</Link>
        </nav>
        
        <Routes>
          <Route path="/" element={<h2>🏠 Главная страница</h2>} />
          <Route path="/catalog" element={<Catalog />} />
        </Routes>
      </div>
    </BrowserRouter>
  );
}

export default App;
```
------------------------------
## 3. Чек-лист для самопроверки (3 минуты)

* Понимаете разницу между динамическим путем ```/catalog/iphone``` и строкой запроса ```/catalog?search=iphone```.
* Знаете, почему код ```searchParams.get('page') + 1``` может превратить страницу 1 в строку 11 (проблема типов).
* Помните, почему прямая передача объекта ```setSearchParams({ search: 'text' })``` сотрет все остальные параметры из URL.
* Умеете использовать класс ```URLSearchParams``` для безопасного точечного обновления ключей.

------------------------------
## 4. Домашнее задание (2 минуты)

   1. Добавить пагинацию в URL: Добавьте в компонент ```Catalog.jsx``` две кнопки: «Предыдущая страница» и «Следующая страница».
   2. Внедрить логику: При клике на кнопки в URL должен появляться или изменяться параметр ```&page=X```.
   3. Реализовать сброс: Сделайте так, чтобы при вводе текста в инпут поиска параметр ```page``` автоматически удалялся из URL (сбрасывался на первую страницу), чтобы избежать багов отображения, когда на 5-й странице результатов поиска просто нет.
