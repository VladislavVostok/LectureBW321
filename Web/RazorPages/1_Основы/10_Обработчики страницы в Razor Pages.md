Для обработки запроса каждая страница может использовать методы OnGet/OnPost/OnPut и так далее, то есть такие методы, которые предназначены для определенного вида запросов. Однако что если нам необходимо определить на странице для разных ситуаций несколько методов для обработки запроса одного и того же типа запросов (GET или POST)? В этом случае мы можем добавить в код страницы дополнительные обработчики запросов.

Например, у нас есть страница Razor Index.cshtml и код связанной модели IndexModel в файле Index.cshtml.cs:

Например, пусть в проекте определен класс Person, который представляет данные:

```cs
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```

В файле коде Index.cshtml.cs определим следующий код:

```cs
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        // начальные данные - список людей
        List<Person> people = new ()
        {
            new Person ("Tom Smith", 23),
            new Person ("Sam Anderson", 23),
            new Person ("Bob Johnson", 25),
            new Person ("Tom Anderson", 25)
        };
        // отображаемые данные
        public List<Person> DisplayedPeople { get; private set; } = new();
 
        public void OnGet()
        {
            DisplayedPeople = people;
        }
 
        public void OnGetByName(string name)
        {
            DisplayedPeople = people.Where(p => p.Name.Contains(name)).ToList();
        }
        public void OnGetByAge(int age)
        {
            DisplayedPeople = people.Where(p => p.Age == age).ToList();
        }
    }
    public record class Person(string Name, int Age);
}
```

Класс содержит некоторые начальные данные в переменной people - список объектов Person. Свойство DisplayedPeople же будет содержать отображаемые на странице данные, которые могут отличаться от списка people. При стандартном get-запросе в свойство DisplayedPeople передается весь этот список.

И также определены два метода - обработчики для фильтрации списка по имени и возрасту объекта Person. Так как оба этих обработчики будут также обрабатывать запросы get, то их название начинается с префикса OnGet.

На странице Index.cshtml определим вывод списка DisplayedPeople из IndexModel:

```html
@page
 
@using RazorPagesApp.Pages
@model IndexModel
 
<h2>Список пользователей</h2>
<table>
    <tr><th>Name</th><th>Age</th></tr>
    @foreach(Person person in Model.DisplayedPeople)
    {
        <tr>
            <td>@person.Name</td>
            <td>@person.Age</td>
        </tr>
    }
</table>
```

И при обычном запросе к странице сработает метод OnGet, поэтому на страницу будет выведен полный список.

Теперь обратимся к конкретному обработчику. Для этого в строку запроса передается параметр "handler" с указанием имени обработчика. Причем именем обработчика считается та часть названия метода, которая идет после префикса OnGet.


В данном случае обращение идет к обработчику OnGetByName. Поскольку он принимает параметр name, то через строку запроса для этого параметра передается значение: https://localhost:7085/Index?handler=ByName&name=Tom.

### Передача обработчика через параметр маршрута

Не всегда может устроить добавление названия обработчика через строку запроса, наподобие ?handler=ByName. В этом случае можно передать название обработчика через параметр маршрута. Для этого изменим код страницы Person.cshtml:

```html
@page "{handler?}"
 
@using RazorPagesApp.Pages
@model IndexModel
 
<h2>Список пользователей</h2>
<table>
    <tr><th>Name</th><th>Age</th></tr>
    @foreach(Person person in Model.DisplayedPeople)
    {
        <tr>
            <td>@person.Name</td>
            <td>@person.Age</td>
        </tr>
    }
</table>
```

Вопросительный знак после названия параметра "{handler?}" указывает, что данный параметр является необязательным. И в этом случае можно просто указать в url последним сегментом название обработчика: