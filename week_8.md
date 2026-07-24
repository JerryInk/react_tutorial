## Лекция 8: Глобальное состояние без сторонних библиотек

* Длительность: 45 минут
* Сложность: Средняя
* Цель: Решить проблему глубокого проброса пропсов (Prop Drilling) с помощью встроенного Context API и научиться управлять сложным состоянием через хук ```useReducer```.

------------------------------
## 1. Теоретический блок (25 минут)
## Проблема Prop Drilling
В Angular для передачи данных между несвязанными компонентами используются сервисы со встроенным Dependency Injection (DI). В React данные идут строго сверху вниз.

* Если данные из корневого компонента ```App``` нужны компоненту на 5 уровней ниже, вам придется передавать их через все промежуточные компоненты как пропсы. Это называется Prop Drilling (проброс пропсов).
* Промежуточным компонентам эти данные не нужны, они просто работают «транспортом», что сильно загрязняет код.

## Решение: React Context API
Context предоставляет способ передавать данные через дерево компонентов без необходимости пробрасывать пропсы вручную на каждом уровне. Это локальный аналог DI-контейнера.
Процесс работы с контекстом состоит из трех шагов:

   1. Инициализация: Создание объекта контекста с помощью ```React.createContext()```.
   2. Провайдер (Provider): Компонент-обертка, который спускает данные вниз по дереву для всех своих детей.
   3. Потребитель (Consumer): Любой дочерний компонент (независимо от глубины вложенности) может мгновенно достать эти данные с помощью хука ```useContext```.

## Хук ```useReducer```
Когда состояние становится сложным (например, корзина интернет-магазина с добавлением, удалением, очисткой и промокодами), обычного ```useState``` начинает не хватать. Логика обновлений размазывается по всему компоненту.
Хук ```useReducer``` — это встроенная альтернатива ```useState``` для управления комплексной логикой. Он работает по паттерну Redux:

* State: Текущее состояние (объект).
* Action: Объект, описывающий, что произошло (например, ```{ type: 'ADD_TODO', payload: 'Купить молоко' }```).
* Reducer: Чистая функция, которая принимает текущий ```state```, объект ```action``` и возвращает новое состояние.
* Dispatch: Функция, с помощью которой мы отправляем (диспатчим) экшены в редюсер.

Термин «чистая функция» (Pure Function) пришел из функционального программирования. В контексте React и редюсеров это означает, что функция обязана строго соблюдать три главных правила:

## 1. Детерминированность (Предсказуемость)
При одних и тех же входных аргументах функция всегда должна возвращать один и тот же результат. Внутри нее не должно быть случайности или зависимости от внешнего мира.

* Грязная функция (Нельзя использовать в редюсере):
```jsx
function reducer(state, action) {
  // Дата каждый раз разная, а ID случайный. Результат непредсказуем!
  return [...state, { id: Math.random(), date: new Date() }]; 
}
```
* Чистая функция (Правильно):
```jsx
function reducer(state, action) {
  // Все данные для новой задачи пришли снаружи в action.payload. 
  // Результат строго предсказуем.
  return [...state, { id: action.payload.id, text: action.payload.text }]; 
}
```

## 2. Отсутствие побочных эффектов (Side Effects)
Чистая функция не имеет права изменять что-либо за пределами своих фигурных скобок. Внутри редюсера категорически запрещено:

* Делать запросы к серверу (```fetch```, ```axios```).
* Записывать данные в ```localStorage```.
* Изменять глобальные переменные.
* Выводить логи в консоль (технически ```console.log``` — это тоже побочный эффект, хотя на работу React он не влияет).

Редюсер должен только принимать старый стейт, экшен и молча возвращать новый стейт.

## 3. Иммутабельность (Неизменяемость входных данных)
Чистая функция не имеет права мутировать (изменять) свои аргументы. Она должна относиться к ним как к защищенным от записи.

* Грязная функция (Сломает React):
```jsx
function reducer(state, action) {
  state.push(action.payload); // МУТАЦИЯ! Изменили исходный массив напрямую.
  return state; // Ссылка на массив не изменилась, React не сделает ререндер.
}
```
* Чистая функция (Правильно):
```jsx
function reducer(state, action) {
  return [...state, action.payload]; // Создали абсолютно новый массив (копию), не трогая старый.
}
```

## Зачем React требует, чтобы редюсер был чистым?
Если функция чистая, React может в любой момент безопасно отменить рендеринг, оптимизировать вычисления, повторно вызвать редюсер или реализовать функцию «Time Travel» (отмотку состояний назад/вперед в DevTools). Если редюсер будет мутировать данные или делать сетевые запросы, приложение станет хаотичным и покроется трудноуловимыми багами.

------------------------------
## 2. Практический блок (15 минут)
Создадим мини-приложение: список задач с глобальным доступом через контекст и управлением логикой через ```useReducer```.

