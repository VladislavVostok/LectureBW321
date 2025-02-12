В get-запросе также можно передать на страницу данные. Распространенным способом передачи данных в GET-запросе представляет строка запроса. Строка запроса представляет ту часть адреса, которая идет после знака вопроса ? и представляет набор параметров, где каждый параметр отделен от другого с помощью амперсанда:

```
название_ресурса?параметр1=значение1&параметр2=значение2
```

Например, в адресе:
```
https://localhost:7085/Index?name=Tom&age=37
```

часть ?name=Tom&age=37 как раз представляет строку запроса, которая содержит два параметра: name и age. Значение параметра name - "Tom", а значение параметра age - 37.


Например, у нас есть страница Razor Index.cshtml и код связанной модели IndexModel в файле Index.cshtml.cs.

Передадим в страницу Index.cshtml через строку запроса данные для параметра name. Для этого определим следующую модель IndexModel:

```cs
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        public string Message { get; private set; } = "";
        public void OnGet(string name)
        {
            Message = $"Name: {name}";
        }
    }
}
```

А на странице Index.cshtml выведем значение свойства Message модели:

```html
@page
 
@model RazorPagesApp.Pages.IndexModel
<h2>@Model.Message</h2>
```


То есть в данном случае при обращении к методу Index с запросом `https://localhost:7085/Index?name=Vlad` параметру name будет передаваться значение "Vlad".

Система привязки по умолчанию сопоставляет параметры запроса и параметры метода по имени. То есть, если в строке запроса идет параметр `name`, то его значение будет передаваться именно параметру метода, который также называется `name`. При этом должно быть также соответствие по типу, то есть если параметр метода принимает числовое значение, то и через строку запроса надо передавать для этого параметра число, а не строку.

Подобным образом можно передать значения для нескольких параметров. Например, изменим модель IndexModel следующим образом:

```cs
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        public string Message { get; private set; } = "";
        public void OnGet(string name, int age)
        {
            Message = $"Name: {name}  Age: {age}";
        }
    }
}
```


В этом случае мы можем обратиться к действию, набрав в адресной строке https://localhost:7085?name=Tom&age=37.

Если же мы не используем параметры в строке запроса, то мы можем использовать параметры по умолчанию, которые будут работать, если через строку запроса не передается никаких параметров:

```cs
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        public string Message { get; private set; } = "";
        public void OnGet(string name = "Bob", int age = 33)
        {
            Message = $"Name: {name}  Age: {age}";
        }
    }
}
```

Например, при отправке запроса https://localhost:7085/Index?age=41 параметр name будет иметь значение по умолчанию.


Подобным образом можно получать данные из строки запроса непосредственно на странице Index.cshtml

```html
@page
 
<h2>@Message</h2>
 
@functions {
    public string Message { get; set; } = "";
    public void OnGet(string name, int age)
    {
        Message = $"Name: {name}  Age: {age}";
    }
}
```

### Передача сложных объектов

Хотя строка запроса преимущественно используется для передачи данных примитивных типов, но мы также можем принимать более сложные объекты. Например, определим рядом с моделью класс Person:

```cs
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        public string Message { get; private set; } = "";
        public void OnGet(Person person)
        {
            Message = $"Person  {person.Name} ({person.Age})";
        }
    }
    public record class Person(string Name, int Age);
}
```

Класс Person определяет два свойства: Name и Age. И в модели IndexModel метод OnGet принимает параметр типа Person. В этом случае значения через строку запроса передаются как и в предыдущем случае. При этом параметры строки запроса должны соответствовать по имени свойствам объекта. Регистр названий параметров при этом не учитывается.


### Передача массивов

Допустим, метод OnGet принимает массив строк-имен:

```cs
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        public string Message { get; private set; } = "";
        public void OnGet(string[] people)
        {
            string result = "";
            foreach (var person in people)
                result = $"{result}{person}; ";
            Message = result;
        }
    }
}
```

Для передачи методу данных через строку запроса применяется название параметра:

```
https://localhost:7085/?people=Tom&people=Bob&people=Sam
```

Можно явным образом указать индексы элементов:

```
https://localhost:7085/?people[0]=Tom&people[2]=Bob&people[1]=Sam
```

Можно просто ограничиться индексами:

