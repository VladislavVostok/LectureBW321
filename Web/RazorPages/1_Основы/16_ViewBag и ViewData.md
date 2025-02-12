

Для передачи данных из кода c# на страницу Razor можно использовать ряд способов. Самый распространенный и предпочтительный - использование свойств модели страницы. Тем не менее есть и другие способы, в частности, примение объектов ViewBag и ViewData.

### ViewData

ViewData представляет словарь из пар ключ-значение.


В коде модели IndexModel в файле Index.cshtml.cs определим следующий код:

```
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        public void OnGet()
        {
            ViewData["Message"] = "Razor Pages on BW321";
        }
    }
}
```

В данном случае в словаре ViewData определяется элемент с ключом "Message", значением которого является строка "Razor Pages on METANIT.COM". На странице Index.cshtml получим это значение:

```
@page 
 
@model RazorPagesApp.Pages.IndexModel
 
<h2>@ViewData["Message"]</h2>
```

Здесь получаем из словаря ViewData элемент с ключом "Message" и выводим его в заголовке.


Подобным образом через ViewData можно передавать и более комплексные данные, только стоит учитывать, что в этом случае может потрбеоваться приведение типов. Например, определим в коде модели передачу списка строк:

```
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        public void OnGet()
        {
            ViewData["Message"] = "Список пользователей";
            ViewData["People"] = new List<string> { "Tom", "Sam", "Bob" };
        }
    }
}
```

На странице Index.cshtml выведем данные из списка:
```
@page 
 
@model RazorPagesApp.Pages.IndexModel
 
<h2>@ViewData["Message"]</h2>
<ul>
    @if(ViewData["People"] is List<string> people)
    {
        foreach(var person in people)
        {
            <li>@person</li>
        }
    }
</ul>
```


Как видно, здесь потребовалось преобразование данных. С этой точки зрения использование свойств модели вместо ViewData было бы более предпочтительным.

При этом данные ViewData можно определять непосредственно на самой странице Razor:

```
@page 
 
@{
    ViewData["Message"] = "Razor Pages on METANIT.COM";
}
 
<h2>@ViewData["Message"]</h2>
```

В конкретно данном случае применение ViewData не имеет большого смысла, посколько можно было бы просто определить переменную, тем не менее в ряде сценариев может потребоваться передать данные из страницы Razor в другие представления или страницы.

### ViewBag

ViewBag во многом подобен ViewData. Он позволяет динамически определить различные свойства и присвоить им любое значение. Так, мы могли бы переписать предыдущий пример следующим образом::

```
@page 
 
@{
    ViewBag.Message = "Razor Pages on BW321";
}
 
<h2>@ViewBag.Message</h2>
```

Стоит отметить, что свойство ViewBag можно использовать только на странице Razor, а в классе модели оно не доступно.

На странице Razor также можно получать все данные, переданные через ViewData, через ViewBag:

```
@page 
 
@{
    ViewData["Message"] = "Razor Pages on BW321";
}
 
<h2>@ViewBag.Message</h2>
```