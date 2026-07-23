## Лекция 7: Оптимизация производительности

* Длительность: 45 минут
* Сложность: Продвинутая
* Цель: Понять причины лишних ререндеров, научиться профилировать компоненты и правильно применять инструменты мемоизации: ```React.memo```, ```useMemo``` и ```useCallback```.

------------------------------
## 1. Теоретический блок (25 минут)
В Angular механизм отслеживания изменений (Change Detection) работает автоматически на основе зоны (zone.js) или сигналов. В React логика обновлений прямолинейна: когда компонент перерисовывается (ререндерится), он по умолчанию перерисовывает всех своих дочерних детей, независимо от того, изменились их пропсы или нет.
Приложения начинают тормозить по двум причинам:

   1. Лишние ререндеры тяжелых UI-деревьев.
   2. Повторное выполнение «тяжелых» вычислений (например, сортировка массива на 10 000 элементов) при каждом рендере.

Для борьбы с этим в React есть три главных инструмента:

## Инструмент 1: ```React.memo```
Это функция высшего порядка (High-Order Component). Она оборачивает компонент и заставляет React сравнивать его новые пропсы со старыми.

* Если пропсы не изменились, React полностью пропускает ререндер этого компонента.
* Важно: Сравнение поверхностное (Shallow Comparison). Если в пропсы передается объект или функция, при каждом рендере родителя их ссылки будут новыми, и ```React.memo``` не сработает.

## Инструмент 2: Хук ```useMemo```
Мемоизирует результат вычислений функции между рендерами.
```javascript
const cachedValue = useMemo(() => calculateSomething(data), [data]);
```
React выполнит функцию ```calculateSomething``` только один раз при старте и перезапустит её исключительно тогда, когда изменится значение в массиве зависимостей ```[data]```.

## Инструмент 3: Хук ```useCallback```
Мемоизирует ссылку на саму функцию. В JavaScript функции — это объекты, поэтому при каждом рендере компонента все объявленные внутри него стрелочные функции создаются заново (получают новую ссылку в памяти).
```javascript
const handleClick = useCallback(() => { console.log(id); }, [id]);
```
Хук сохраняет одну и ту же ссылку на функцию между рендерами, пока не изменятся зависимости. Это критически важно при передаче колбэков в дочерние компоненты, оптимизированные через ```React.memo```.

------------------------------
## 2. Практический блок (15 минут)
## Шаг 1. Создаем тяжелые вычисления и оптимизируем через ```useMemo```
Создадим компонент ```src/components/HeavyCalculator.jsx```, где покажем фильтрацию списка и то, как стейт счетчика не должен перезапускать тяжелый цикл:
```jsx
import { useState, useMemo } from 'react';

function HeavyCalculator() {
  const [count, setCount] = useState(0);
  const [searchTerm, setSearchTerm] = useState('');

  // Имитируем большой массив данных
  const items = useMemo(() => {
    return Array.from({ length: 5000 }, (_, i) => `Элемент списка №${i + 1}`);
  }, []);

  // Оптимизируем фильтрацию. Без useMemo этот цикл крутился бы при каждом клике на счетчик!
  const filteredItems = useMemo(() => {
    console.log('=== Фильтрация массива (тяжелая операция) ===');
    return items.filter(item => item.toLowerCase().includes(searchTerm.toLowerCase()));
  }, [searchTerm, items]); // Перезапустится только если изменился поисковый запрос

  return (
    <div style={{ border: '1px solid #ccc', padding: '15px', marginBottom: '15px' }}>
      <h3>Тяжелые вычисления (useMemo)</h3>
      
      <button onClick={() => setCount(prev => prev + 1)}>
        Кликнуто раз: {count} (ререндер без повторной фильтрации)
      </button>

      <div style={{ marginTop: '10px' }}>
        <input 
          type="text" 
          value={searchTerm} 
          onChange={(e) => setSearchTerm(e.target.value)} 
          placeholder="Поиск по 5000 элементам..." 
        />
      </div>
      <p>Найдено элементов: {filteredItems.length}</p>
    </div>
  );
}

export default HeavyCalculator;
```
## Шаг 2. Оптимизируем дочерний компонент через ```React.memo```
Создадим дочернюю кнопку ```src/components/SmartButton.jsx``` и обернем её в ```React.memo```:
```jsx
import React from 'react';

// Компонент перерисуется только если изменится проп onClick
const SmartButton = React.memo(({ onClick, children }) => {
  console.log(`=> Рендерится кнопка: ${children}`);
  return (
    <button onClick={onClick} style={{ margin: '5px', padding: '8px' }}>
      {children}
    </button>
  );
});

export default SmartButton;
```
## Шаг 3. Собираем всё в ```App.jsx``` и используем ```useCallback```
В файле ```src/App.jsx``` стабилизируем ссылки на функции с помощью ```useCallback```, чтобы ```SmartButton``` не рендерился зря:
```jsx
import { useState, useCallback } from 'react';
import HeavyCalculator from './components/HeavyCalculator';
import SmartButton from './components/SmartButton';

function App() {
  const [appState, setAppState] = useState(false);
  const [points, setPoints] = useState(0);

  // Ссылка на эту функцию будет стабильной между рендерами
  const logMessage = useCallback(() => {
    console.log("Клик по оптимизированной кнопке");
  }, []); // Пустой массив — ссылка не изменится никогда

  const incrementPoints = () => setPoints(prev => prev + 1);

  return (
    <div style={{ maxWidth: '500px', margin: '40px auto', fontFamily: 'sans-serif' }}>
      <h1>Оптимизация производительности</h1>
      
      <HeavyCalculator />

      <div style={{ border: '1px solid #aaa', padding: '15px' }}>
        <h3>Стабилизация ссылок (useCallback)</h3>
        <p>Очки: {points}</p>
        <button onClick={incrementPoints}>Добавить очки (обычная кнопка)</button>
        
        <hr />
        {/* Изменение points заставит App рендериться, но SmartButton пропустит ререндер */}
        <SmartButton onClick={logMessage}>Умная кнопка</SmartButton>
        
        <button onClick={() => setAppState(!appState)} style={{ marginLeft: '10px' }}>
          Переключить стейт App
        </button>
      </div>
    </div>
  );
}

export default App;
```
------------------------------
## 3. Чек-лист для самопроверки (3 минуты)

* Понимаете, почему по умолчанию дочерний компонент рендерится при обновлении родителя.
* Можете объяснить разницу в целях использования ```useMemo``` и ```useCallback```.
* Помните, почему ```React.memo``` не защитит компонент от рендера, если передать в пропсы обычную стрелочную функцию ```onClick={() => console.log(1)}```.
* Знаете, почему не нужно оборачивать абсолютно каждую функцию в приложении в ```useCallback``` (преждевременная оптимизация).

------------------------------
## 4. Домашнее задание (2 минуты)

   1. Создать профиль задачи: Возьмите ваш массив задач из домашнего задания к Лекции 3.
   2. Внедрить ```useMemo```: Добавьте на страницу счетчик, который считает только выполненные задачи. Оберните этот подсчет в ```useMemo```, чтобы он не пересчитывался, когда пользователь просто вводит текст в инпут добавления задачи.
   3. Проверить логи: Расставьте ```console.log``` в вычислениях и убедитесь, что при вводе букв в инпут логи подсчета выполненных задач не дублируются в консоли.