```
https://localhost:7085/?[0]=Tom&[2]=Bob&[1]=Sam
```

### Передача массивов сложных объектов

Подобным образом можно принимать массивы сложных объектов. Например, определим следующую модель IndexModel, которая в методе Onget принимает массив объектов некоторого класса Person:

```cs
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        public string Message { get; private set; } = "";
        public void OnGet(Person[] people)
        {
            string result = "";
            foreach (Person person in people)
            {
                result = $"{result} {person.Name}; ";
            }
            Message = result;
        }
    }
    public record class Person(string Name, int Age);
}
```

И чтобы передать в этот метод данные, нам надо использовать запрос типа

```
https://localhost:7085/?people[0].name=Tom&people[0].age=37&people[1].name=Bob&people[1].age=41
```

В этом случае в массиве people будут два объекта Person.

Также можно опустить название параметра и оставить только индексы:

```
https://localhost:7085/?[0].name=Tom&[0].age=37&[1].name=Bob&[1].age=41
```


### Передача словарей Dictionary

Передача в метод словарей аналогично передаче массивов, только у передаваемых элементов необходимо установить ключ. Например, пусть метод OnGet получает в качестве параметра объект Dictionary:


```cs
public void OnGet(Dictionary<string,string> items)
{
    string result = "";
    foreach (var item in items)
    {
        result = $"{result} {item.Key} - {item.Value}; ";
    }
    Message = result;
}
```

Для отправки данных мы можем использовать строку запроса:

```
https://localhost:7085/?items[germany]=berlin&items[france]=paris&items[spain]=madrid
```

Каждый отдельный элемент этой строки типа `items[germany]=berlin` будет представлять элемент словаря, где `items` - название словаря, "germany" - ключ, а "berlin" - значение.

### Объект Request.Query

Параметры представляют самый простой способ получения данных, но в действительности нам необязательно их использовать. На странице и в ее модели доступен объект Request, у которого можно получить как данные строки запроса через свойство Request.Query. Это свойство представляет объект `IQueryCollection`, где по ключу - названию параметра можно получить его значение. Например:

```cs
public void OnGet()
{
    string name = Request.Query["name"];
    string age = Request.Query["age"];
    Message = $"Name: {name}  Age: {age}";
}
```

В данном случае мы можем передать странице данные через запрос типа https://localhost:7085/Index?name=Tom&age=37.


## POST-запросы и отправка форм

Кроме GET-запросов для отправки данных приложению также широко применяются POST-запросы. Как правило, такие запросы отправляются с помощью форм на html-странице. Но основные принципы передачи данных будут теми же, что и в GET-запросах. Рассмотрим различные сценарии при передаче данных в POST-запросе странице Razor.

Например, у нас есть страница Razor Index.cshtml и код связанной модели IndexModel в файле Index.cshtml.cs/


Например, пусть страница Index.cshtml будет отправлять форму и получать из формы некоторое значение. Для этого определим следующую модель IndexModel:

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
 
    [IgnoreAntiforgeryToken]
    public class IndexModel : PageModel
    {
        public string Message { get; private set; } = "";
        public void OnGet()
        {
            Message = "Введите свое имя";
        }
        public void OnPost(string username)
        {
            Message = $"Ваше имя: {username}";
        }
    }
}
```

Здесь определено два метода. В методе OnGet обрабатываются get-запросы, что сводится к установки свойства Message.

В методе OnPost() обрабатываем POST-запросы. Данный метод будет получать извне из отправленной формы некоторую строку через параметр username и с его помощью переустанавливает значение свойства Message.

Самый важный момент при обработки отправляемых форм - класс модели Razor для валидации полученных форм использует специальный токен - AntiforgeryToken. Далее мы рассмотрим, как его устанавливать. Однако мы можем отключить валидацию формы с помощью этого токена с помощью атрибута [IgnoreAntiforgeryToken], который применяется к классу модели (также можно применять глобально в приложении).

Для отправки формы на странице Index.cshtml определим следующий код:


```html
@page
 
@model RazorPagesApp.Pages.IndexModel
 
<h2>@Model.Message</h2>
<form method="post" >
    <input type="text" name="username" />
    <input type="submit" value="Отправить" />
