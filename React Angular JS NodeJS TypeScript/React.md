### 1. **Установите Node.js и npm**

React требует установленного Node.js и npm. Скачайте и установите [Node.js](https://nodejs.org/). Проверьте установку:

```cs
node -v 
npm -v
```



### 2. **Создайте проект с помощью Create React App**

Create React App — это инструмент для быстрого создания проектов React. Выполните команду:

```cs
npx create-react-app my-react-app
```

- **`npx`**: инструмент, поставляемый с npm 5.2+.
- **`my-react-app`**: имя вашего проекта. Вы можете использовать любое другое имя.

Перейдите в созданную папку:

```bash
cd my-react-app
```



### 3. **Запустите проект**

Запустите сервер разработки:

```cs
npm start
```

Откройте http://localhost:3000 в браузере. Вы увидите стартовую страницу React.



### 4. **Структура проекта**

Основные файлы и папки:

- **`public/`**: статические файлы, такие как `index.html`.
- **`src/`**: основной код приложения.
    - **`index.js`**: точка входа в приложение.
    - **`App.js`**: главный компонент приложения.
    - **`App.css`**: стили для компонента `App`.



### 5. **Создание компонента**

Добавьте новый компонент. Например, создадим `HelloComponent`.

1. Создайте файл `src/HelloComponent.js`:

```js
import React from 'react';  
const HelloComponent = () => {   
		return <h1>Hello, React!</h1>; 
	};  
	
export default HelloComponent;
```

2. Импортируйте и используйте его в `App.js`:

```js
import React from 'react'; 
import HelloComponent from './HelloComponent';  
function App() {   
	return (     
		<div className="App">       
			<HelloComponent />     
		</div>   
		); 
	}  
	
export default App;
```

### 6. **Работа с состоянием и событиями**

Пример компонента с состоянием:


```js
import React, { useState } from 'react';  const Counter = () => {   const [count, setCount] = useState(0);    const increment = () => setCount(count + 1);    return (     <div>       <p>Счётчик: {count}</p>       <button onClick={increment}>Увеличить</button>     </div>   ); };  export default Counter;
```

Используйте его в `App.js`:

`import Counter from './Counter';  function App() {   return (     <div className="App">       <Counter />     </div>   ); }  export default App;`

---

### 7. **Добавление стилей**

React поддерживает стили несколькими способами:

- **CSS-файлы**: просто импортируйте файл. Например, `App.css`.
- **CSS-модули**: создайте файл с именем `ComponentName.module.css`.
- **Inline-стили**: используйте объект стилей:

`const style = {   color: 'blue',   fontSize: '20px', };  return <h1 style={style}>Hello, Styled React!</h1>;`

---

### 8. **Работа с маршрутизацией**

Установите библиотеку `react-router-dom` для маршрутизации:


`npm install react-router-dom`

Пример маршрутизации:

`import React from 'react'; import { BrowserRouter as Router, Route, Routes, Link } from 'react-router-dom';  const Home = () => <h1>Главная</h1>; const About = () => <h1>О нас</h1>;  function App() {   return (     <Router>       <nav>         <Link to="/">Главная</Link> | <Link to="/about">О нас</Link>       </nav>       <Routes>         <Route path="/" element={<Home />} />         <Route path="/about" element={<About />} />       </Routes>     </Router>   ); }  export default App;`

---

### 9. **Установка дополнительных библиотек**

Часто используемые библиотеки:

- **Axios** (для работы с API):
    
    
    `npm install axios`
    
- **Styled Components** (для стилизации компонентов):
    
    
    `npm install styled-components`
    
- **Redux** (для управления состоянием):
    
    
    `npm install @reduxjs/toolkit react-redux`
    

---

### 10. **Сборка для продакшена**

Для создания оптимизированной сборки:

`npm run build`

Готовый код будет находиться в папке `build/`.