## Шаг 1. Создаем контекст и редюсер
Создайте файл ```src/context/TodoContext.jsx```. Здесь мы создадим контекст:
```jsx
import { createContext } from 'react';

// 1. Создаем сам контекст
const TodoContext = createContext(null);
```

Создайте файл ```src/context/TodoProvider.jsx```. Здесь мы опишем всю бизнес-логику управления данными:
```jsx
import { useReducer } from 'react';
import { TodoContext } from "./TodoContext";

// 2. Чистая функция-редюсер, управляющая состояниями
function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD':
      return [...state, { id: Date.now(), text: action.payload, completed: false }];
    case 'TOGGLE':
      return state.map(todo => 
        todo.id === action.payload ? { ...todo, completed: !todo.completed } : todo
      );
    case 'DELETE':
      return state.filter(todo => todo.id !== action.payload);
    default:
      return state;
  }
}

// 3. Компонент-Провайдер, который объединяет редюсер и контекст
export function TodoProvider({ children }) {
  const [todos, dispatch] = useReducer(todoReducer, [
    { id: 1, text: 'Изучить Context API', completed: false }
  ]);

  return (
    <TodoContext.Provider value={{ todos, dispatch }}>
      {children}
    </TodoContext.Provider>
  );
}
```

## Шаг 2. Создаем компоненты формы и списка задач
Обратите внимание: мы больше не передаем пропсы. Компоненты сами забирают нужные инструменты из контекста.
Создайте файл ```src/components/TodoForm.jsx``` для добавления задач:
```jsx
import { useState, useContext } from 'react';
import { TodoContext } from '../context/TodoContext';

function TodoForm() {
  const [text, setText] = useState('');
  const { dispatch } = useContext(TodoContext); // Забираем dispatch из глобального контекста

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!text.trim()) return;
    
    // Отправляем экшен в редюсер
    dispatch({ type: 'ADD', payload: text });
    setText('');
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" value={text} onChange={e => setText(e.target.value)} placeholder="Новое дело..." />
      <button type="submit" style={{ marginLeft: '5px' }}>Добавить</button>
    </form>
  );
}
export default TodoForm;
```
Создайте файл ```src/components/TodoList.jsx``` для вывода и управления задачами:
```jsx
import { useContext } from 'react';
import { TodoContext } from '../context/TodoContext';

function TodoList() {
  const { todos, dispatch } = useContext(TodoContext); // Забираем состояние и dispatch из контекста

  return (
    <ul style={{ padding: 0, listStyle: 'none', marginTop: '15px' }}>
      {todos.map(todo => (
        <li key={todo.id} style={{ display: 'flex', alignItems: 'center', marginBottom: '8px' }}>
          <input 
            type="checkbox" 
            checked={todo.completed} 
            onChange={() => dispatch({ type: 'TOGGLE', payload: todo.id })} 
          />
          <span style={{ textDecoration: todo.completed ? 'line-through' : 'none', marginLeft: '8px', flexGrow: 1 }}>
            {todo.text}
          </span>
          <button onClick={() => dispatch({ type: 'DELETE', payload: todo.id })} style={{ color: 'red' }}>
            Удалить
          </button>
        </li>
      ))}
    </ul>
  );
}
export default TodoList;
```
## Шаг 3. Оборачиваем приложение провайдером в ```App.jsx```
Все компоненты внутри ```TodoProvider``` теперь имеют доступ к общему состоянию:
```jsx
import { TodoProvider } from './context/TodoContext';
import TodoForm from './components/TodoForm';
import TodoList from './components/TodoList';

function App() {
  return (
    <TodoProvider>
      <div style={{ maxWidth: '400px', margin: '40px auto', fontFamily: 'sans-serif' }}>
        <h1>Управление состоянием: Context + Reducer</h1>
        <TodoForm />
        <TodoList />
      </div>
    </TodoProvider>
  );
}

export default App;
```
------------------------------
## 3. Чек-лист для самопроверки (3 минуты)

* Понимаете, какую проблему решает Context API и что такое Prop Drilling.
* Знаете, почему функция редюсера (reducer) должна быть чистой и не должна мутировать стейт напрямую.
* Помните, что передается в объекте action (какие стандартные свойства он имеет).
* Понимаете, почему Context API не всегда заменяет полноценные стейт-менеджеры (Zustand/Redux) на огромных проектах (контекст при изменении данных ререндерит всех своих потребителей).

------------------------------
## 4. Домашнее задание (2 минуты)

   1. Добавить глобальную очистку: В файл ```TodoProvider.jsx``` в редюсер добавить новый кейс ```case 'CLEAR_ALL': return [];```.
   2. Создать компонент панели управления: Создать кнопку «Очистить все задачи».
   3. Подключить кнопку: Вызвать при клике на эту кнопку ```dispatch({ type: 'CLEAR_ALL' })``` и проверить, что весь список мгновенно удаляется, на каком бы уровне интерфейса эта кнопка ни находилась.