</form>
```

Здесь определена форма для ввода условного имени. При получении запроса get приложение просто будет отдавать данную страницу с формой ввода. При получении запроса post класс IndexModel получит отправленное значение и изменит значение свойства Message, которое потом выводится на страницу. При этом, чтобы система могла связать параметры запроса со значениями формы поля форм (их атрибут name) должны называться также, как и параметры методов OnPost или OnGet. То есть параметр в методе OnPost называется username, и поле формы (значение его атрибута name) тоже называется username.

При запуске страница отобразит приглашение к вводу.

А при отправке введенного значения отобразится результат.

Для обработки запроса можно использовать класс, производный от PageModel. Однако страница Razor уже сама по себе представляет модель. И мы можем всю логику обработки и свойства модели определить напрямую в странице. Например, изменим страницу Index.cshtml:

```html
@page
 
<h2>@Model.Message</h2>
<form method="post" >
    <input type="text" name="username" />
    <input type="submit" value="Отправить" />
</form>
 
@functions {
     
    public string Message { get; private set; } = "";
    public void OnGet() => Message = "Введите свое имя";
         
    public void OnPost(string username) => Message = $"Ваше имя: {username}";
}
```


Но в этом случае мы можем опять же столкнуться с необходимостью верификации токена AntiForgery-токена. Чтобы решить проблему с этим токеном, мы можем отключить его глобально. Для этого изменим файл Program.cs:

```cs
using Microsoft.AspNetCore.Mvc;
 
var builder = WebApplication.CreateBuilder(args);
 
// добавляем в приложение сервисы Razor Pages
builder.Services.AddRazorPages(options =>
{
    // отключаем глобально Antiforgery-токен
    options.Conventions.ConfigureFilter(new IgnoreAntiforgeryTokenAttribute());
});
 
var app = builder.Build();
 
// добавляем поддержку маршрутизации для Razor Pages
app.MapRazorPages();  
 
app.Run();
```

### Получение сложных объектов

Подобным образом мы можем получать и большее количество данных. Например, определим на странице Index.cshtml следующую форму:

```html
@page
 
@model RazorPagesApp.Pages.IndexModel
 
<h2>@Model.Message</h2>
<form method="post" >
    <p>
        <label>Имя:</label><br />
        <input type="text" name="name" />
    </p>
    <p>
        <label>Возраст:</label><br />
        <input type="number" name="age" />
    </p>
    <input type="submit" value="Отправить" />
</form>
```

В данном случае пользователь должен ввести имя и возраст.

А в классе модели IndexModel определим следующую логику:

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
 
    [IgnoreAntiforgeryToken]
    public class IndexModel : PageModel
    {
        public string Message { get; private set; } = "";
        public void OnGet()
        {
            Message = "Введите свои данные";
        }
        public void OnPost(string name, int age)
        {
            Message = $"Ваше имя: {name}. Ваш возраст: {age}";
        }
    }
}
```


Хотя мы можем получать весь набор отправляемых данных по отдельности, однако также можно все эти значения объединить в более сложные объекты. Например, определим рядом с моделью класс Person:


```cs
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        public string Message { get; private set; } = "";
        public void OnGet(Person person)
        {
            Message = $"Person  {person.Name} ({person.Age})";
        }
    }
    public record class Person(string Name, int Age);
}
```

Класс Person определяет два свойства: Name и Age. И в модели IndexModel метод OnPost принимает параметр типа Person. При чем поля формы name и age соответствуют по названию свойствам класса Person, поэтому вместо одиночных разрозненных значений мы можем получить отправленную форму в виде объекта Person.

### Получение массивов

Для передачи массивов с помощью формы надо создать набор одноименных полей, которые называются по имени массива. Например, определим следующую модель IndexModel:

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
 
    [IgnoreAntiforgeryToken]
    public class IndexModel : PageModel
    {
        public string[] People { get; private set; } = Array.Empty<string>();
        
        public void OnPost(string[] people)
        {
            People = people;
        }
    }
}
```

В данном случае модель в методе OnPost получает массив строк и передает его свойству People.

Также определим на странице Index.cshtml отправки данных и вывода элементов из People следующий код:

```html
@page
 
@model RazorPagesApp.Pages.IndexModel
 
