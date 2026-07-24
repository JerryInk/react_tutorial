## Лекция 9: Кастомные хуки и переиспользование логики

* Длительность: 45 минут
* Сложность: Средняя
* Цель: Научиться извлекать повторяющуюся бизнес-логику из UI-компонентов в отдельные переиспользуемые функции — кастомные хуки.

------------------------------
## 1. Теоретический блок (25 минут)
В Angular для переиспользования логики (например, работы с API или локальным хранилищем) используются сервисы с декоратором ```@Injectable()```. В React эту роль выполняют кастомные хуки (Custom Hooks).

## Что такое кастомный хук?
Кастомный хук — это обычная JavaScript-функция, имя которой обязательно начинается с префикса ```use``` (например, ```useFetch```, ```useAuth```).

* Главная особенность: внутри кастомного хука можно вызывать другие встроенные хуки React (```useState```, ```useEffect```, ```useRef``` и т.д.).
* Кастомные хуки позволяют полностью отделить логику (как данные загружаются/хранятся) от представления (как эти данные отображаются на экране в JSX).

## Важное правило: Изоляция состояния
Кастомный хук переиспользует логику, но не само состояние.

* Если вы подключите хук ```useLocalStorage``` в пяти разных компонентах, у каждого из них будет свое собственное, независимое состояние. Они не будут синхронизироваться друг с другом автоматически, если они не завязаны на общий Context API.

------------------------------
## 2. Практический блок (15 минут)
## Шаг 1. Создаем кастомный хук ```useLocalStorage```
Создадим файл ```src/hooks/useLocalStorage.js```. Этот хук будет автоматически сохранять любое состояние в браузерное хранилище при его изменении:
```javascript
import { useState, useEffect } from 'react';
// Хук принимает ключ для localStorage и начальное значение

export function useLocalStorage(key, initialValue) {
  // Инициализируем стейт. Передаем функцию в useState, 
  // чтобы тяжелое чтение из localStorage выполнялось только 1 раз при старте
  const [value, setValue] = useState(() => {
    try {
      const localValue = localStorage.getItem(key);
      return localValue ? JSON.parse(localValue) : initialValue;
    } catch (error) {
      console.error("Ошибка чтения localStorage:", error);
      return initialValue;
    }
  });

  // Следим за изменением value или key и синхронизируем с localStorage
  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  // Возвращаем массив в стиле стандартного useState
  return [value, setValue];
}
```
## Шаг 2. Создаем кастомный хук ```useFetch```
Создадим файл ```src/hooks/useFetch.js``` для удобной работы с сетевыми запросами, обработки загрузки и ошибок:
```javascript
import { useState, useEffect } from 'react';

export function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  // Используем технику синхронизации состояния во время рендера.
  // Храним в памяти предыдущий URL, чтобы поймать момент его изменения
  const [prevUrl, setPrevUrl] = useState(url);

  if (url !== prevUrl) {
    setPrevUrl(url);
    setLoading(true);
    setData(null);
    setError(null);
  }

  useEffect(() => {
    // Внутри самого эффекта синхронный вызов setLoading(true) больше не нужен!
    const fetchData = async () => {
      try {
        const response = await fetch(url);
        if (!response.ok) throw new Error('Ошибка сети');
        const json = await response.json();
        setData(json);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false); // Асинхронный вызов внутри колбэка разрешен и безопасен
      }
    };

    fetchData();
  }, [url]);

  return { data, loading, error };
}
```
## Шаг 3. Используем кастомные хуки в компонентах
Теперь наши компоненты станут невероятно тонкими и чистыми.
Создадим ```src/components/ThemeSettings.jsx``` (использует первый хук):
```jsx
import { useLocalStorage } from '../hooks/useLocalStorage';

function ThemeSettings() {
  // Используем наш хук вместо обычного useState
  const [theme, setTheme] = useLocalStorage('app-theme', 'light');

  return (
    <div style={{ padding: '10px', background: theme === 'dark' ? '#333' : '#fff', color: theme === 'dark' ? '#fff' : '#000' }}>
      <p>Текущая тема: {theme}</p>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Переключить тему
      </button>
    </div>
  );
}
export default ThemeSettings;
```
Создадим ```src/components/PostList.jsx``` (использует второй хук):
```jsx
import { useFetch } from '../hooks/useFetch';

function PostList() {
  const { data: posts, loading, error } = useFetch('https://typicode.com');

  if (loading) return <p>Загрузка постов...</p>;
  if (error) return <p style={{ color: 'red' }}>Ошибка: {error}</p>;

  return (
    <ul>
      {posts && posts.map(post => (
        <li key={post.id}>
          <strong>{post.title}</strong>
        </li>
      ))}
    </ul>
  );
}
export default PostList;
```
## Шаг 4. Подключение в ```App.jsx```
```jsx
import ThemeSettings from './components/ThemeSettings';
import PostList from './components/PostList';

function App() {
  return (
    <div style={{ maxWidth: '500px', margin: '40px auto', fontFamily: 'sans-serif' }}>
      <h1>Кастомные хуки React</h1>
      <ThemeSettings />
      <hr style={{ margin: '20px 0' }} />
      <h3>Список постов с сервера:</h3>
      <PostList />
    </div>
  );
}
export default App;
```
------------------------------
## 3. Чек-лист для самопроверки (3 минуты)

* Понимаете, почему имя хука ```fetchData()``` вызовет ошибку линтера, а ```useFetchData()``` — нет.
* Помните, делят ли компоненты между собой стейт, если вызывают один и тот же кастомный хук.
* Знаете, в каком формате кастомный хук может возвращать данные (ограничений нет: массив, объект, строка, число).

------------------------------
## 4. Домашнее задание (2 минуты)

   1. Создать хук ```useWindowSize```: Написать кастомный хук, который отслеживает ширину и высоту экрана браузера.
   2. Внедрить логику: Внутри хука использовать ```useState``` для размеров и ```useEffect``` со слушателем события ```window.addEventListener('resize', ...)```. Не забудьте вернуть функцию очистки!
   3. Применить в UI: Использовать хук в компоненте, чтобы выводить на экран плашку «Мобильная версия», если ширина экрана становится меньше 768 пикселей.
