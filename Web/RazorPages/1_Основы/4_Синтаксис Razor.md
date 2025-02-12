Ключевым моментом в определении интерфейса на страницах Razor Page является использование конструкций движка Razor. Благодаря Razor мы можем применять на странице выражения языка C#. Синтаксис Razor довольно прост - все его конструкции предваряются символом **@**, после которого происходит переход от кода HTML к коду C#. При генерации ответа клиенту Razor обрабатывает выражения языка C# и на их основе генерирует код HTML. Например, определим следующее представление:

```cs
@page
<!DOCTYPE html>
<html>
<head>
    <title>Shalom Shabbat</title>
    <meta charset="utf-8" />
</head>
<body>
    <h2>Time: @DateTime.Now.ToShortTimeString()</h2>
</body>
</html>
```

Здесь вместо выражения `@DateTime.Now.ToShortTimeString()` при рендеринге представления будет вставляться текущее время:![[2.2.png]]

Стоит отметить, что по умолчанию Razor подключает на страницы следующие пространства имен

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Rendering;
using Microsoft.AspNetCore.Mvc.ViewFeatures;
```

Соответственно мы можем использовать функционал этих пространств имен на страницах Razor, как в примере выше структуру DateTime из пространства System.

### Типы конструкций Razor

Все конструкции Razor можно условно разделить на два вида: однострочные выражения и блоки кода.

Пример применения однострочных выражений:

```html
<p>Date: @DateTime.Now.ToLongDateString()</p>
```

В данном случае используется объект DateTime и его метод `ToLongDateString()`

Или еще один пример:

```html
<p>@(20 + 30)</p>
```

Так как перед скобками стоит знак @, то выражение в скобках будет интерпретироваться как выражение на языке C#. Поэтому браузер выведет число 50, а не "20 + 30".

Но если вдруг мы создаем код html, в котором присутствует символ @ не как часть синтаксиса Razor, а сам по себе, то, чтобы его отобразить, нам надо его дублировать:

```html
<p>@@DateTime.Now =@DateTime.Now.ToLongDateString()</p>
```

Блоки кода могут иметь несколько выражений. Блок кода заключается в фигурные скобки, а каждое выражение завершается точкой с запятой аналогично блокам кода и выражениям на C#:


```html
@page
 
@{
    string head = "Hello БВ321"; // определяем переменную head
    string text = "ASP.NET Core Application";   // определяем переменную text
}
 
<h2>@head</h2> <!-- используем переменную head -->
<div>@text</div> <!-- используем переменную text -->
```

В блоках кода мы можем определить обычные переменные и потом их использовать в представлении.

Если необходимо вывести значение переменной без каких-либо html-элементов, то мы можем использовать специальный снипет `<text>`:


```html
@page
 
@{
    int i = 8;
    <text>@i</text>
}
<text>@(i+1)</text>
```

В Razor могут использоваться комментарии. Они располагаются между символами `@**@`:

```html
@* текст комментария *@
```


### Условные конструкции

Также мы можем использовать условные конструкции:

```html

@page
 
@{
    string morning = "Good Morning";
    string evening = "Good Evening";
    string hello = "Hello";
    int hour = DateTime.Now.Hour;
}
@if (hour < 12)
{
    <h2>@morning</h2>
}
else if (hour > 17)
{
    <h2>@evening</h2>
}
else
{
    <h2>@hello</h2>
}

```

Конструкция `switch`:

```html
@page
 
@{
    string language = "german";
}
@switch(language)
{
    case "russian":
        <h3>Привет мир!</h3>
        break;
    case "german":
        <h3>Hallo Welt!</h3>
        break;
    default:
        <h3>Hello World!</h3>
        break;
}
```

### Циклы

Кроме того, мы можем использовать все возможные циклы. Цикл for:

```html
@page
 
@{
    string[] people = { "Tom", "Sam", "Bob" };
}
<ul>
    @for (var i = 0; i < people.Length; i++)
    {
        <li>@people[i]</li>
    }
</ul>
```


Цикл foreach:

```html
@page
 
@{
    string[] people = { "Tom", "Sam", "Bob" };
}
<ul>
    @foreach (var person in people)
    {
        <li>@person</li>
    }
</ul>
```

Цикл while:

```html
@page
 
@{
    string[] people = { "Tom", "Sam", "Bob" };
    var i = 0;
}
<ul>
    @while ( i < people.Length)
    {
        <li>@people[i++]</li>
    }
</ul>

```

Цикл do..while:

```html
@page
 
@{
    var i = 1;
}
<ul>
    @do
    {
        <li>@(i * i)</li>
    }
    while ( i++ < 5);
</ul>
```

### try...catch

Конструкция `try...catch...finally`, как и в C#, позволяет обработать исключение, которое может возникнуть при выполнение кода:

```html
@page
 
@try
{
    throw new InvalidOperationException("Something wrong");
}
catch (Exception ex)
{
    <p>Exception: @ex.Message</p>
}
finally
{
    <p>finally</p>
}
```

Если в блоке try выбрасывается исключение, то выполняется блок catch. И в любом случае в конце блока try и catch выполняется блок finaly.

### Вывод текста в блоке кода

Обычный текст в блоке кода мы не сможем вывести:

```html
@page
 
@{
    bool isEnabled = true;
}
@if (isEnabled)
{
    Hello World
}
```

В этом случае Razor будет рассматривать строку "Hello" как набор операторов языка C#, которых, естественно в C# нет, поэтому мы получим ошибку. И чтобы вывести текст как есть в блоке кода, нам надо использовать выражение `@:`:

```html
@page
 
@{
    bool isEnabled = true;
}
@if (isEnabled)
{
    @: Hello
}
```


### Функции
Директива @functions позволяет определить функции, которые могут применяться в представлении. Например:

```html
@page
 
@functions
{
    public int Sum(int a, int b) 
    {
        return a + b;
    }
    public int Square(int n) => n * n;
}
<p>Sum of 5 and 4: <b> @Sum(5, 4)</b></p>
<p>Square of 4: <b>@Square(4)</b></p>
```


### Локальные функции

В блоках кода также можно определять локальные функции:

```html
@page
 
@{
    void RenderName(string name)
    {
        <p>Name: <b>@name</b></p>
    }
 
    RenderName("Tom");
    RenderName("Bob");
}
 
<div>@{RenderName("Sam");}</div>
```

В данном случае функция RenderName выводит некоторую разметку html, в которую передается значение параметра name:
![[2.6.png]]

### Инструкция using

С помощью директивы using можно подключать на страницу Razor различные пространства. Например, определим в проекте новый класс Person:

```cs
namespace RazorPagesApp
{
    public class Person
    {
        public string Name { get; }
        public int Age { get; }
        public Person(string name, int age)
        {
            Name = name;
            Age = age;
        }
        public override string ToString() => $"Person {Name} ({Age} лет)";
    }
}
```

В данном случае класс Person расположен в пространстве имен RazorPagesApp:

![[2.7.png]]
Чтобы использовать данный класс на странице Razor, его пространство имен необходимо подключить с помощью директивы @using:

```html
@page
 
@using RazorPagesApp @* подключение пространства имен RazorPagesApp *@
 
@{
    Person tom = new Person("Tom", 37);
}
 
<h2>@tom</h2>
```


В качестве альтрнативы, как и в общем в C#, можно было бы указать полное имя класса с учетом пространства имен:

```cs
RazorPagesApp.Person tom = new Razor
PagesApp.Person("Tom", 37);
```

Но директива using позволяет сократить код.