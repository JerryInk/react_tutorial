## Лекция 12: Профессиональная работа с формами через React Hook Form

* Длительность: 50 минут
* Сложность: Средняя
* Цель: Научиться создавать производительные формы, настраивать валидацию без лишних ререндеров и обрабатывать ошибки ввода.

------------------------------
## 1. Теоретический блок (25 минут)
Для начала установим библиотеку:
```bash
npm install react-hook-form
```
## Философия React Hook Form
В отличие от Angular, где реактивные формы (Reactive Forms) требуют создания больших объектов в TypeScript, RHF использует подход на основе рефов (```useRef```). Компонент формы не перерисовывается, пока пользователь пишет текст. Перерисовка происходит только в момент вывода ошибки или финальной отправки.

## Главный инструмент: Хук ```useForm```
Этот хук возвращает объект с набором методов для управления формой. Основные из них:

   1. ```register```: Функция, которая связывает ваш HTML-инпут с системой React Hook Form. Она автоматически вешает на инпут нужные рефы и слушатели событий. Мы вызываем её через spread-оператор: ```{...register('fieldName')}```.
   2. ```handleSubmit```: Функция-обертка для отправки формы. Она сама перехватывает событие браузера ```e.preventDefault()```, проверяет валидацию, и если всё в порядке, передает чистый объект с данными в ваш финальный обработчик.
   3. ```formState: { errors }```: Объект, в котором в реальном времени хранятся все ошибки валидации для каждого поля.

## Правила валидации
Параметры валидации передаются вторым аргументом в функцию ```register```. Они полностью повторяют стандартные атрибуты HTML5:

* ```required: 'Поле обязательно для заполнения'```
* ```minLength: { value: 3, message: 'Минимум 3 символа' }```
* ```pattern: { value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+.[A-Z]{2,}$/i, message: 'Неверный формат email' }```

------------------------------
## 2. Практический блок (20 минут)
Создадим профессиональную форму регистрации пользователя с валидацией имени, email и пароля.

## Шаг 1. Создаем компонент формы ```src/components/RegisterForm.jsx```
Создайте файл и добавьте следующий код:
```jsx
import { useForm } from 'react-hook-form';

function RegisterForm() {
  // 1. Инициализируем хук и настраиваем дефолтные значения
  const {
    register,
    handleSubmit,
    formState: { errors },
    reset
  } = useForm({
    defaultValues: {
      username: '',
      email: '',
      role: 'user'
    }
  });

  // 2. Финальный обработчик, который сработает ТОЛЬКО если форма валидна
  const onSubmit = (data) => {
    console.log("Данные формы успешно валидированы и готовы к отправке:", data);
    reset(); // Очищаем форму после успешной отправки
  };

  console.log("=== Рендер компонента формы ==="); 
  // Обратите внимание в консоли: компонент рендерится при старте, 
  // но НЕ рендерится, пока вы печатаете буквы! Только в момент появления/исчезновения ошибок.

  return (
    <form onSubmit={handleSubmit(onSubmit)} style={{ display: 'flex', flexDirection: 'column', gap: '15px' }}>
      
      {/* Поле: Имя пользователя */}
      <div>
        <label style={{ display: 'block', marginBottom: '5px' }}>Имя пользователя:</label>
        <input
          type="text"
          {...register('username', { 
            required: 'Имя обязательно для заполнения',
            minLength: { value: 3, message: 'Имя должно быть не короче 3 символов' }
          })}
          style={{ width: '100%', padding: '8px', borderColor: errors.username ? 'red' : '#ccc' }}
        />
        {/* Вывод ошибки по условию */}
        {errors.username && <span style={{ color: 'red', fontSize: '12px' }}>{errors.username.message}</span>}
      </div>

      {/* Поле: Email */}
      <div>
        <label style={{ display: 'block', marginBottom: '5px' }}>Email:</label>
        <input
          type="text"
          {...register('email', {
            required: 'Email обязателен',
            pattern: {
              value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
              message: 'Введен некорректный email адрес'
            }
          })}
          style={{ width: '100%', padding: '8px', borderColor: errors.email ? 'red' : '#ccc' }}
        />
        {errors.email && <span style={{ color: 'red', fontSize: '12px' }}>{errors.email.message}</span>}
      </div>

      {/* Поле: Роль (Селект) */}
      <div>
        <label style={{ display: 'block', marginBottom: '5px' }}>Роль в системе:</label>
        <select {...register('role')} style={{ width: '100%', padding: '8px' }}>
          <option value="user">Пользователь</option>
          <option value="moderator">Модератор</option>
          <option value="admin">Администратор</option>
        </select>
      </div>

      <button type="submit" style={{ padding: '10px', background: '#007bff', color: '#fff', border: 'none', borderRadius: '4px', cursor: 'pointer' }}>
        Зарегистрироваться
      </button>
    </form>
  );
}

export default RegisterForm;
```
## Шаг 2. Подключаем форму в ```src/App.jsx```
```jsx
import RegisterForm from './components/RegisterForm';

function App() {
  return (
    <div style={{ maxWidth: '400px', margin: '5px auto', padding: '40px 20px', fontFamily: 'sans-serif' }}>
      <h2 style={{ textAlign: 'center' }}>Регистрация нового аккаунта</h2>
      <div style={{ border: '1px solid #ddd', padding: '20px', borderRadius: '8px', background: '#f9f9f9' }}>
        <RegisterForm />
      </div>
    </div>
  );
}

export default App;
```
------------------------------
## 3. Чек-лист для самопроверки (3 минуты)

* Понимаете, почему React Hook Form работает быстрее, чем связка из нескольких ```useState``` на каждый инпут.
* Знаете, какую роль выполняет функция ```register``` и почему мы пишем перед ней три точки ```{...register()}```.
* Помните, нужно ли вручную писать ```e.preventDefault()``` внутри финальной функции ```onSubmit```.
* Умеете доставать текст ошибки из объекта ```formState.errors```.

------------------------------
## 4. Домашнее задание (2 минуты)

   1. Добавить поле «Пароль»: Добавьте в форму инпут типа ```password```.
   2. Настроить сложную валидацию:
      * Сделать поле обязательным.
      * Установить минимальную длину в 6 символов.
      * Добавить кастомную валидацию (свойство ```validate```), которая проверяет, содержит ли пароль хотя бы одну цифру.
      * Подсказка: ```validate: (value) => /\d/.test(value) || 'Пароль должен содержать хотя бы одну цифру'```.
   3. Проверить работу: Убедиться, что форма не отправляется и выводит кастомное сообщение, если ввести только буквы.
