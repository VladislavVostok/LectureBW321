## Отправка файлов в Razor Pages

Для отправки клиенту файлов предназначен абстрактный класс FileResult, функционал которого реализуется в классах-наследниках:

- FileContentResult: отправляет клиенту массив байтов, считанный из файла
    
- VirtualFileResult: представляет простую отправку файла напрямую с сервера по виртуальному пути
    
- FileStreamResult: создает поток - объект System.IO.Stream, с помощью которого считывает и отправляет файл клиенту
    
- PhysicalFileResult: также отправляет файл с сервера, но для отправки используется реальный физический путь
    

Во первых трех случаях для отправки файлов применяется метод File(), а для создания объекта PhysicalFileResult используется метод PhysicalFile(). Только в зависимости от выбранного способа используется соответствующая перегруженная версия этого метода.

Для примера добавим в корень проекта папку Files, в которой находится файл hello.txt:
![[2.40.png]]

Для файла hello.txt установим копирование в выходной каталог.

### Загрузка файла по пути. PhysicalFileResult

Воспользуемся `PhysicalFileResult` для отправки файла hello.txt клиенту:

```
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
 
namespace RazorPagesApp.Pages
{
    public class IndexModel : PageModel
    {
        public IActionResult OnGet()
        {
            // Путь к файлу
            string file_path = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "Files/hello.txt");
            // Тип файла - content-type
            string file_type = "text/plain";
            // Имя файла - необязательно
            string file_name = "hello.txt";
            return PhysicalFile(file_path, file_type, file_name);
        }
    }
}
```

Чтобы получить полный физический путь каталога относительно проекта, воспользуемся свойством AppDomain.CurrentDomain.BaseDirectory, которое возвращает путь к папке, где находится текущее приложение. И при обращении к странице будет загружаться файл hello.txt.

### Загрузка массива байтов

Похожим образом работает и класс FileContentResult, только используется метод File(), а вместо имени файла передается массив байтов, в который был считан файл:

```
// Отправка массива байтов
public IActionResult OnGet()
{
    string path = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "Files/hello.txt");
    byte[] mas = System.IO.File.ReadAllBytes(path);
    string file_type = "text/plain";
    string file_name = "hello2.txt";
    return File(mas, file_type, file_name);
}
```

### Отправка потока. FileStreamResult

Если мы хотим возвратить объект FileStreamResult, то в качестве первого аргумента в методе File идет объект Stream для отправляемого файла:

```
// Отправка потока
public IActionResult OnGet()
{
    string path = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "Files/hello.txt");
    FileStream fs = new FileStream(path, FileMode.Open);
    string file_type = "text/plain";
    string file_name = "hello3.txt";
    return File(fs, file_type, file_name);
}
```

### VirtualFileResult

VirtualFileResult работает похожим образом, только возвращает файл по виртуальному пути. Здесь надо учитывать, что по умолчанию все пути к файлам в данном случае будут сопоставляться с папкой wwwroot. То есть нам надо помещать папки с файлами или отдельные файлы в каталог wwwroot:

```
public IActionResult OnGet()
{
    return File("Files/hello.txt", "text/plain", "hello4.txt");
}
```

В данном случае предполагается, что файл "hello.txt" располагается в папке "wwwroot/Files/".
![[2.41.png]]


Во всех выше перечисленных случаях использование имени файла в качестве третьего параметра метода File/PhysicalFile необязательно. А вот тип файла обязательно надо передавать. Но подобное поведение может быть не всегда удобным: мы можем точно не знать тип отправляемых файлов, или файлы представляют самые разные типы. И в этом случае мы можем использовать универсальный тип application/octet-stream.