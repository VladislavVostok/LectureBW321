### 1. **Установите Node.js и npm**

Angular требует установленного Node.js и npm. Скачайте и установите [Node.js](https://nodejs.org/). Проверьте установку:

```bash
node -v 
npm -v
```

### 2. **Установите Angular CLI**

Angular CLI (Command Line Interface) упрощает создание и управление проектами Angular.


```bash
npm install -g @angular/cli
```

Проверьте версию Angular CLI:


```cs
ng version
```

### 3. **Создайте новый проект Angular**

Создайте папку для нового проекта и запустите команду создания проекта:


```cs
ng new my-angular-project
```

Вас спросят:

- Использовать **CSS**, **SCSS**, **SASS** или **LESS** для стилей. Выберите нужный вариант.
- Включить ли Angular Routing (задайте `Yes`, если планируете работать с маршрутизацией).

После этого Angular CLI создаст структуру проекта.

### 4. **Запустите проект**

Перейдите в папку проекта:


```cs
cd my-angular-project
```

Запустите локальный сервер разработки:

```cs
ng serve
```

Откройте в браузере: http://localhost:4200.

---

### 5. **Структура проекта**

Основные папки и файлы:

- **`src/app/`**: основной код приложения.
- **`src/app/app.module.ts`**: главный модуль приложения.
- **`src/app/app.component.ts`**: главный компонент.
- **`src/environments/`**: конфигурации для разных сред (например, `dev` и `prod`).
- **`angular.json`**: настройки проекта.
- **`package.json`**: зависимости и команды npm.

---

### 6. **Добавление компонента**

Для добавления нового компонента используйте Angular CLI:


```cs
ng generate component my-component
```

Или коротко:

```cs
ng g c my-component
```

CLI создаст папку `my-component` с файлами:

- **`my-component.component.ts`**: код компонента.
- **`my-component.component.html`**: шаблон (разметка) компонента.
- **`my-component.component.css`**: стили компонента.
- **`my-component.component.spec.ts`**: тесты.

---

### 7. **Работа с модулями**

Если вы добавили компонент вручную, зарегистрируйте его в файле `app.module.ts`:


```js
import { MyComponent } from './my-component/my-component.component';  

@NgModule({   declarations: [     AppComponent,     MyComponent   ],   imports: [     BrowserModule   ],   providers: [],   bootstrap: [AppComponent] }) 
export class AppModule { }
```


### 8. **Маршрутизация**

Для добавления маршрутов настройте `app-routing.module.ts` или создайте маршруты:


```cs
ng generate module app-routing --flat --module=app
```

Добавьте маршруты:


```js
import { NgModule } from '@angular/core'; 
import { RouterModule, Routes } from '@angular/router'; 
import { MyComponent } from './my-component/my-component.component';  

const routes: Routes = [   
			{ path: 'my-route', component: MyComponent },   
			{ path: '', redirectTo: '/my-route', pathMatch: 'full' } 
		];  
		
		@NgModule({   imports: [RouterModule.forRoot(routes)],ports: [RouterModule] }) 
		export class AppRoutingModule { }
```

---

### 9. **Добавление сервисов**

Создайте сервис:

```cs
ng generate service my-service
```

Сервис будет создан в `src/app/my-service.service.ts`.

Пример использования сервиса:

- **Сервис (`my-service.service.ts`)**:

```js
import { Injectable } from '@angular/core';  

@Injectable({   providedIn: 'root' }) 
export class MyService {   
	getData(): string {
		return 'Hello from MyService!';   
	} 
}
```

- **Компонент (`my-component.component.ts`)**:

```js

import { Component } from '@angular/core'; 
import { MyService } from '../my-service.service';

@Component({   selector: 'app-my-component',   template: '<h1>{{ message }}</h1>',   styles: [] })
export class MyComponent {   
	message: string = '';    
	constructor(private myService: MyService) {     
		this.message = this.myService.getData();   
	} 
}
```

---

### 10. **Запуск сборки для продакшена**

Для сборки оптимизированного приложения:


```bash
ng build --prod
```

Результат появится в папке `dist/`.