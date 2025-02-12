Вместе со страницей Razor в проект добавляется файл связанного кода на языке C#. Этот код определяется в виде класса, который по умолчанию называется по имени страницы плюс суффикс "Model" (например, для страницы Index - класс IndexModel) и который помещается в файл с именем [файл_страницы_razor].cs.
![[2.11.png]]

Например, при добавлении страницы Index.cshtml вместе с ней будет добавляться файл Index.cshtml.cs, который будет содержать определение класса IndexModel

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        public void OnGet()
        {
        }
    }
}
```

Класс модели страницы Razor должен обязательно наследоваться от абстрактного класса PageModel. По умолчанию данный класс содержит пустой метод OnGet(), который призван обрабатывать get-запросы. Здесь мы можем определить какие-то данные, какую-то логику, которая будет применяться на странице razor.

К пример, изменим этот класс следующим образом:


```cs
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        public string Message { get; }
        public IndexModel()
        {
            Message = "Hello World";
        }
        public string PrintTime() => DateTime.Now.ToShortTimeString();
    }
}
```

В данном случае класс IndexModel определяет свойство Message и метод PrintTime, который возвращает текущее время.

Для подключения класса-модели на страницу Razor применяется директива @model, после которой идет имя класса. Например, изменим определение страницы Index.cshtml следующим образом:

```html
@page
 
@model RazorPagesApp.Pages.IndexModel
<h2>@Model.Message</h2>
 
<h3>Time: @Model.PrintTime()</h3>
```

Обратите внимание, что после директивы `@model` указывается полное имя класса с учетом пространства имен.

После подключения модели мы можем обращаться к ее функционалу через свойство Model. Например, с помощью выражения `@Model.Message` мы можем обратиться к свойству Message в модели.

В итоге при обращении к странице Index и генерации ответа будет использоваться функционал модели IndexModel: