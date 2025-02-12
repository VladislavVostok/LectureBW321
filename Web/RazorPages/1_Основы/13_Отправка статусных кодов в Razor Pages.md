Нередко возникает необходимость отправить в ответ на запрос какой-либо статусный код. Например, если пользователь пытается получить доступ к ресурсу, который недоступен, или для которого у пользователя нету прав. Либо если просто нужно уведомить браузер пользователя с помощью статусного кода об успешном выполнении операции, как иногда применяется в ajax-запросах. Для этого на страницах Razor и их моделях можно применять различные методы и классы результатов.

### StatusCodeResult

StatusCodeResult позволяет отправить любой статусный код клиенту. Для создания этого результата используется метод StatusCode(), в который передается отправляемый код статуса.

Например, у нас есть страница Razor Index.cshtml и код связанной модели IndexModel в файле Index.cshtml.cs:

В файле Index.cshtml.cs определим следующую модель:

```
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        public IActionResult OnGet(string? name)
        {
            if(name != "admin") return StatusCode(401);
            return Page();
        }
    }
}
```

В данном случае, если значение параметра name не равно строке "admin", то посылаем в ответ пользователю статусный код 401, который говорит о том, что доступ запрещен. Иначе возвращаем страницу Razor.

На самой странице Index.cshtml определим какое-нибудь простейшее содержимое для тестирования:

```
@page 
 
@model RazorPagesApp.Pages.IndexModel
<h2>Hello BW321</h2>
```

То есть если странице не будет передан параметр name со значением "admin", то пользователь увидит сообщение об ошибке 401. Иначе же ему отобразиться заголовок из страницы Razor.

Подобным образом мы можем послать браузеру любой другой статусный код. Но для отдельных кодов статуса предназначены свои отдельные классы.

### UnauthorizedResult и UnauthorizedObjectResult

UnauthorizedResult посылает код 401, уведомляя пользователя, что он не авторизован для доступа к ресурсу:

```
public IActionResult OnGet(string? name)
{
    if(name != "admin") return Unauthorized();
    return Page();
}
```

Для создания ответа используется метод Unauthorized().

UnauthorizedObjectResult также посылает код 401, только позволяет добавить в ответ некоторый объект с информацией об ошибке. Конструктор этого класса принимает произвольный объект, который отправляется клиенту в формате json:

```
public IActionResult OnGet(string? name)
{
    if(name != "admin") 
        return new UnauthorizedObjectResult(new {Message="Unauthorized access"});
    return Page();
}
```

### HttpNotFoundResult и HttpNotFoundObjectResult

NotFoundResult и NotFoundObjectResult посылает код 404, уведомляя браузер о том, что ресурс не найден. Второй класс в дополнении к статусному коду позволяет отправить доплнительную информацию, которая потом отобразится в браузере.

Объекты обоих классов создаются методом NotFound. Для первого класса - это метод без параметров, для второго класса - метод, который в качестве параметра принимает отправляемую информацию. Например, используем NotFoundObjectResult:

```
public IActionResult OnGet(string? name)
{
    // если строка name пуста или равна null
    if(string.IsNullOrEmpty(name)) return NotFound("Resource not found");
    return Page();
}
```

### BadResult и BadObjectResult

BadResult и BadObjectResult посылают код 400, что говорит о том, что запрос некорректный. Второй класс в дополнении к статусному коду позволяет отправить доплнительную информацию, которая потом отобразится в браузере.

Эти классы можно применять, например, если в запросе нет каких-то параметров или данные представляют совсем не те типы, которые мы ожидаем получить, и т.д.

Объекты обоих классов создаются методом BadRequest. Для первого класса - это метод без параметров, для второго класса - метод, который в качестве параметра принимает отправляемую информацию:

```
public IActionResult OnGet(string? name)
{
    if (string.IsNullOrEmpty(name)) return BadRequest("Name undefined");
    return Page();
}
```

### OkResult и OkObjectResult

OkResult и OkObjectResult посылают код 200, уведомляя об успешном выполнении запроса. Первый класс просто отправляет статусный код 200:

```
public IActionResult OnGet(string? name)
{
    // если параметр name передан и это не пустая строка, то посылаем статусный код 200
    if(!string.IsNullOrEmpty(name))  return new OkResult();
    return Page();
}
```

Второй класс в дополнении к статусному коду позволяет отправить доплнительную информацию, которая передается через конструктор и которая отправляется в формате json:

```
public IActionResult OnGet(string? name)
{
    // если параметр name передан и это не пустая строка, то посылаем объект со свойством username
    if(!string.IsNullOrEmpty(name)) 
        return new OkObjectResult(new {username = name});
    return Page();
}
```