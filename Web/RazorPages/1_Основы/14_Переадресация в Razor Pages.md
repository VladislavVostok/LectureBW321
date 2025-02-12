### Переадресация на страницу Razor

Для переадресации на страницу Razor применяются методы RedirectToPage() и RedirectToPagePermanent(). Метод RedirectToPage() выполняет временную переадресацию, отправляя статусный код 302. А метод RedirectToPagePermanent() выполняет постоянную переадресацию и отправляет статусный код 301.

Рассмотрим варианты метода `RedirectToPage()`:

- `RedirectToPage()`: выполняет переадресацию на текущую страницу, отправляя статусный код 302
    
- `RedirectToPage(object routeValues)`: выполняет переадресацию на текущую страницу, отправляя статусный код 302 и объект с параметрами для маршрутов
    
- `RedirectToPage(string pageName)`: выполняет переадресацию на определенную страницу
    
- `RedirectToPage(string pageName, object routeValues)`: выполняет переадресацию на определенную страницу, отправляя объект с параметрами маршрута
    
- `RedirectToPage(string pageName, string pageHandler)`: выполняет переадресацию на определенную страницу pageName и обращается к ее обработчику pageHandler
    
- `RedirectToPage(string pageName, string pageHandler, string fragment)`: выполняет переадресацию на определенную страницу pageName и обращается к ее обработчику pageHandler, добавляя к адресу переадресации фрагмент fragment
    
- `RedirectToPage(string pageName, string pageHandler, object routeValues, string fragment)`: выполняет переадресацию на определенную страницу pageName и обращается к ее обработчику pageHandler, добавляя к адресу переадресации фрагмент fragment и отправляя объект с параметрами маршрута
    

Метод RedirectToPagePermanent() имеет аналогичные версии.

Обычно применяется версия метода, которая получает страницу Razor, на которую надо выполнить переадресацию. Например, пусть у нас в проекте есть две страницы New.cshtml и Old.cshtml.

На странице New.cshtml определим какое-нибудь простейшее содержимое:

```
@page
 
<h2>New Page</h2>
```

А на странице Old.cshtml определим редирект на New.cshtml:

```
@page
 
<h2>Old Page</h2>
 
@functions{
    public IActionResult OnGet()
    {
        return RedirectToPage("New");
    }
}
```

В данном случае для краткости метод OnGet, который обрабатывает GET-запросы и выполняет редирект на страницу Old, определен непосредственно на странице Razor, но также можно было бы сделать редирект и в модели страницы.

И в этом случае при обращении по адресу "/Old" произойдет редирект на адрес "/New".

Стоит отметить, что передаваемый путь объединяется с путем к текущей странице, из которой вызывается данный метод. Рассмотрим некоторые возможные сопоставления передаваемых путей и страниц Razor. Например, страница Old.cshtml располагается в папке Pages/Products
![[2.44.png]]

Допустим, из этой страницы мы выполняем редирект на страницу New.cshtml:

| Вызов метода             | На какую страницу ведет |
| ------------------------ | ----------------------- |
| RedirectToPage("/New")   | Pages/New               |
| RedirectToPage("./New")  | Pages/Products/New      |
| RedirectToPage("../New") | Pages/New               |
| RedirectToPage("New")    | Pages/Products/New      |

#### Передача параметров маршрута

Рассмотрим передачу параметров маршрута. Допустим, страница New.cshtml принимает два параметра маршрута:

```
@page "{name}/{age}"
 
<h2>Person</h2>
<p>Name: @RouteData.Values["name"]</p>
<p>Age: @RouteData.Values["age"]</p>
```

Для передачи значений для параметров name и age мы могли бы на странице Old.cshtml выполнить следующую переадресацию:

```
@page
 
@functions{
    public IActionResult OnGet()
    {
        return RedirectToPage("/New", new{name="Sam", age=28});
    }
}
```

### Переадресация на другой адрес

Вместо переадресации на конкретную страницу можно выполнить переадресацию на любой адрес. Для временной переадресации применяется метод Redirect, в который передается адрес для перехода:

```
public IActionResult OnGet()
{
    return Redirect("New/Sam/23");
}
```

В данном случае идет переадресация на локальный адрес "New/Sam/23".

Ну и кроме того, также можно обращаться к внешнему ресурсу:

```
public IActionResult OnGet()
{
    return Redirect("https://github.com");
}
```

Для постоянной переадресации подобным образом используется метод RedirectPermanent. Принцип его применения тот же самый:

```
public IActionResult OnGet()
{
    return RedirectPermanent("New/Sam/23");
}
```

Для обращения к локальным адресам в нашей системе мы можем использовать класс LocalRedirectResult. Для создания временной переадресации применяется метод LocalRedirect(), а для создания постоянной переадресации - метод LocalRedirectPermanent

Эти методы также принимают адрес ресурса:

```
public IActionResult OnGet()
{
    return LocalRedirect("~/Home/About");
}
```