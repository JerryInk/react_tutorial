## Лекция 5: Работа с формами и ```useRef```

* Длительность: 45 минут
* Сложность: Средняя
* Цель: Научиться обрабатывать пользовательский ввод, понять разницу между управляемыми и неуправляемыми компонентами, а также освоить хук ```useRef``` для работы с DOM и сохранения данных без рендеринга.

------------------------------
## 1. Теоретический блок (25 минут)
## Управляемые компоненты (Controlled Components)
В React стандартным подходом является синхронизация состояния полей ввода с состоянием компонента. Такое поле называется управляемым.

* Источником правды является React State (```useState```).
* Значение инпута привязывается к переменной через атрибут ```value```.
* Любое изменение текста отслеживается через событие ```onChange```, которое обновляет стейт.
* Это аналог двустороннего связывания ```[(ngModel)]``` из Angular.

## Неуправляемые компоненты (Uncontrolled Components)
В этом случае данные формы хранятся внутри самого DOM-элемента (как в классическом HTML). React не следит за каждым нажатием клавиши. Значение считывается один раз — например, в момент отправки формы. Для этого используется хук ```useRef```.

## Хук ```useRef```: Две главные задачи
Хук ```useRef``` возвращает изменяемый объект, свойство ```.current``` которого инициализируется переданным аргументом. Этот объект сохраняет свое значение между рендерами.
```javascript
const myRef = useRef(initialValue);
```
У ```useRef``` есть два принципиально разных сценария использования:

   1. Доступ к DOM-элементам напрямую:
   Вы связываете реф с HTML-тегом через атрибут ```ref={myRef}```. После этого в ```myRef.current``` будет лежать ссылка на реальный DOM-узел (аналог ```@ViewChild``` или ```viewChild()``` в Angular). Это нужно для фокуса на инпутах, запуска анимаций или измерения размеров элементов.
   2. Хранение мутирующих данных без перезапуска рендеринга:
   В отличие от ```useState```, изменение значения в ```myRef.current = newValue``` не вызывает ререндер компонента. Это идеальное место для хранения ID таймеров, счетчиков или любых технических флагов, которые не влияют на визуальное отображение.

------------------------------
## 2. Практический блок (15 минут)## Шаг 1. Создаем управляемую форму
Создадим файл src/components/ControlledForm.jsx. Здесь каждое движение пользователя контролируется стейтом:

import { useState } from 'react';

function ControlledForm() {
  const [formData, setFormData] = useState({ username: '', email: '' });

  const handleChange = (e) => {
    const { name, value } = e.target;
    // Сохраняем иммутабельность, обновляя конкретное поле по ключу [name]
    setFormData(prev => ({ ...prev, [name]: value }));
  };

  const handleSubmit = (e) => {
    e.preventDefault(); // Отменяем перезагрузку страницы
    console.log("Данные управляемой формы отправлены:", formData);
  };

  return (
    <form onSubmit={handleSubmit} style={{ border: '1px solid #ddd', padding: '15px', marginBottom: '15px' }}>
      <h3>Управляемая форма</h3>
      <div>
        <label>Имя: </label>
        <input type="text" name="username" value={formData.username} onChange={handleChange} />
      </div>
      <div style={{ marginTop: '10px' }}>
        <label>Email: </label>
        <input type="email" name="email" value={formData.email} onChange={handleChange} />
      </div>
      <button type="submit" style={{ marginTop: '10px' }}>Отправить</button>
    </form>
  );
}

export default ControlledForm;

## Шаг 2. Работа с useRef: Фокус и неуправляемый ввод
Создадим файл src/components/UncontrolledForm.jsx. Здесь мы автоматически сфокусируем инпут при старте и считаем данные без стейта:

import { useEffect, useRef } from 'react';

function UncontrolledForm() {
  // Ссылки на DOM-элементы
  const inputRef = useRef(null);
  const emailRef = useRef(null);

  useEffect(() => {
    // Задача 1: Автоматически ставим фокус в поле "Имя" при монтировании
    inputRef.current.focus();
  }, []);

  const handleSubmit = (e) => {
    e.preventDefault();
    // Задача 2: Читаем значения напрямую из DOM-узлов в момент сабмита
    const username = inputRef.current.value;
    const email = emailRef.current.value;
    
    console.log("Данные неуправляемой формы:", { username, email });
  };

  return (
    <form onSubmit={handleSubmit} style={{ border: '1px solid #ddd', padding: '15px' }}>
      <h3>Неуправляемая форма + useRef</h3>
      <div>
        <label>Имя: </label>
        <input type="text" ref={inputRef} placeholder="Сюда встанет фокус" />
      </div>
      <div style={{ marginTop: '10px' }}>
        <label>Email: </label>
        <input type="email" ref={emailRef} />
      </div>
      <button type="submit" style={{ marginTop: '10px' }}>Отправить</button>
    </form>
  );
}

export default UncontrolledForm;

## Шаг 3. Подключение в App.jsx
Обновим src/App.jsx, чтобы вывести обе формы:

import ControlledForm from './components/ControlledForm';
import UncontrolledForm from './components/UncontrolledForm';

function App() {
  return (
    <div style={{ maxWidth: '450px', margin: '40px auto', fontFamily: 'sans-serif' }}>
      <h1>Работа с формами и useRef</h1>
      <ControlledForm />
      <UncontrolledForm />
    </div>
  );
}

export default App;

------------------------------
## 3. Чек-лист для самопроверки (3 минуты)

* Понимаете разницу между управляемым инпутом и неуправляемым.
* Знаете, какое свойство объекта рефа содержит ссылку на DOM-элемент или значение.
* Можете ответить, вызовет ли код myRef.current = 10 повторный рендер компонента.
* Помните, как отменить стандартное поведение перезагрузки страницы при отправке формы.

------------------------------
## 4. Домашнее задание (2 минуты)

   1. Создать счетчик кликов на useRef: Добавить в любой компонент кнопку и скрытый технический счетчик через useRef (например, const clickCount = useRef(0)).
   2. Внедрить логику:
   * При клике на кнопку увеличивать clickCount.current на 1 и выводить это значение в обычный console.log.
      * Обратить внимание, что цифра на экране не меняется при кликах, так как нет рендера.
   3. Добавить триггер рендера: Добавить вторую кнопку, которая меняет обычный стейт (useState), и убедиться, что после её нажатия компонент перерисовывается и показывает актуальное накопленное число из useRef.

------------------------------
Переходим к визуальному оформлению приложений в Лекции 6: Стилизация в React (CSS-модули, Tailwind CSS), или у вас возникли вопросы по формам и рефам? Как продолжаем?

