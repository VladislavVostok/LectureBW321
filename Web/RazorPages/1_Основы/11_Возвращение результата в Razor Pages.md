Все рассмотренные в прошлых статьях методы OnGet и OnPost, предназначенные для обработки GET и POST-запросов, ничего не возвращали, то есть имели в качестве возвращаемого типа тип void и просто выполняли некоторое действие:

```cs
public void OnGet()
{
    // обработка запроса
}
public void OnPost()
{
    // обработка запроса
}
```

В реальности данные методы по умолчанию возвращали связанную с классом страницу Razor. Однако иногда возникает необходимость возвратить некоторый результат, например, статусный код ошибки или сделать редирект. Для подобных ситуаций данные методы также могут возвращать объекты IActionResult. И для этого в классе PageModel определено ряд методов:

- Content() возвращает объект ContentResult, то есть фактически некоторое текстовое содержимое
    
- File() возвращает с помощью различных перегруженных версий объекты FileContentResult/FileStreamResult/VirtualFileResult, то есть отправляет клиенту файл
    
- Forbid() возвращает статусный код 403
    
- LocalRedirect()/LocalRedirectPermanent() выполняет переадресацию по определенному локальному адресу
    
- NotFound() возвращает статусный код 404
    
- PhysicalFile() возвращает файл по физическому пути
    
- Page() возвращает объект PageResult или фактически текущую страницу Razor
    
- Redirect()/RedirectPermanent() выполняет переадресацию по определенному адресу
    
- RedirectToAction()/RedirectToActionPermanent() выполняет переадресацию на определенное действие контроллера
    
- RedirectToPage()/RedirectToPagePermanent() выполняет переадресацию на определенную страницу Razor
    
- RedirectToRoute()/RedirectToRoutePermanent() выполняет переадресацию по определенному маршруту
    
- StatusCode() возвращает объект StatusCodeResult, то есть посылает статусный код
    
- Unauthorized() возвращает объект UnauthorizedResult, то есть статусный код ошибки 401
    
- Для отправки json нет определенного метода, но определен специальный тип JsonResult, в конструктор которого можно передавать отправляемый код json.
    

Пусть у нас есть страница Razor Index.cshtml и код связанной модели IndexModel в файле Index.cshtml.cs.

Например, на странице Index.cshtml определим следующий код:

```html
@page
 
@model RazorPagesApp.Pages.IndexModel
 
<h2>Hello Razor Pages!</h2>
```

Здесь выводится заголовок "Hello Razor Pages!".

А в файле Index.cshtml.cs определим следующий код:

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        public IActionResult OnGet()
        {
            return Content("Hello BW321");
        }
    }
}
```

Теперь метод OnGet() возвращает объект IActionResult. В частности, он вызывает другой метод - Content(). В этот метод передается отправляемое клиенту текстовое содержимое. Результатом метода `Content()` является объект `ContentResult` - реализация интерфейса IActionResult.

В итоге при запуске приложения мы увидим в браузере не то содержимое, которое определено на странице Index.cshtml, а тот текст, который отправляется из метода Content()

То есть в данном случае страница Razor никак не задействуется. Где это может пригодиться? Это может пригодиться в ситуациях, когда нам надо возвратить клиенту результат, отличный от страницы razor, например, статусный код, какой-то контент в формате json, отправить файл, сделать переадресацию или что-то другое.

В то же время мы можем также явным образом возвратить код страницы Razor с помощью метода Page():

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        public IActionResult OnGet()
        {
            return Page();
        }
    }
}
```


Использование типа IActionResult в качестве возвращаемого значения может пригодиться также, когда необходимо возвратить разные результаты в зависимости от условий. Например:

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        public IActionResult OnGet(string? name)
        {
            if(name != null) 
	            return Content($"Hello {name}");
            return Page();
        }
    }
}
```

В данном случае если при обращении к странице через строку запроса передан параметр name, то возвращается результат метода Content(). Если параметр name не установлен, то возвращается связанная страница Razor.