<h2>Ввeдите имена сотрудников</h2>
<form method="post">
    <p><input name="people" /></p>
    <p><input name="people" /></p>
    <p><input name="people" /></p>
    <input type="submit" value="Отправить" />
</form>
<h3>Список сотрудников</h3>
<ul>
    @foreach(var person in Model.People)
    {
        <li>@person</li>  
    }
</ul>
```

В данном случае на форме, отправляемой пользователю, расположены три поля с именем "people". В итоге при отправке формы будет сформирован массив people из трех элементов, который можно получить в модели методе OnPost:![[2.24.png]]

И также у элементов формы можно было бы явным образом указать индексы:

```html
<form method="post">
    <p><input name="people[0]" /></p>
    <p><input name="people[2]" /></p>
    <p><input name="people[1]" /></p>
    <input type="submit" value="Send" />
</form>
```

Либо можно было бы вовсе ограничиться одними индексами

```html
<form method="post">
    <p><input name="[0]" /></p>
    <p><input name="[2]" /></p>
    <p><input name="[1]" /></p>
    <input type="submit" value="Send" />
</form>
```


### Передача словарей Dictionary

Передача словарей в метод контроллера аналогична передаче элементов массивов за тем исключением, что для каждого элемента устанавливается ключ. Так, определим следующую страницу Index.cshtml:


```html
@page
 
<h2>Ввeдите столицы стран</h2>
<form method="post">
    <p>
        Германия: <input type="text" name="items[germany]" />
    </p>
    <p>
        Франция: <input type="text" name="items[france]" />
    </p>
    <p>
        Испания: <input type="text" name="items[spain]" />
    </p>
    <input type="submit" value="Отправить" />
</form>
<h3>Страны и их столицы</h3>
<ul>
    @foreach(var country in Countries)
    {
        <li>@country.Key - @country.Value</li>  
    }
</ul>
 
@functions{
    Dictionary<string, string> Countries { get; set; } = new();
    public void OnPost(Dictionary<string, string> items)
    {
        Countries = items;
    }
}
```
![[2.25.png]]

### Отправка массивов сложных объектов

При отправке массивов сложных объектов на форме также определяется набор полей, где каждое поле привязано к определенному свойству объекта. Например, изменим страницу Index.cshtml следующим образом:


```html
@page
 
<h2>Ввeдите данные</h2>
<form method="post">
    <p>
        Person1 Name:<br/> 
        <input name="people[0].name" /><br/>
        Person1 Age:<br/>
        <input name="people[0].age" />
    </p>
    <p>
        Person2 Name:<br/> 
        <input name="people[1].name" /><br/>
        Person2 Age:<br/>
        <input name="people[1].age" />
    </p>
    <input type="submit" value="Отправить" />
</form>
<h3>Список сотрудников</h3>
<ul>
    @foreach(var person in People)
    {
        <li>@person.Name (@person.Age)</li>  
    }
</ul>
 
@functions{
    Person[] People { get; set; } = { };
    public void OnPost(Person[] people)
    {
        People = people;
    }
 
    public record class Person(string Name, int Age);
}
```

В данном случае на форму вводятся значения для свойств Name и Age двух объектов Person. После отправке эти объекты уйдут в виде массива people второму методу Index.
![[2.26.png]]


### Получение данных из контекста запроса

Для получения данных отправленных форм через параметры метода OnPost также на страницах Razor и в их моделях можно использовать свойство Request.Form. Это свойство представляет коллекцию `IFormsCollection`, где каждый элемент имеет ключ и значение. В качестве ключа элемента выступает название поля формы, а в качестве значения - введенные в это поле данные. Например, используем Request.Form на странице Index.cshtml:

```html
@page
 
<h2>@Message</h2>
<form method="post" >
    <p>
        <label>Имя:</label><br />
        <input type="text" name="name" />
    </p>
    <p>
        <label>Возраст:</label><br />
        <input type="number" name="age" />
    </p>
    <input type="submit" value="Отправить" />
</form>
 
@functions{
    string Message { get; set; } = "";
    public void OnPost()
    {
        string name = Request.Form["name"];
        string age = Request.Form["age"];
        Message = $"Ваше имя: {name}. Ваш возраст: {age}";
    }
}
```