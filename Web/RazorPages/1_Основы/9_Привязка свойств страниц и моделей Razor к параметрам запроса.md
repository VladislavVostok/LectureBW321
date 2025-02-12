Для получения отправленных данных мы можем использовать параметры в методах OnGet/OnPost/OnPut/OnDelete и затем передавать их значения свойствам или как-то иначе обрабатывать. Однако Razor Pages позволяет напрямую установить привязку свойств страницы Razor Pages и ее модели и параметров запроса с помощью атрибута BindProperty, что в ряде случаев может упростить обработку.

Например, у нас есть страница Razor Index.cshtml и код связанной модели IndexModel в файле Index.cshtml.cs:


Определим в файле Index.cshtml.cs следующий код:

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
 
    [IgnoreAntiforgeryToken]
    public class IndexModel : PageModel
    {
        [BindProperty]
        public string Name { get; set; } = "";
 
        [BindProperty]
        public int Age { get; set; }
 
    }
}
```

В модели IndexModel определены два свойства, к которым применяется атрибут BindProperty. Причем в модели нет никаких методов OnGet или OnPost. Тем не менее если в запросе будут данные с ключами name и age (регистр названий ключей не имеет значения), то эти данные будут автоматически передаваться одноименным свойствам.

Далее на странице Index.cshtml определим форму для ввода данных и элементы для их вывода:

```html
@page
 
@model RazorPagesApp.Pages.IndexModel
 
<h2>Введите данные</h2>
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
@if(Request.Method == "POST")
{
    <h3>Полученные данные</h3>
    <p>Name: @Model.Name</p>
    <p>Age: @Model.Age</p>
}
```

В данном случае названия полей формы соответствуют именам свойств модели IndexModel, за счет чего с помощью атрибута BindProperty будет происходить автоматическая привязка


При этом можно также обрабатываться запросы в методах OnGet/OnPost, например:

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
 
    [IgnoreAntiforgeryToken]
    public class IndexModel : PageModel
    {
        [BindProperty]
        public string Name { get; set; } = "";
 
        [BindProperty]
        public int Age { get; set; }
 
        public string Message { get; private set; } = "";
        public void OnGet()
        {
            Message = "Введите данные";
        }
        public void OnPost()
        {
            Message = $"Имя: {Name}  Возраст: {Age}";
        }
 
    }
}
```

### Привязка к сложным объектам

Выше каждое из отправляемых из формы значений связывалось с определенным свойством. Однако можно также определить общую модель, которая будет объединять все эти значения. Например, изменим модель IndexModel следующим образом:

```cs
sing Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
 
    [IgnoreAntiforgeryToken]
    public class IndexModel : PageModel
    {
        [BindProperty]
        public Person? Person { get; set; }
    }
    public record class Person (string Name, int Age);
}
```

Теперь данные будут связываться с объектом класса Person.

На странице Index.cshtml изменим код вывода полученных значений:

```html
@page
 
@model RazorPagesApp.Pages.IndexModel
 
<h2>Введите данные</h2>
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
@if(Request.Method == "POST")
{
    <h3>Полученные данные</h3>
    <p>Name: @Model.Person?.Name</p>
    <p>Age: @Model.Person?.Age</p>
}
```

### Получение данных из GET-запросов

По умолчанию атрибут BindProperty не поддерживает привязку свойств к значениям из GET-запрос. Для добавления этого типа запросов у атрибута для свойства SupportGet необходимо установить значение true:

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
 
    [IgnoreAntiforgeryToken]
    public class IndexModel : PageModel
    {
        [BindProperty(SupportsGet = true)]
        public string? Name { get; set; }
    }
}
```

На странице Index.cshtml выведем полученное значение:

```html
@page
 
@model RazorPagesApp.Pages.IndexModel
 
<h2>Name: @Model.Name</h2>
```

### Переопределение названий параметров

Выше свойства модели и параметры запросы связывались по имени. Однако мы можем переопределить имя параметра запроса с помощью свойства Name атрибута BindProperty:

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        [BindProperty(SupportsGet = true, Name = "id")]
        public string? Name { get; set; }
    }
}
```

В данном случае свойству Name будет передаваться значение параметра "id":