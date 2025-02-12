
### Шаг 1: Создание проекта ASP.NET Core

1. **Откройте Visual Studio** и выберите **Create a new project**.
2. Выберите шаблон **ASP.NET Core Web App (Model-View-Controller)**.
3. Дайте проекту имя, например `<ПриемлемоеНазвание>`.
4. В настройках аутентификации выберите **Individual User Accounts**, чтобы включить встроенную систему авторизации с ASP.NET Identity.
5. Нажмите **Create**.

### Шаг 2: Настройка подключения SqlServer

1. Откройте **Package Manager Console** и установите пакет для работы с SqlServer:

```sh
Install-Package Microsoft.EntityFrameworkCore.SqlServer
Install-Package Microsoft.EntityFrameworkCore.Tools
```
2. В файле **appsettings.json** настройте строку подключения для SqlServer, можно использовать несколько вариантов подключения, т.к. мы будем менять различные среды, которые окружают ваш код. Допустим такими средами будут ваш рабочий компьютер, на котором ведётся разработка и VPS сервер, который будет доступен вашим пользователям, такой сервер называется Продуктовый сервер:

```json
{
"ConnectionStrings": {
  "PRODConnectionString": "Data Source=DESKTOP-T9QBO67;Initial Catalog=ecomerce_shop;Trust Server Certificate=True;User Id=sa;Password=shalom***;",
  "DEVConnectionString": "Data Source=DESKTOP-T9QBO67;Initial Catalog=ecomerce_shop;Trust Server Certificate=True;User Id=sa;Password=shalom***;"
}
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  }
}

```
Перед тем как производить миграцию нужно создать базу данных в СУБД Microsoft SQL Server.

В дальнейшем мы будем прятать важные данные, такие как API ключи от других сервисов и различные подключения к базе данных в секретных местах вашего сервера.

---
### Шаг 3. Прячем данные.
#### 1. Secret Manager

**Secret Manager** — это инструмент, который предоставляет способ безопасного хранения секретов, таких как строки подключения и API ключи, во время разработки приложений на платформе .NET. Он позволяет избежать хранения чувствительных данных в коде или в файлах конфигурации, что снижает риски утечки информации. Давайте рассмотрим его использование более подробно.

##### Основные характеристики Secret Manager
1. **Безопасное хранилище**: Secret Manager хранит секреты в пользовательском профиле, что делает их недоступными для других пользователей на том же компьютере.
2. **Простота использования**: Предоставляет удобный интерфейс командной строки для управления секретами.
3. **Изоляция секретов**: Секреты хранятся отдельно от проекта, что позволяет использовать разные значения для разных сред (например, для разработки и производства).

##### Как использовать Secret Manager

###### 1. Инициализация Secret Manager

Первым делом вам нужно инициализировать Secret Manager для вашего проекта. Для этого выполните команду в терминале в корне проекта:

```sh
dotnet user-secrets init
```

Эта команда добавит в ваш проект файл `*.csproj` с тегом, который указывает, что для этого проекта используются секреты. Это также создает хранилище для секретов, связанное с вашим проектом.

###### 2. Добавление секретов

Чтобы добавить секрет, используйте команду:

```sh
dotnet user-secrets set "MySecretKey" "MySecretValue"
```

Вы можете добавлять сколько угодно секретов. Пример:


```sh
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=myServer;Database=myDB;User Id=myUser;Password=myPassword;"
```

###### 3. Просмотр секретов

Чтобы просмотреть все добавленные секреты, выполните команду:

```sh
dotnet user-secrets list
```

Это покажет список всех секретов, связанных с вашим проектом.

###### 4. Удаление секретов

Если вам нужно удалить секрет, используйте команду:

```sh
dotnet user-secrets remove "MySecretKey"
```

###### 5. Использование секретов в приложении

Для доступа к секретам в вашем приложении, вы должны сначала добавить их в конфигурацию. Это можно сделать в `Program.cs` следующим образом:

```cs
var builder = WebApplication.CreateBuilder(args);

// Загрузка секретов
builder.Configuration.AddUserSecrets<Program>();

// Теперь вы можете использовать конфигурацию
var app = builder.Build();
string connectionString = builder.Configuration["ConnectionStrings:DefaultConnection"];

```

###### Примечания

- **Локальная разработка**: Secret Manager предназначен только для локальной разработки. Он не следует использовать в производственной среде.
- **Не включайте секреты в систему контроля версий**: Убедитесь, что файлы конфигурации, содержащие секреты, не включаются в систему контроля версий. Вы можете использовать `.gitignore`, чтобы исключить такие файлы.
- **Проверка секретов**: Убедитесь, что все разработчики, работающие над проектом, имеют доступ к необходимым секретам. Это можно сделать, сохраняя их в отдельном документе или предоставляя инструкцию по их добавлению.

##### Пример проекта с Secret Manager

Вот простой пример, как вы можете использовать Secret Manager в вашем проекте:

1. Создайте новый проект:

```sh
dotnet new console -n SecretManagerExample 
cd SecretManagerExample
```

2. Инициализируйте Secret Manager:

```
dotnet user-secrets init
```

3. Добавьте секрет:

```sh
dotnet user-secrets set "MySecret" "SuperSecretValue"
```

4. Измените `Program.cs` для чтения секрета:

```cs
using Microsoft.Extensions.Configuration;

var builder = new ConfigurationBuilder()
    .AddUserSecrets<Program>();

var configuration = builder.Build();

string mySecret = configuration["MySecret"];
Console.WriteLine($"My secret is: {mySecret}");

```

5. Запустите проект:

```sh
dotnet run
```

Использование переменных окружения для хранения конфиденциальной информации в приложениях ASP.NET Core является распространенным и безопасным способом управления секретами, такими как строки подключения, API ключи и другие чувствительные данные. Этот подход позволяет отделить конфигурацию от кода, что значительно упрощает управление настройками в различных средах (разработка, тестирование, производство).

---
#### 2. Переменные окружения
##### Преимущества использования переменных окружения

1. **Безопасность**: Переменные окружения хранятся вне кода и конфигурационных файлов, что снижает риск утечки конфиденциальной информации.
2. **Гибкость**: Легко изменять конфигурацию без необходимости модифицировать файлы проекта.
3. **Совместимость с контейнерами**: Подход хорошо интегрируется с контейнерами (например, Docker), где переменные окружения часто используются для передачи конфигурации в приложение.
4. **Простота**: Установка переменных окружения на сервере или локальной машине — это простая и быстрая процедура.

##### Настройка переменных окружения

###### 1. Установка переменных окружения

Переменные окружения можно устанавливать различными способами в зависимости от операционной системы.

- **Windows**: Откройте командную строку и выполните:


```sh
setx MySecret "MySecretValue"
```

После этого вам может понадобиться перезапустить вашу среду разработки или командную строку, чтобы изменения вступили в силу.

- **Linux и macOS**: Вы можете использовать терминал для установки переменных окружения:


```sh
export MySecret="MySecretValue"
```

Для того чтобы переменные сохранялись после перезапуска, вы можете добавить их в файл конфигурации оболочки, например, в `~/.bashrc` или `~/.bash_profile`.


##### Использование переменных окружения в ASP.NET Core

После установки переменных окружения вы можете получить доступ к ним в своем приложении. Для этого нужно просто добавить чтение переменных окружения в конфигурацию вашего приложения.

В файле `Program.cs` добавьте следующий код:

```cs
var builder = WebApplication.CreateBuilder(args);

// Добавление переменных окружения в конфигурацию
builder.Configuration.AddEnvironmentVariables();

var app = builder.Build();

// Получение значения переменной окружения
string mySecret = builder.Configuration["MySecret"];
Console.WriteLine($"My secret is: {mySecret}");

```

##### Пример использования

Вот полный пример приложения, использующего переменные окружения:

1. Создайте новый проект:

```sh
dotnet new console -n EnvironmentVariablesExample cd EnvironmentVariablesExample
```

2. Установите переменную окружения:

- **Windows**:

```sh
setx MySecret "SuperSecretValue"
```

- **Linux или macOS**:

```sh
export MySecret="SuperSecretValue"
```

3. Измените `Program.cs`:

```cs
var builder = WebApplication.CreateBuilder(args);

// Добавление переменных окружения в конфигурацию
builder.Configuration.AddEnvironmentVariables();

var app = builder.Build();

// Получение значения переменной окружения
string mySecret = builder.Configuration["MySecret"];
Console.WriteLine($"My secret is: {mySecret}");

```

4. Запустите проект:

```sh
dotnet run
```

##### Использование переменных окружения в Docker

Если вы разворачиваете приложение в контейнере Docker, переменные окружения можно передать в контейнер с помощью флага `-e`:

```sh
docker run -e MySecret="SuperSecretValue" myapp
```

##### Заключение

Использование переменных окружения — это мощный и безопасный способ управления конфигурацией в приложениях ASP.NET Core. Этот подход позволяет обеспечить гибкость и безопасность, что делает его идеальным для различных сред развертывания. Если у вас есть дополнительные вопросы или нужно больше информации о конкретных аспектах работы с переменными окружения, дайте знать!

---


#### 3. ProtectedJson — Интеграция конфигурации и защиты данных в ASP.NET Core

###### 1. Создаём директорию в проекте Security и создаём класс ProtectedJsonConfigurationProvider.cs

```cs
using Microsoft.AspNetCore.DataProtection;
using System.Text.Json;

namespace dotNetShop.Security
{
	public class ProtectedJsonConfigurationProvider : ConfigurationProvider
	{
		// IDataProtector используется для защиты данных, таких как шифрование и дешифрование
		private readonly IDataProtector DataProtector;
        private readonly string FilePath; // Новый член для хранения пути к файлу

                                          
        public static readonly string DataProtectionPurpose = "ProtectedJsonConfiguration";   // Строковая константа, определяющая цель защиты данных

        // Конструктор провайдера ProtectedJsonConfigurationProvider принимает источник конфигурации
        public ProtectedJsonConfigurationProvider(string filePath, ProtectedJsonConfigurationSource source)
		{
			// Если источник имеет действие по настройке защиты данных
			if (source.DataProtectionBuildAction != null)
			{
				// Создает новую коллекцию сервисов
				var services = new ServiceCollection();

				// Настраивает защиту данных с помощью предоставленного действия
				source.DataProtectionBuildAction(services.AddDataProtection());

				// Создает провайдер сервисов из коллекции, содержащей защиту данных
				source.ServiceProvider = services.BuildServiceProvider();
			}
			else if (source.ServiceProvider == null)
			{
				// Если нет провайдера сервисов или действия для настройки защиты данных, бросает исключение
				throw new ArgumentNullException(nameof(source.ServiceProvider));
			}

			// Получает IDataProtector из провайдера сервисов и задает цель защиты данных
			DataProtector = source.ServiceProvider
				.GetRequiredService<IDataProtectionProvider>()
				.CreateProtector(DataProtectionPurpose);

            FilePath = filePath;
        }

        // Метод Load загружает данные конфигурации и расшифровывает зашифрованные значения
        public override void Load()
        {
            // Проверяет наличие конфигурационного файла, указанного в source.Path
            if (!File.Exists(FilePath))
            {
                throw new FileNotFoundException($"JSON configuration file '{FilePath}' not found.");
            }

            // Считывает содержимое JSON-файла в строку
            var jsonContent = File.ReadAllText(FilePath);

            // Десериализует JSON-содержимое в словарь
            //var data = JsonSerializer.Deserialize<Dictionary<string, string>>(jsonContent);
            //if (data == null) return;

            // Используем JsonDocument для обработки сложной структуры JSON
            using (var document = JsonDocument.Parse(jsonContent))
            {
                var data = new Dictionary<string, string>();
                ProcessJsonElement(document.RootElement, data, null);

                // Обрабатывает каждый ключ-значение
                foreach (var kvp in data)
                {
                    var key = kvp.Key;
                    var value = kvp.Value;

                    // Проверяет, зашифрованы ли данные (заключены в "Protected:{}")
                    if (!string.IsNullOrEmpty(value) && value.StartsWith("Protected:{") && value.EndsWith("}"))
                    {
                        var encryptedData = value.Substring(11, value.Length - 12);
                        var decryptedData = DataProtector.Unprotect(encryptedData);
                        Data[key] = decryptedData;
                    }
                    else
                    {
                        Data[key] = value;
                    }
                }
            }

        }

        // Рекурсивно обрабатывает элементы JSON, включая вложенные объекты
        private void ProcessJsonElement(JsonElement element, Dictionary<string, string> data, string? parentKey)
        {
            foreach (var property in element.EnumerateObject())
            {
                var key = parentKey == null ? property.Name : $"{parentKey}:{property.Name}";

                if (property.Value.ValueKind == JsonValueKind.Object)
                {
                    // Рекурсивно обрабатывает вложенные объекты
                    ProcessJsonElement(property.Value, data, key);
                }
                else
                {
                    // Добавляет значение в словарь как строку
                    data[key] = property.Value.ToString();
                }
            }
        }

    }
}

```
###### 2. Создаём директорию в проекте Security и создаём класс ProtectedJsonConfigurationSource.cs

```cs
using Microsoft.AspNetCore.DataProtection;
using Microsoft.Extensions.Configuration.Json;
using System.Text.RegularExpressions;

namespace dotNetShop.Security
{
	public class ProtectedJsonConfigurationSource : JsonConfigurationSource
	{
		// Сервисный провайдер, используемый для получения IDataProtector
		public IServiceProvider? ServiceProvider { get; set; }

		// Действие для настройки IDataProtectionBuilder, если ServiceProvider отсутствует
		public Action<IDataProtectionBuilder>? DataProtectionBuildAction { get; set; }

		// Регулярное выражение, используемое для поиска и извлечения защищенных данных в формате Protected:{data}
		public Regex ProtectedRegex { get; set; } = new Regex(@"Protected:{(?<protectedData>.+?)}", RegexOptions.Compiled);

		// Метод Build создает и возвращает экземпляр ProtectedJsonConfigurationProvider
		public override IConfigurationProvider Build(IConfigurationBuilder builder)
		{
			// Проверяет, что указан либо ServiceProvider, либо DataProtectionBuildAction
			if (ServiceProvider == null && DataProtectionBuildAction == null)
			{
				// Если ни ServiceProvider, ни DataProtectionBuildAction не указаны, вызывает исключение
				throw new InvalidOperationException("Необходимо указать либо ServiceProvider, либо DataProtectionBuildAction для ProtectedJsonConfigurationSource.");
			}

			// Возвращает новый провайдер конфигурации с поддержкой защиты данных
			return new ProtectedJsonConfigurationProvider(Path, this);
		}
	}
}

```
###### 3. Вспомогательный класс AppSettings.cs

```cs
namespace dotNetShop.Security
{
	// Пример класса AppSettings для привязки настроек
	public class AppSettings
	{
		public string? MysqlConnectionString { get; set; }
		// Добавьте другие свойства, необходимые для конфигурации приложения
	}
}
```
###### 3.1. Немного о классе `AppSettings.cs`
Класс `AppSettings` представляет собой типизированную обертку для конфигурационных настроек, которые могут быть сохранены в файле `appsettings.json` или других источниках конфигурации. Использование `AppSettings` помогает удобно и безопасно управлять параметрами конфигурации в приложении ASP.NET Core.

###### 3.2. Конкретные примеры использования `AppSettings`

Представим, что у нас есть настройки, которые определяют параметры подключения к API, базу данных, лимиты или пути для хранения данных. Все эти параметры можно добавить в `appsettings.json` и привязать к `AppSettings`, чтобы затем легко их использовать в коде.
###### Пример 1: Настройки API

Предположим, что приложение взаимодействует с внешним API. Эти настройки могут храниться в `appsettings.json`:

**appsettings.json**

```json
{
  "AppSettings": {
    "ApiUrl": "https://api.example.com",
    "ApiKey": "12345-abcde-67890-fghij"
  }
}
```

**AppSettings.cs**


```cs
public class AppSettings
{
    public string ApiUrl { get; set; } // Базовый URL API
    public string ApiKey { get; set; } // Ключ для доступа к API
}
```

**Использование в коде:**

```cs
public class MyService
{
    private readonly AppSettings _appSettings;

    public MyService(IOptions<AppSettings> appSettings)
    {
        _appSettings = appSettings.Value;
    }

    public void ConnectToApi()
    {
        string url = _appSettings.ApiUrl;
        string apiKey = _appSettings.ApiKey;
        // Логика для подключения к API с использованием этих настроек
    }
}

```

###### Пример 2: Настройки базы данных

Если у приложения несколько баз данных, можно добавить эти параметры в `appsettings.json`:

**appsettings.json**

```json
{
  "AppSettings": {
    "PrimaryDatabaseConnectionString": "Server=myServer;Database=myDb;User Id=myUser;Password=myPassword;",
    "SecondaryDatabaseConnectionString": "Server=myServer2;Database=myDb2;User Id=myUser2;Password=myPassword2;"
  }
}

```

**AppSettings.cs**

```cs
public class AppSettings
{
    public string PrimaryDatabaseConnectionString { get; set; } // Строка подключения для основной БД
    public string SecondaryDatabaseConnectionString { get; set; } // Строка подключения для дополнительной БД
}

```

**Использование в коде:**

```cs
public class DatabaseService
{
    private readonly AppSettings _appSettings;

    public DatabaseService(IOptions<AppSettings> appSettings)
    {
        _appSettings = appSettings.Value;
    }

    public void ConnectToPrimaryDatabase()
    {
        string connectionString = _appSettings.PrimaryDatabaseConnectionString;
        // Логика подключения к основной базе данных
    }
}

```
###### Пример 3: Настройки для хранения данных

Например, пути для временных файлов и файлов отчетов могут быть указаны в `appsettings.json`:

**appsettings.json**

```json
{
  "AppSettings": {
    "TempFilesPath": "/temp",
    "ReportsPath": "/reports"
  }
}

```

**AppSettings.cs**

```cs
public class AppSettings
{
    public string TempFilesPath { get; set; } // Путь для временных файлов
    public string ReportsPath { get; set; } // Путь для отчетов
}

```

**Использование в коде:**

```cs
public class FileService
{
    private readonly AppSettings _appSettings;

    public FileService(IOptions<AppSettings> appSettings)
    {
        _appSettings = appSettings.Value;
    }

    public void SaveReport(string reportData)
    {
        string path = _appSettings.ReportsPath;
        // Логика для сохранения отчета в указанное место
    }
}

```

###### Итог

Класс `AppSettings` позволяет вам:

- **Хранить** конфигурационные параметры в одном месте и менять их без необходимости изменять код.
- **Получать настройки безопасным и типизированным образом** с помощью внедрения зависимостей и системы конфигурации ASP.NET Core.
- **Организовать код** так, чтобы он был удобочитаемым и легко поддерживался.

Этот подход упрощает управление конфигурацией и помогает избежать жестко закодированных значений, улучшая гибкость и переносимость приложения.



### Шаг 4: Создание моделей для Магазина

#### 4.1. Создаём в директории Models классы:

##### Brand.cs
```cs
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;


namespace dotNetShop.Models{
    public class Brand
    {
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        [Key]
        public int Id { get; set; }
        
        [Required]
        [MaxLength(32)]
        public string Name { get; set; }
        
        public ICollection<Product> Products { get; set; }
    }
}
```

##### Category.cs
```cs
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;


namespace dotNetShop.Models
{

    public class Category
    {
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        [Key]
        public int Id { get; set; }

        [Required]
        [MaxLength(100)]
        public string Name { get; set; }

        public string? Description { get; set; }

        public ICollection<Product> Products { get; set; }
    }
}
```

##### Comment.cs
```cs
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace dotNetShop.Models
{
    public class Comment
    {
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        [Key]
        public int Id { get; set; }

        [Required]
        [MaxLength(500)]
        public string Content { get; set; }

        [Required]
        public DateTime CreatedAt { get; set; } = DateTime.UtcNow.ToLocalTime().AddHours(3);

        [Required]
        public string UserName { get; set; }

        public int ProductId { get; set; }

        [ForeignKey("ProductId")]
        public Product Product { get; set; }
    }
}
```
##### Product.cs
```cs
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;



namespace dotNetShop.Models{

    public class Product
    {
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        [Key]
        public int Id { get; set; }
        
        [Required]
        [MaxLength(128)]
        public string Name { get; set; }

        [MaxLength(1024)]
        public string? Description { get; set; }

        [Required]
        public decimal Price { get; set; }
        
        public decimal? DiscountedPrice { get; set; }
        
        public int CategoryId { get; set; }
        
        [ForeignKey("CategoryId")]
        public Category Category { get; set; }

        public int BrandId { get; set; }
        
        [ForeignKey("BrandId")]
        public Brand Brand { get; set; }
        

        public ICollection<ProductImage> Images { get; set; }
        
        public ICollection<Color> AvailableColors { get; set; }

        public ICollection<Comment> Comments{get;set;}
    }
}
```
##### ProductImage.cs
```cs
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;



namespace dotNetShop.Models
{

    public class ProductImage
    {
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        [Key]
        public int Id { get; set; }

        [Required]
        public string Url { get; set; }

        public int ProductId { get; set; }

        [ForeignKey("ProductId")]
        public Product Product { get; set; }
    }
}
```

### Шаг 5: Создаём контекст базы данных
#### 5.1. Создаем директорию data и в ней создаём класс `ShopDBContext.cs`

```cs
using dotNetShop.Models;
using Microsoft.EntityFrameworkCore;

namespace dotNetShop.Data
{
	public class ShopDBContext : DbContext
	{

		public DbSet<Category> Categories { get; set; }
		public DbSet<Product> Products { get; set; }
		public DbSet<Brand> Brands { get; set; }
		public DbSet<Color> Colors { get; set; }
		public DbSet<ProductImage> ProductImages { get; set; }
		public DbSet<Comment> Comments { get; set; }


		public ShopDBContext() { }
		public ShopDBContext(DbContextOptions<ShopDBContext> options) : base(options) {}

		protected override void OnModelCreating(ModelBuilder modelBuilder)
		{
			modelBuilder.Entity<Product>()
			.HasOne(p => p.Category)
			.WithMany(p => p.Products)
			.HasForeignKey(p => p.CategoryId)
			.OnDelete(DeleteBehavior.Cascade);

			modelBuilder.Entity<Product>()
                .HasOne(p => p.Brand)
                .WithMany(b => b.Products)
                .HasForeignKey(p => p.BrandId)
                .OnDelete(DeleteBehavior.Restrict);

            // Связь многие-ко-многим: Product и Color
            modelBuilder.Entity<Product>()
                .HasMany(p => p.AvailableColors)
                .WithMany(c => c.Products)
                .UsingEntity(j => j.ToTable("ProductColors"));

            // Связь один-ко-многим: Product и ProductImage
            modelBuilder.Entity<ProductImage>()
                .HasOne(pi => pi.Product)
                .WithMany(p => p.Images)
                .HasForeignKey(pi => pi.ProductId)
                .OnDelete(DeleteBehavior.Cascade);

            // Связь один-ко-многим: Product и Comment
            modelBuilder.Entity<Comment>()
                .HasOne(c => c.Product)
                .WithMany(p => p.Comments)
                .HasForeignKey(c => c.ProductId)
                .OnDelete(DeleteBehavior.Cascade);






			base.OnModelCreating(modelBuilder);
		}




		public static void SeedData(ShopDBContext context)
		{
			// Создаем категории
			var categories = new List<Category>
		 	{
		 		new Category { Name = "Кроссовки", Description = "Спортивные кроссовки для бега." },
		 		new Category { Name = "Ботинки", Description = "Теплые ботинки для зимы." },
		 		new Category { Name = "Сандалии", Description = "Летние сандалии для отдыха." },
		 		new Category { Name = "Кеды", Description = "Удобные кеды для повседневной носки." },
		 		new Category { Name = "Туфли", Description = "Элегантные туфли для особых случаев." },
		 		new Category { Name = "Сапоги", Description = "Демисезонные сапоги." },
		 		new Category { Name = "Кроссовки для фитнеса", Description = "Кроссовки для занятий спортом." },
		 		new Category { Name = "Теннисные кроссовки", Description = "Кроссовки для тенниса." },
		 		new Category { Name = "Беговые кроссовки", Description = "Кроссовки для длительных пробежек." },
		 		new Category { Name = "Спортивные ботинки", Description = "Спортивные ботинки для активного отдыха." },
		 		new Category { Name = "Детские кроссовки", Description = "Кроссовки для детей." },
		 		new Category { Name = "Кроссовки для футбола", Description = "Кроссовки для футбольных тренировок." },
		 		new Category { Name = "Кроссовки для баскетбола", Description = "Кроссовки для игры в баскетбол." },
		 		new Category { Name = "Сапоги для дождя", Description = "Водонепроницаемые сапоги." },
		 		new Category { Name = "Треккинговые ботинки", Description = "Ботинки для походов." },
		 		new Category { Name = "Слепоны", Description = "Легкие и удобные слепоны." },
		 		new Category { Name = "Кроссовки для йоги", Description = "Удобные кроссовки для занятий йогой." },
		 		new Category { Name = "Кроссовки для тренировок", Description = "Кроссовки для тренировок в зале." },
		 		new Category { Name = "Туристические ботинки", Description = "Ботинки для активного отдыха." },
		 		new Category { Name = "Кроссовки для активного отдыха", Description = "Кроссовки для отдыха и спорта." },
		 		new Category { Name = "Кроссовки для ходьбы", Description = "Кроссовки для прогулок." },
		 		new Category { Name = "Кроссовки для пляжа", Description = "Легкие кроссовки для пляжа." },
		 		new Category { Name = "Кроссовки с амортизацией", Description = "Кроссовки с хорошей амортизацией." },
		 		new Category { Name = "Кроссовки с поддержкой", Description = "Кроссовки с поддержкой стопы." },
		 		new Category { Name = "Кроссовки для занятия бегом", Description = "Кроссовки для занятия бегом." },
		 		new Category { Name = "Кроссовки для спортивной ходьбы", Description = "Кроссовки для спортивной ходьбы." },
		 		new Category { Name = "Кроссовки с дышащей верхней частью", Description = "Кроссовки с хорошей вентиляцией." },
		 		new Category { Name = "Кроссовки с системой быстрого шнурования", Description = "Кроссовки с удобной системой шнуровки." },
		 		new Category { Name = "Кроссовки с защитой от влаги", Description = "Кроссовки, защищающие от влаги." },
		 		new Category { Name = "Кроссовки для зимних видов спорта", Description = "Кроссовки для зимних видов спорта." },
		 		new Category { Name = "Кроссовки для танцев", Description = "Кроссовки для танцев." },
		 		new Category { Name = "Кроссовки с высоким берцем", Description = "Кроссовки с высоким берцем." },
		 		new Category { Name = "Кроссовки для верховой езды", Description = "Кроссовки для верховой езды." },
		 		new Category { Name = "Кроссовки для велоспорта", Description = "Кроссовки для велоспорта." },
		 		new Category { Name = "Кроссовки для скейтбординга", Description = "Кроссовки для скейтбординга." },
		 	};

			context.Categories.AddRange(categories);
			context.SaveChanges();

			// Создаем бренды
			var brands = new List<Brand>
		 	{
		 		new Brand { Name = "Nike" },
		 		new Brand { Name = "Adidas" },
		 		new Brand { Name = "Puma" },
		 		 new Brand { Name = "Reebok" },
		 		 new Brand { Name = "Asics" },
		 		 new Brand { Name = "New Balance" },
		 		 new Brand { Name = "Under Armour" },
		 		 new Brand { Name = "Saucony" },
		 		 new Brand { Name = "Hoka One One" },
		 		 new Brand { Name = "Salomon" },
		 		 new Brand { Name = "Skechers" },
		 		 new Brand { Name = "Merrell" },
		 		 new Brand { Name = "Columbia" },
		 		 new Brand { Name = "Vans" },
		 		 new Brand { Name = "Converse" },
		 		 new Brand { Name = "Dr. Martens" },
		 		 new Brand { Name = "Keen" },
		 		 new Brand { Name = "Timberland" },
		 		 new Brand { Name = "Fila" },
		 		 new Brand { Name = "Diadora" },
		 		 new Brand { Name = "On" },
		 		 new Brand { Name = "Brooks" },
		 		 new Brand { Name = "La Sportiva" },
		 		 new Brand { Name = "Altra" },
		 		 new Brand { Name = "Lowa" },
		 		 new Brand { Name = "Nordica" },
		 		 new Brand { Name = "Garmont" },
		 		 new Brand { Name = "Lacoste" },
		 		 new Brand { Name = "Superga" },
		 		 new Brand { Name = "Sorel" },
		 		 new Brand { Name = "Nautica" },
		 		 new Brand { Name = "Clarks" },
		 	};

			context.Brands.AddRange(brands);
			context.SaveChanges();

			// Создаем цвета
			var colors = new List<Color>
		 	{
		 		new Color { Name = "Красный", HexCode = "#FF0000" },
		 		new Color { Name = "Синий", HexCode = "#0000FF" },
		 		new Color { Name = "Зеленый", HexCode = "#008000" },
		 		 new Color { Name = "Черный", HexCode = "#000000" },
		 		 new Color { Name = "Белый", HexCode = "#FFFFFF" },
		 		 new Color { Name = "Желтый", HexCode = "#FFFF00" },
		 		 new Color { Name = "Оранжевый", HexCode = "#FFA500" },
		 		 new Color { Name = "Фиолетовый", HexCode = "#800080" },
		 		 new Color { Name = "Розовый", HexCode = "#FFC0CB" },
		 		 new Color { Name = "Серый", HexCode = "#808080" },
		 		 new Color { Name = "Коричневый", HexCode = "#A52A2A" },
		 		 new Color { Name = "Бежевый", HexCode = "#F5F5DC" },
		 		 new Color { Name = "Циановый", HexCode = "#00FFFF" },
		 		 new Color { Name = "Лаванда", HexCode = "#E6E6FA" },
		 		 new Color { Name = "Небесно-голубой", HexCode = "#87CEEB" },
		 		 new Color { Name = "Малиновый", HexCode = "#C71585" },
		 		 new Color { Name = "Золотой", HexCode = "#FFD700" },
		 		 new Color { Name = "Серебряный", HexCode = "#C0C0C0" },
		 		 new Color { Name = "Изумрудный", HexCode = "#50C878" },
		 		 new Color { Name = "Терракотовый", HexCode = "#E2725B" },
		 		 new Color { Name = "Бирюзовый", HexCode = "#40E0D0" },
		 		 new Color { Name = "Салатовый", HexCode = "#90EE90" },
		 		 new Color { Name = "Персиковый", HexCode = "#FFDAB9" },
		 		 new Color { Name = "Синевато-серый", HexCode = "#4682B4" },
		 		 new Color { Name = "Лососевый", HexCode = "#FF7F50" },
		 		 new Color { Name = "Морская волна", HexCode = "#2E8B57" },
		 		 new Color { Name = "Песочный", HexCode = "#F4A460" },
		 		 new Color { Name = "Тихий белый", HexCode = "#F5FFFA" },
		 		 new Color { Name = "Графитовый", HexCode = "#474747" },
		 	};


			context.Colors.AddRange(colors);
			context.SaveChanges();

			// Создаем продукты
			var products = new List<Product> {
		 	new Product { Name = "Nike Air Max 270", Description = "Современные кроссовки с отличной амортизацией.", Price = 120.99m, DiscountedPrice = 100.99m, CategoryId = categories[0].Id, BrandId = brands[0].Id },
		 	new Product { Name = "Adidas Ultraboost 21", Description = "Удобные кроссовки для бега с отличной поддержкой.", Price = 150.50m, DiscountedPrice = 120.50m, CategoryId = categories[0].Id, BrandId = brands[1].Id },
		 	new Product { Name = "Puma RS-X", Description = "Модные кроссовки с ярким дизайном.", Price = 110.00m, DiscountedPrice = 95.00m, CategoryId = categories[0].Id, BrandId = brands[2].Id },
		 	 new Product { Name = "Reebok Classic Leather", Description = "Классические кроссовки для повседневной носки.", Price = 80.00m, DiscountedPrice = 70.00m, CategoryId = categories[0].Id, BrandId = brands[3].Id },
		 	 new Product { Name = "Asics Gel-Kayano 27", Description = "Кроссовки для бега с отличной поддержкой.", Price = 160.00m, DiscountedPrice = 140.00m, CategoryId = categories[0].Id, BrandId = brands[4].Id },
		 	 new Product { Name = "New Balance 990v5", Description = "Кроссовки с отличной амортизацией для активных людей.", Price = 175.00m, DiscountedPrice = 150.00m, CategoryId = categories[0].Id, BrandId = brands[5].Id },
		 	 new Product { Name = "Under Armour HOVR Phantom", Description = "Современные кроссовки с технологией HOVR.", Price = 140.00m, DiscountedPrice = 120.00m, CategoryId = categories[0].Id, BrandId = brands[6].Id },
		 	 new Product { Name = "Saucony Triumph 18", Description = "Легкие и комфортные кроссовки для бега.", Price = 160.00m, DiscountedPrice = 145.00m, CategoryId = categories[0].Id, BrandId = brands[7].Id },
		 	 new Product { Name = "Hoka One One Bondi 7", Description = "Кроссовки с максимальной амортизацией.", Price = 180.00m, DiscountedPrice = 160.00m, CategoryId = categories[0].Id, BrandId = brands[8].Id },
		 	 new Product { Name = "Salomon Speedcross 5", Description = "Кроссовки для трейлового бега.", Price = 130.00m, DiscountedPrice = 110.00m, CategoryId = categories[0].Id, BrandId = brands[9].Id },
		 	 new Product { Name = "Skechers Go Walk 5", Description = "Комфортные кроссовки для прогулок.", Price = 90.00m, DiscountedPrice = 75.00m, CategoryId = categories[0].Id, BrandId = brands[10].Id },
		 	 new Product { Name = "Merrell Moab 2", Description = "Треккинговые кроссовки для активного отдыха.", Price = 100.00m, DiscountedPrice = 85.00m, CategoryId = categories[0].Id, BrandId = brands[11].Id },
		 	 new Product { Name = "Columbia Bugaboot Plus IV", Description = "Сапоги для зимних прогулок.", Price = 150.00m, DiscountedPrice = 130.00m, CategoryId = categories[0].Id, BrandId = brands[12].Id },
		 	 new Product { Name = "Vans Old Skool", Description = "Классические кеды для повседневного использования.", Price = 70.00m, DiscountedPrice = 60.00m, CategoryId = categories[0].Id, BrandId = brands[13].Id },
		 	 new Product { Name = "Converse Chuck Taylor", Description = "Знаменитые кеды для молодежи.", Price = 65.00m, DiscountedPrice = 55.00m, CategoryId = categories[0].Id, BrandId = brands[14].Id },
		 	 new Product { Name = "Dr. Martens 1460", Description = "Классические ботинки Dr. Martens.", Price = 160.00m, DiscountedPrice = 140.00m, CategoryId = categories[0].Id, BrandId = brands[15].Id },
		 	 new Product { Name = "Keen Targhee III", Description = "Кроссовки для активного отдыха на природе.", Price = 140.00m, DiscountedPrice = 120.00m, CategoryId = categories[0].Id, BrandId = brands[16].Id },
		 	 new Product { Name = "Timberland 6\" Premium", Description = "Классические сапоги Timberland.", Price = 200.00m, DiscountedPrice = 180.00m, CategoryId = categories[0].Id, BrandId = brands[17].Id },
		 	 new Product { Name = "Fila Disruptor II", Description = "Модные кроссовки с массивной подошвой.", Price = 90.00m, DiscountedPrice = 75.00m, CategoryId = categories[0].Id, BrandId = brands[18].Id },
		 	 new Product { Name = "Diadora N9000", Description = "Кроссовки с итальянским стилем.", Price = 110.00m, DiscountedPrice = 95.00m, CategoryId = categories[0].Id, BrandId = brands[19].Id },
		 	 new Product { Name = "On Cloudstratus", Description = "Кроссовки с двойной амортизацией.", Price = 160.00m, DiscountedPrice = 140.00m, CategoryId = categories[0].Id, BrandId = brands[20].Id },
		 	 new Product { Name = "Brooks Ghost 13", Description = "Кроссовки для бега с отличной амортизацией.", Price = 130.00m, DiscountedPrice = 115.00m, CategoryId = categories[0].Id, BrandId = brands[21].Id },
		 	 new Product { Name = "La Sportiva Ultra Raptor", Description = "Кроссовки для трейлового бега.", Price = 150.00m, DiscountedPrice = 135.00m, CategoryId = categories[0].Id, BrandId = brands[22].Id },
		 	 new Product { Name = "Altra Lone Peak 4.5", Description = "Кроссовки с широкой носочной частью.", Price = 140.00m, DiscountedPrice = 125.00m, CategoryId = categories[0].Id, BrandId = brands[23].Id },
		 	 new Product { Name = "Lowa Renegade GTX", Description = "Треккинговые ботинки с водонепроницаемой мембраной.", Price = 200.00m, DiscountedPrice = 180.00m, CategoryId = categories[0].Id, BrandId = brands[24].Id },
		 	 new Product { Name = "Nordica Speedmachine 3", Description = "Ботинки для горных лыж.", Price = 500.00m, DiscountedPrice = 450.00m, CategoryId = categories[0].Id, BrandId = brands[25].Id },
		 	 new Product { Name = "Garmont Dragontail MNT", Description = "Ботинки для скалолазания.", Price = 200.00m, DiscountedPrice = 180.00m, CategoryId = categories[0].Id, BrandId = brands[26].Id },
		 	 new Product { Name = "Lacoste Carnaby", Description = "Модные кроссовки от Lacoste.", Price = 120.00m, DiscountedPrice = 100.00m, CategoryId = categories[0].Id, BrandId = brands[27].Id },
		 	 new Product { Name = "Superga 2750", Description = "Классические итальянские кеды.", Price = 75.00m, DiscountedPrice = 65.00m, CategoryId = categories[0].Id, BrandId = brands[28].Id },
		 	 new Product { Name = "Sorel Caribou", Description = "Теплые зимние сапоги.", Price = 180.00m, DiscountedPrice = 160.00m, CategoryId = categories[0].Id, BrandId = brands[29].Id }
		 	};

			context.Products.AddRange(products);
			context.SaveChanges();

			// Создаем изображения продуктов
			var productImages = new List<ProductImage>
		 	{
		 		new ProductImage { Url = "https://example.com/nike-air-max-270.jpg", ProductId = products[0].Id },
		 		new ProductImage { Url = "https://example.com/adidas-ultraboost-21.jpg", ProductId = products[1].Id },
		 		new ProductImage { Url = "https://example.com/puma-rs-x.jpg", ProductId = products[2].Id },
		 		 new ProductImage { Url = "https://example.com/reebok-classic-leather.jpg", ProductId = products[3].Id },
		 		 new ProductImage { Url = "https://example.com/asics-gel-kayano-27.jpg", ProductId = products[4].Id },
		 		 new ProductImage { Url = "https://example.com/new-balance-990v5.jpg", ProductId = products[5].Id },
		 		 new ProductImage { Url = "https://example.com/under-armour-hovr-phantom.jpg", ProductId = products[6].Id },
		 		 new ProductImage { Url = "https://example.com/saucony-triumph-18.jpg", ProductId = products[7].Id },
		 		 new ProductImage { Url = "https://example.com/hoka-one-one-bondi-7.jpg", ProductId = products[8].Id },
		 		 new ProductImage { Url = "https://example.com/salomon-speedcross-5.jpg", ProductId = products[9].Id },
		 		 new ProductImage { Url = "https://example.com/skechers-go-walk-5.jpg", ProductId = products[10].Id },
		 		 new ProductImage { Url = "https://example.com/merrell-moab-2.jpg", ProductId = products[11].Id },
		 		 new ProductImage { Url = "https://example.com/columbia-bugaboot-plus-iv.jpg", ProductId = products[12].Id },
		 		 new ProductImage { Url = "https://example.com/vans-old-skool.jpg", ProductId = products[13].Id },
		 		 new ProductImage { Url = "https://example.com/converse-chuck-taylor.jpg", ProductId = products[14].Id },
		 		 new ProductImage { Url = "https://example.com/dr-martens-1460.jpg", ProductId = products[15].Id },
		 		 new ProductImage { Url = "https://example.com/keen-targhee-iii.jpg", ProductId = products[16].Id },
		 		 new ProductImage { Url = "https://example.com/timberland-6-premium.jpg", ProductId = products[17].Id },
		 		 new ProductImage { Url = "https://example.com/fila-disruptor-ii.jpg", ProductId = products[18].Id },
		 		 new ProductImage { Url = "https://example.com/diadora-n9000.jpg", ProductId = products[19].Id },
		 		 new ProductImage { Url = "https://example.com/on-cloudstratus.jpg", ProductId = products[20].Id },
		 		 new ProductImage { Url = "https://example.com/brooks-ghost-13.jpg", ProductId = products[21].Id },
		 		 new ProductImage { Url = "https://example.com/la-sportiva-ultra-raptor.jpg", ProductId = products[22].Id },
		 		 new ProductImage { Url = "https://example.com/altra-lone-peak-4-5.jpg", ProductId = products[23].Id },
		 		 new ProductImage { Url = "https://example.com/lowa-renegade-gtx.jpg", ProductId = products[24].Id },
		 		 new ProductImage { Url = "https://example.com/nordica-speedmachine-3.jpg", ProductId = products[25].Id },
		 		 new ProductImage { Url = "https://example.com/garmont-dragontail-mnt.jpg", ProductId = products[26].Id },
		 	};

			context.ProductImages.AddRange(productImages);
			context.SaveChanges();
			var comments = new List<Comment>
		 	{
		 		// Создаем комментарии
		 		new Comment { Content = "Отличные кроссовки для бега!", UserName = "Алексей", ProductId = products[0].Id },
		 		new Comment { Content = "Очень удобные, рекомендую.", UserName = "Мария", ProductId = products[1].Id },
		 		new Comment { Content = "Стильный дизайн и хорошее качество.", UserName = "Иван", ProductId = products[2].Id },
		 		 new Comment { Content = "Супер комфортные для повседневной носки.", UserName = "Екатерина", ProductId = products[3].Id },
		 		 new Comment { Content = "Потрясающая амортизация!", UserName = "Сергей", ProductId = products[4].Id },
		 		 new Comment { Content = "Отлично подходят для долгих пробежек.", UserName = "Анна", ProductId = products[5].Id },
		 		 new Comment { Content = "Лучшие кроссовки, которые у меня были!", UserName = "Дмитрий", ProductId = products[6].Id },
		 		 new Comment { Content = "Легкие и дышащие, идеально для лета.", UserName = "Ольга", ProductId = products[7].Id },
		 		 new Comment { Content = "Идеальны для походов!", UserName = "Николай", ProductId = products[8].Id },
		 		 new Comment { Content = "С удовольствием ношу каждый день.", UserName = "Елена", ProductId = products[9].Id },
		 		 new Comment { Content = "Хороший выбор для тренировок.", UserName = "Максим", ProductId = products[10].Id },
		 		 new Comment { Content = "Кроссовки для всех случаев жизни.", UserName = "Татьяна", ProductId = products[11].Id },
		 		 new Comment { Content = "Идеальны для зимних прогулок.", UserName = "Станислав", ProductId = products[12].Id },
		 		 new Comment { Content = "Классика, которая никогда не устареет.", UserName = "Ксения", ProductId = products[13].Id },
		 		 new Comment { Content = "Лучший выбор для активных людей.", UserName = "Роман", ProductId = products[14].Id },
		 		 new Comment { Content = "Стильно и удобно.", UserName = "Анастасия", ProductId = products[15].Id },
		 		 new Comment { Content = "Приятные в носке, ничего не жмёт.", UserName = "Виктор", ProductId = products[16].Id },
		 		 new Comment { Content = "Супер для долгих прогулок.", UserName = "Евгений", ProductId = products[17].Id },
		 		 new Comment { Content = "Отличная поддержка стопы.", UserName = "Анна", ProductId = products[18].Id },
		 		 new Comment { Content = "Кроссовки просто супер!", UserName = "Кирилл", ProductId = products[19].Id },
		 		 new Comment { Content = "Сначала не удобно, но потом в них удобно.", UserName = "Светлана", ProductId = products[20].Id },
		 		 new Comment { Content = "Обожаю их!", UserName = "Игорь", ProductId = products[21].Id },
		 		 new Comment { Content = "Отличный выбор для бега по пересеченной местности.", UserName = "Лариса", ProductId = products[22].Id },
		 		 new Comment { Content = "Удобно, рекомендую.", UserName = "Денис", ProductId = products[23].Id },
		 		 new Comment { Content = "Самые лучшие кроссовки!", UserName = "Наталья", ProductId = products[24].Id },
		 		 new Comment { Content = "Хорошие кроссовки, стоят своих денег.", UserName = "Петр", ProductId = products[25].Id },
		 		 new Comment { Content = "Просто великолепно!", UserName = "Галина", ProductId = products[26].Id },
		 		 new Comment { Content = "Подходят для любых активностей.", UserName = "Владимир", ProductId = products[27].Id },
		 		 new Comment { Content = "Не пожалел, что купил.", UserName = "Елена", ProductId = products[28].Id },
		 		 new Comment { Content = "Кроссовки очень легкие и удобные.", UserName = "Ирина", ProductId = products[29].Id },
		 	};

			context.Comments.AddRange(comments);
			context.SaveChanges();
		}
	}
}
```


### Шаг 6: Миграции и обновление базы данных

#### 1. Откройте **Package Manager Console** или откройте **CMD** в папке с проектом.
    
#### 2. Выполните команду для создания миграций (**Package Manager Console**):

```sh
Add-Migration <Имя миграции задаёте сами>
```
или

```bash
dotnet ef migrations add <Имя миграции задаёте сами>
```
если команда выше не отработала, тогда:

```sh
dotnet tool install --global dotnet-ef 
```
данная команда установит dotnet-ef инструменты для миграции

#### 3. Примените миграции:

C помощью  (**Package Manager Console**)

```sh
Update-Database
```

или

C помощью (**CMD**)

```sh
dotnet ef database update
```

### Шаг 7: Создаём контроллер

Создайте контроллер **CommentController.cs** в папке **Controllers**:

```cs
using dotNetShop.Data;
using dotNetShop.Models;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace dotNetShop.Controllers
{

	public class ProductCommentViewModel
	{
		public Product Product { get; set; }
		public Comment NewComment { get; set; } // Модель для нового комментария 
		public List<Comment> Comments { get; set; } // Список всех комментариев к продукту }
	}
	public class OverviewViewModel
	{
		public List<BrandCountViewModel> Brands { get; set; }
		public List<ColorCountViewModel> Colors { get; set; }
		public List<CategoryCountViewModel> Categories { get; set; }
		public List<Product> Products { get; set; }
	}

	public class BrandCountViewModel
	{
		public string BrandName { get; set; }
		public int ProductCount { get; set; }
	}

	public class ColorCountViewModel
	{
		public string ColorName { get; set; }
		public int ProductCount { get; set; }
	}

	public class CategoryCountViewModel
	{
		public string CategoryName { get; set; }
		public int ProductCount { get; set; }
	}


	public class ShopController : Controller
	{

		private readonly ShopDBContext _context;

		public ShopController(ShopDBContext context)
		{
			_context = context;
		}

		// GET: ShopController
		public async Task<IActionResult> Index()
		{
			var brands = await _context.Brands
				.Include(b => b.Products) // Загрузка связанных продуктов
			.ToListAsync();

			var colors = await _context.Colors
				.Include(c => c.Products) // Загрузка связанных продуктов
			.ToListAsync();

			var categories = await _context.Categories
				.Include(c => c.Products) // Загрузка связанных продуктов
			.ToListAsync();

			var products = await _context.Products.ToListAsync();

			var viewModel = new OverviewViewModel
			{
				Brands = brands.Select(b => new BrandCountViewModel
				{
					BrandName = b.Name,
					ProductCount = b.Products.Count()
				}).ToList(),
				Colors = colors.Select(c => new ColorCountViewModel
				{
					ColorName = c.Name,
					ProductCount = c.Products.Count()
				}).ToList(),
				Categories = categories.Select(c => new CategoryCountViewModel
				{
					CategoryName = c.Name,
					ProductCount = c.Products.Count()
				}).ToList(),
				Products = products
			};

			return View(viewModel);
		}

		// GET: ShopController/Details/5
		// GET: ProductComments/ProductDetails/{productId}
		public async Task<IActionResult> Details(int productId)
		{

			var product = await _context.Products
				.Include(p => p.Comments) // Загрузка связанных комментариев
				.FirstOrDefaultAsync(p => p.Id == productId);

			if (product == null)
			{
				return NotFound();
			}

			var viewModel = new ProductCommentViewModel
			{
				Product = product,
				NewComment = new Comment(), // Пустая модель для нового комментария
				Comments = product.Comments.OrderByDescending(c => c.CreatedAt).ToList()
			};

			return View(viewModel);
		}

        // POST: ProductComments/AddComment
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> AddComment(ProductCommentViewModel model)
        {
            if (ModelState.IsValid)
            {
                model.NewComment.ProductId = model.Product.Id;
                model.NewComment.CreatedAt = DateTime.UtcNow;

                _context.Comments.Add(model.NewComment);
                await _context.SaveChangesAsync();

                return RedirectToAction("ProductDetails", new { productId = model.Product.Id });
            }

            // Если валидация не прошла, снова загружаем комментарии
            model.Comments = await _context.Comments
                .Where(c => c.ProductId == model.Product.Id)
                .OrderByDescending(c => c.CreatedAt)
                .ToListAsync();

            return View("ProductDetails", model);
        }
    }
}


```
### Шаг 8: Настройка представлений

#### Index.cshtml

```cs
@model dotNetShop.Controllers.OverviewViewModel

<h2>Обзор продуктов</h2>

<!-- Таблица брендов -->
<h3>Бренды и количество товаров</h3>
<table class="table">
	<thead>
		<tr>
			<th>Бренд</th>
			<th>Количество товаров</th>
		</tr>
	</thead>
	<tbody>
		@foreach (var brand in Model.Brands)
		{
			<tr>
				<td>@brand.BrandName</td>
				<td>@brand.ProductCount</td>
			</tr>
		}
	</tbody>
</table>

<!-- Таблица цветов -->
<h3>Цвета и количество товаров</h3>
<table class="table">
	<thead>
		<tr>
			<th>Цвет</th>
			<th>Количество товаров</th>
		</tr>
	</thead>
	<tbody>
		@foreach (var color in Model.Colors)
		{
			<tr>
				<td>@color.ColorName</td>
				<td>@color.ProductCount</td>
			</tr>
		}
	</tbody>
</table>

<!-- Таблица категорий -->
<h3>Категории и количество товаров</h3>
<table class="table">
	<thead>
		<tr>
			<th>Категория</th>
			<th>Количество товаров</th>
		</tr>
	</thead>
	<tbody>
		@foreach (var category in Model.Categories)
		{
			<tr>
				<td>@category.CategoryName</td>
				<td>@category.ProductCount</td>
			</tr>
		}
	</tbody>
</table>

<!-- Таблица всех товаров -->
<h3>Все товары</h3>
<table class="table">
	<thead>
		<tr>
			<th>Название</th>
			<th>Цена</th>
			<th>Бренд</th>
			<th>Категория</th>
		</tr>
	</thead>
	<tbody>
		@foreach (var product in Model.Products)
		{
			<tr>
				<td>@product.Name</td>
				<td>@product.Price.ToString("C")</td>
				<td>@product.Brand.Name</td>
				<td>@product.Category.Name</td>
			</tr>
		}
	</tbody>
</table>
```
#### Details.cshtml

Создайте файл представления `ProductDetails.cshtml` в папке `Views/ProductComments/`. Ниже представлен пример кода для этого представления.

```html
@model YourNamespace.Models.ProductCommentViewModel

<h2>Детали продукта: @Model.Product.Name</h2>

<!-- Информация о продукте -->
<div>
    <h4>Описание:</h4>
    <p>@Model.Product.Description</p>
    <h4>Цена:</h4>
    <p>@Model.Product.Price.ToString("C")</p>
    <h4>Бренд:</h4>
    <p>@Model.Product.Brand.Name</p>
    <h4>Категория:</h4>
    <p>@Model.Product.Category.Name</p>
</div>

<!-- Форма для добавления нового комментария -->
<h3>Добавить комментарий</h3>
<form asp-action="AddComment" method="post">
    <input type="hidden" asp-for="Product.Id" />
    <div class="form-group">
        <label asp-for="NewComment.Content"></label>
        <textarea asp-for="NewComment.Content" class="form-control" required></textarea>
    </div>
    <div class="form-group">
        <label asp-for="NewComment.UserName"></label>
        <input asp-for="NewComment.UserName" class="form-control" required />
    </div>
    <button type="submit" class="btn btn-primary">Добавить комментарий</button>
</form>

<!-- Список комментариев -->
<h3>Комментарии</h3>
<table class="table">
    <thead>
        <tr>
            <th>Пользователь</th>
            <th>Комментарий</th>
            <th>Дата создания</th>
        </tr>
    </thead>
    <tbody>
        @if (Model.Comments != null && Model.Comments.Any())
        {
            foreach (var comment in Model.Comments)
            {
                <tr>
                    <td>@comment.UserName</td>
                    <td>@comment.Content</td>
                    <td>@comment.CreatedAt.ToString("g")</td>
                </tr>
            }
        }
        else
        {
            <tr>
                <td colspan="3">Нет комментариев.</td>
            </tr>
        }


    </tbody>
</table>

<!-- Ссылка на страницу возврата к списку товаров -->
<a asp-action="Index" asp-controller="Products" class="btn btn-secondary">Вернуться к списку товаров</a>
```
#### Create.cshtml (создание комментария)

```html
@model YourNamespace.Models.Comment

<h2>Добавить комментарий</h2>
<form asp-action="Create">
    <input type="hidden" asp-for="ProductId" value="@ViewBag.ProductId" />
    <div>
        <label asp-for="UserName"></label>
        <input asp-for="UserName" />
    </div>
    <div>
        <label asp-for="Content"></label>
        <textarea asp-for="Content"></textarea>
    </div>
    <button type="submit">Сохранить</button>
</form>
```

```html
@model YourNamespace.ViewModels.ProductCommentViewModel

<h2>@Model.Product.Name</h2>
<p>@Model.Product.Description</p>

<h3>Комментарии</h3>

@foreach (var comment in Model.Comments)
{
    <div>
        <strong>@comment.UserName</strong>
        <p>@comment.Content</p>
        <small>@comment.CreatedAt.ToString("g")</small>
    </div>
}

<h3>Добавить комментарий</h3>
<form asp-action="AddComment">
    <input type="hidden" asp-for="Product.Id" />
    
    <div>
        <label asp-for="NewComment.UserName"></label>
        <input asp-for="NewComment.UserName" />
    </div>
    
    <div>
        <label asp-for="NewComment.Content"></label>
        <textarea asp-for="NewComment.Content"></textarea>
    </div>
    
    <button type="submit">Отправить</button>
</form>

```
### Шаг 9. Внедрение DI

Чтобы внедрить Dependency Injection (DI) в CRUD-контроллер для работы с товарами, создадим интерфейс `IProductService` и его реализацию `ProductService`, которые будут использоваться для взаимодействия с базой данных. Затем зарегистрируем этот сервис в DI-контейнере и внедрим его в контроллер.

#### Шаг 1: Создание интерфейса и реализации `IProductService`

Определим интерфейс `IProductService` с CRUD-операциями для товаров.

```cs
using System.Collections.Generic;
using System.Threading.Tasks;

public interface IProductService
{
    Task<IEnumerable<Product>> GetAllProductsAsync();
    Task<Product> GetProductByIdAsync(int id);
    Task AddProductAsync(Product product);
    Task UpdateProductAsync(Product product);
    Task DeleteProductAsync(int id);
}

```

Реализация `ProductService` будет использовать `DbContext` для взаимодействия с базой данных.

```cs
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;

public class ProductService : IProductService
{
    private readonly ShopDBContext _context;

    public ProductService(ShopDBContext context)
    {
        _context = context;
    }

    public async Task<IEnumerable<Product>> GetAllProductsAsync()
    {
        return await _context.Products.ToListAsync();
    }

    public async Task<Product> GetProductByIdAsync(int id)
    {
        return await _context.Products.FindAsync(id);
    }

    public async Task AddProductAsync(Product product)
    {
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
    }

    public async Task UpdateProductAsync(Product product)
    {
        _context.Products.Update(product);
        await _context.SaveChangesAsync();
    }

    public async Task DeleteProductAsync(int id)
    {
        var product = await _context.Products.FindAsync(id);
        if (product != null)
        {
            _context.Products.Remove(product);
            await _context.SaveChangesAsync();
        }
    }
}

```

#### Шаг 2: Регистрация `ProductService` и `ShopDBContext` в DI-контейнере

В `Program.cs` добавляем регистрацию `ShopDBContext` и `ProductService` в DI-контейнере.

```cs
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Регистрация контекста базы данных
builder.Services.AddDbContext<ShopDBContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("MyConnString")));

// Регистрация сервиса для работы с товарами
builder.Services.AddScoped<IProductService, ProductService>();

var app = builder.Build();

// Настройка HTTP-конвейера
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
}

app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();

```

#### Шаг 3: Использование сервисов в контроллере

Теперь создадим контроллер `ProductController` и внедрим `IProductService` через конструктор.

```cs
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.Threading.Tasks;

public class ProductController : Controller
{
    private readonly IProductService _productService;

    public ProductController(IProductService productService)
    {
        _productService = productService;
    }

    [HttpGet]
    public async Task<IActionResult> Index()
    {
        var products = await _productService.GetAllProductsAsync();
        return View(products);
    }

    [HttpGet]
    public async Task<IActionResult> Details(int id)
    {
        var product = await _productService.GetProductByIdAsync(id);
        if (product == null) return NotFound();
        return View(product);
    }

    [HttpGet]
    public IActionResult Create()
    {
        return View();
    }

    [HttpPost]
    public async Task<IActionResult> Create(Product product)
    {
        if (ModelState.IsValid)
        {
            await _productService.AddProductAsync(product);
            return RedirectToAction(nameof(Index));
        }
        return View(product);
    }

    [HttpGet]
    public async Task<IActionResult> Edit(int id)
    {
        var product = await _productService.GetProductByIdAsync(id);
        if (product == null) return NotFound();
        return View(product);
    }

    [HttpPost]
    public async Task<IActionResult> Edit(Product product)
    {
        if (ModelState.IsValid)
        {
            await _productService.UpdateProductAsync(product);
            return RedirectToAction(nameof(Index));
        }
        return View(product);
    }

    [HttpPost]
    public async Task<IActionResult> Delete(int id)
    {
        await _productService.DeleteProductAsync(id);
        return RedirectToAction(nameof(Index));
    }
}

```




### Шаг 10. Авторизация

#### 1. Добавьте поддержку Identity
Вам потребуется добавить `Identity` в ваш проект. Установите пакет NuGet:

```bash
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore

```

---

#### 2. Создайте пользовательскую модель

Добавьте класс `ApplicationUser`, который будет расширять стандартный `IdentityUser`:

```cs
using Microsoft.AspNetCore.Identity;

namespace dotNetShop.Models
{
    public class ApplicationUser : IdentityUser
    {
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public DateTime DateOfBirth { get; set; }
        public string ProfilePictureUrl { get; set; }
        public string Bio { get; set; }
        public bool IsEmailConfirmed { get; set; }
        public DateTime? LastLoginDate { get; set; }
        public DateTime? AccountCreatedDate { get; set; }
        public string ShippingAddress { get; set; }
        public ICollection<Order> Orders { get; set; }
        public bool IsDeleted { get; set; }

    }
}

```

---

#### 3. Добавьте Identity в контекст базы данных

Обновите `ShopDBContext`, чтобы он наследовался от `IdentityDbContext`:

```cs
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;

namespace dotNetShop.Data
{
    public class ShopDBContext : IdentityDbContext<ApplicationUser>
    {
        public DbSet<Category> Categories { get; set; }
        public DbSet<Product> Products { get; set; }
        public DbSet<Brand> Brands { get; set; }
        public DbSet<Color> Colors { get; set; }
        public DbSet<ProductImage> ProductImages { get; set; }
        public DbSet<Comment> Comments { get; set; }

        public ShopDBContext() { }
        public ShopDBContext(DbContextOptions<ShopDBContext> options) : base(options) { }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder); // Не забудьте вызвать базовую реализацию

            modelBuilder.Entity<Product>()
                .HasOne(p => p.Category)
                .WithMany(p => p.Products)
                .HasForeignKey(p => p.CategoryId)
                .OnDelete(DeleteBehavior.Cascade);

            modelBuilder.Entity<Product>()
                .HasOne(p => p.Brand)
                .WithMany(b => b.Products)
                .HasForeignKey(p => p.BrandId)
                .OnDelete(DeleteBehavior.Restrict);

            modelBuilder.Entity<Product>()
                .HasMany(p => p.AvailableColors)
                .WithMany(c => c.Products)
                .UsingEntity(j => j.ToTable("ProductColors"));

            modelBuilder.Entity<ProductImage>()
                .HasOne(pi => pi.Product)
                .WithMany(p => p.Images)
                .HasForeignKey(pi => pi.ProductId)
                .OnDelete(DeleteBehavior.Cascade);

            modelBuilder.Entity<Comment>()
                .HasOne(c => c.Product)
                .WithMany(p => p.Comments)
                .HasForeignKey(c => c.ProductId)
                .OnDelete(DeleteBehavior.Cascade);
        }
    }
}

```

---

#### 4. Настройте сервисы в `Program.cs`

Обновите регистрацию сервисов:

```cs
builder.Services.AddDbContext<ShopDBContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddIdentity<ApplicationUser, IdentityRole>()
    .AddEntityFrameworkStores<ShopDBContext>()
    .AddDefaultTokenProviders();

builder.Services.ConfigureApplicationCookie(options =>
{
    options.LoginPath = "/Account/Login"; // Укажите путь к странице входа
    options.AccessDeniedPath = "/Account/AccessDenied"; // Страница отказа в доступе
});

```

Добавьте `Authentication` и `Authorization` в конвейер обработки запросов:

```csharp

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

---

#### 5. Добавьте миграции и обновите базу данных

Добавьте миграцию и примените её к базе данных:

```bash
dotnet ef migrations add AddIdentityToShopDB
dotnet ef database update
```

---

#### 6. Создайте интерфейсы для авторизации

Создайте контроллер для управления пользователями и аутентификацией:

#### Пример: Контроллер `AccountController`

```cs
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;

namespace dotNetShop.Controllers
{
    public class AccountController : Controller
    {
        private readonly UserManager<ApplicationUser> _userManager;
        private readonly SignInManager<ApplicationUser> _signInManager;

        public AccountController(UserManager<ApplicationUser> userManager, SignInManager<ApplicationUser> signInManager)
        {
            _userManager = userManager;
            _signInManager = signInManager;
        }

        public IActionResult Login() => View();

        [HttpPost]
        public async Task<IActionResult> Login(string email, string password)
        {
            var result = await _signInManager.PasswordSignInAsync(email, password, isPersistent: false, lockoutOnFailure: false);
            if (result.Succeeded)
            {
                return RedirectToAction("Index", "Home");
            }

            ModelState.AddModelError("", "Invalid login attempt");
            return View();
        }

        public IActionResult Register() => View();

        [HttpPost]
        public async Task<IActionResult> Register(string email, string password)
        {
            var user = new ApplicationUser { UserName = email, Email = email };
            var result = await _userManager.CreateAsync(user, password);
            if (result.Succeeded)
            {
                await _signInManager.SignInAsync(user, isPersistent: false);
                return RedirectToAction("Index", "Home");
            }

            foreach (var error in result.Errors)
            {
                ModelState.AddModelError("", error.Description);
            }
            return View();
        }

        public async Task<IActionResult> Logout()
        {
            await _signInManager.SignOutAsync();
            return RedirectToAction("Index", "Home");
        }
    }
}

```
#### 1. Представление для логина (`Login.cshtml`)

##### Код:

```html
@model LoginViewModel

@{
    ViewData["Title"] = "Login";
}

<h2>Login</h2>

<div class="row">
    <div class="col-md-6">
        <form asp-action="Login" method="post">
            <div class="form-group">
                <label asp-for="Email"></label>
                <input asp-for="Email" class="form-control" />
                <span asp-validation-for="Email" class="text-danger"></span>
            </div>
            <div class="form-group">
                <label asp-for="Password"></label>
                <input asp-for="Password" type="password" class="form-control" />
                <span asp-validation-for="Password" class="text-danger"></span>
            </div>
            <button type="submit" class="btn btn-primary">Login</button>
        </form>
    </div>
</div>

```

---

#### 2. Представление для регистрации (`Register.cshtml`)

##### Код:

```html
@model RegisterViewModel

@{
    ViewData["Title"] = "Register";
}

<h2>Register</h2>

<div class="row">
    <div class="col-md-6">
        <form asp-action="Register" method="post">
            <div class="form-group">
                <label asp-for="Email"></label>
                <input asp-for="Email" class="form-control" />
                <span asp-validation-for="Email" class="text-danger"></span>
            </div>
            <div class="form-group">
                <label asp-for="Password"></label>
                <input asp-for="Password" type="password" class="form-control" />
                <span asp-validation-for="Password" class="text-danger"></span>
            </div>
            <div class="form-group">
                <label asp-for="ConfirmPassword"></label>
                <input asp-for="ConfirmPassword" type="password" class="form-control" />
                <span asp-validation-for="ConfirmPassword" class="text-danger"></span>
            </div>
            <button type="submit" class="btn btn-primary">Register</button>
        </form>
    </div>
</div>

```

---

#### 3. ViewModel для логина и регистрации

Добавьте классы `LoginViewModel` и `RegisterViewModel`:

##### `LoginViewModel`

```cs
public class LoginViewModel
{
    [Required(ErrorMessage = "Email is required")]
    [EmailAddress(ErrorMessage = "Invalid Email Address")]
    public string Email { get; set; }

    [Required(ErrorMessage = "Password is required")]
    [DataType(DataType.Password)]
    public string Password { get; set; }
}

```

##### `RegisterViewModel`

```cs
public class RegisterViewModel
{
    [Required(ErrorMessage = "Email is required")]
    [EmailAddress(ErrorMessage = "Invalid Email Address")]
    public string Email { get; set; }

    [Required(ErrorMessage = "Password is required")]
    [DataType(DataType.Password)]
    [MinLength(6, ErrorMessage = "Password must be at least 6 characters long.")]
    public string Password { get; set; }

    [Required(ErrorMessage = "Confirm Password is required")]
    [Compare("Password", ErrorMessage = "Passwords do not match.")]
    [DataType(DataType.Password)]
    public string ConfirmPassword { get; set; }
}

```

---

#### 4. Добавление валидации

Убедитесь, что в вашем `_Layout.cshtml` подключены необходимые скрипты для клиентской валидации:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-validate/1.19.3/jquery.validate.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-validation-unobtrusive/3.2.12/jquery.validate.unobtrusive.min.js"></script>

```
---

#### 5. Используйте атрибуты для защиты данных

Добавьте атрибуты `[Authorize]` для защиты определённых действий или контроллеров:

```cs
[Authorize]
public class SecureController : Controller
{
    public IActionResult Index()
    {
        return View();
    }
}

```




### Шаг 11. Развёртывание в Linux Debian
#### 1. Установите .NET SDK и Runtime на Debian

##### Подготовка
```sh
sudo apt install wget -y
```

```sh
wget https://packages.microsoft.com/config/debian/12/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
```
```sh
sudo dpkg -i packages-microsoft-prod.deb
```
```sh
rm packages-microsoft-prod.deb
```
##### Установка пакета SDK

```sh
sudo apt-get update && sudo apt-get install -y dotnet-sdk-8.0
```

##### Установка среды выполнения

```sh
sudo apt-get update && sudo apt-get install -y aspnetcore-runtime-8.0
```

##### Обновление .NET с помощью APT

```sh
sudo apt-get update
sudo apt-get upgrade
```



#### 2. Установка и настройка базы данных MySql

##### Install MySQL

```sh
sudo apt update
```

```sh
wget https://dev.mysql.com/get/mysql-apt-config_0.8.30-1_all.deb
```
```sh
sudo dpkg -i mysql-apt-config_0.8.30-1_all.deb
```
```sh
sudo apt install mysql-server -y
```
```sh
sudo dpkg-reconfigure mysql-apt-config
```
```sh
sudo apt update
```
##### Manage the MySQL System Service

```sh
sudo systemctl enable mysql
```

```sh
sudo systemctl start mysql
```

```sh
sudo systemctl status mysql
```

```sh
sudo journalctl -u mysql
```
##### Secure the MySQL Database Server

```sh
sudo mysql_secure_installation
```

##### Access MySQL

```sh
sudo mysql -u root -p
```

```sql
CREATE DATABASE <database_name>;
```

```sql
SHOW DATABASES;
```

Create a new sample database user `db_user` with a strong password. Replace `Strong@@password123` with your desired password depending on the database server policy.
```sh
CREATE USER 'admin'@'localhost' IDENTIFIED BY 'Strong@@password123';
```

Grant the `db_user` full privileges to the `shop` database.

```sh
GRANT ALL PRIVILEGES ON database_name.* TO 'admin'@'localhost';
```

```sql
FLUSH PRIVILEGES;
```

```sh
EXIT
```
##### Error "Access denied for user 'root'@'localhost' (using password: YES)"

```sh
sudo nano /etc/mysql/my.cnf
```

```init
[mysqld] 
skip-grant-tables 
skip-networking
```

```sh
sudo systemctl restart mysql
```

```sh
mysql
```

```sql
FLUSH PRIVILEGES;
```

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '<password>';
```


##### Прочие команды mysql cli
Для того чтобы посмотреть структуру таблицы в базе данных _MySQL_, можно воспользоваться командой `DESCRIBE` или `SHOW COLUMNS`.

```sql
DESCRIBE table_name;
```
ИЛИ

```mysql
SHOW COLUMNS FROM table_name;
```

Connection string
```mysql
Server=myServerAddress;Database=myDataBase;Uid=myUsername;Pwd=myPassword;
```


#### 3. Опубликуйте ваше ASP.NET Core приложение

После того как приложение разработано и готово к развертыванию, создайте публикацию приложения:

```sh
cd /path/to/your/project 
dotnet publish -c Release -o /var/www/yourapp
```

##### 3.1. Установите и настройте Nginx

Чтобы настроить Nginx как обратный прокси, выполните следующие шаги:

```sh
sudo apt-get install nginx
```

После установки Nginx, настройте его для проксирования запросов к вашему приложению ASP.NET Core.

Откройте конфигурационный файл Nginx:

```sh
sudo nano /etc/nginx/sites-available/yourapp
```

Добавьте следующее содержимое, чтобы настроить проксирование:

```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:5000;  # Порт, на котором будет работать ваше приложение
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

```

Сохраните файл и создайте символическую ссылку, чтобы активировать конфигурацию:

```sh
sudo ln -s /etc/nginx/sites-available/yourapp /etc/nginx/sites-enabled/
```

##### 3.2. Настройте Systemd для вашего приложения

Для удобства использования создайте unit-файл для вашего приложения в `systemd`, чтобы автоматически запускать его как службу.

Создайте файл `/etc/systemd/system/yourapp.service`:

```sh
sudo nano /etc/systemd/system/yourapp.service
```

Добавьте следующее содержимое:

```ini
[Unit]
Description=ASP.NET Core Application

[Service]
WorkingDirectory=/var/www/yourapp
ExecStart=/usr/bin/dotnet /var/www/yourapp/yourapp.dll
Restart=always
# Убедитесь, что сервер будет работать под нужным пользователем
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false
# Важный момент для загрузки переменных среды
Environment=PATH=/usr/bin:$PATH
Environment=DOTNET_ROOT=/usr/share/dotnet
ExecReload=/bin/kill -s HUP $MAINPID

[Install]
WantedBy=multi-user.target

```

Затем перезапустите systemd и запустите службу:

```sh
sudo systemctl daemon-reload 
sudo systemctl start yourapp 
sudo systemctl enable yourapp
```

##### 3.3. Перезапустите Nginx

После настройки Nginx перезапустите его, чтобы применить конфигурации:

```sh
sudo systemctl restart nginx
```

##### 3.4. Откройте необходимые порты в firewall

Если вы используете firewall, откройте порты для Nginx (80) и вашего приложения (например, 5000):

```sh
sudo ufw allow 'Nginx Full'
sudo ufw allow 5000
sudo ufw enable
```

Теперь ваше приложение ASP.NET Core будет доступно через Nginx на порту 80.

##### Проверка

Перейдите по адресу вашего сервера или домену, и ваше приложение должно быть доступно.



### Шаг 12. Отдача статических файлов

Для настройки отдачи статических файлов в ASP.NET и Nginx необходимо настроить серверное приложение и веб-сервер, чтобы правильно обрабатывать запросы к статическим ресурсам, таким как CSS, JavaScript, изображения и другие файлы.

#### 1. Настройка в ASP.NET

В ASP.NET Core статические файлы обрабатываются с помощью middleware `StaticFiles`.

##### Пример конфигурации:

```cs
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Добавляем поддержку статических файлов
app.UseStaticFiles();

// Запускаем приложение
app.Run();

```

#### Статические файлы:

- По умолчанию файлы размещаются в папке `wwwroot`.
- Убедитесь, что файлы доступны, добавив их в папку `wwwroot` и проверив права доступа.

#### Пример структуры проекта:


```
MyApp/
├── wwwroot/
│   ├── css/
│   ├── js/
│   ├── images/

```

Статические файлы из `wwwroot` доступны по URL без указания папки `wwwroot`. Например:

- Файл: `wwwroot/css/styles.css`
- URL: `/css/styles.css`

#### Дополнительные настройки:

Можно настроить поведение для отдачи статических файлов, например, указать кэширование или ограничения:

```cs
app.UseStaticFiles(new StaticFileOptions
{
    OnPrepareResponse = ctx =>
    {
        ctx.Context.Response.Headers["Cache-Control"] = "public,max-age=3600";
    }
});

```

---

#### 2. Настройка в Nginx

Nginx используется как прокси-сервер или для отдачи статических файлов напрямую.

#### Пример конфигурации:

Если Nginx отдает статические файлы напрямую:


```
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:5000; # Проксирование к ASP.NET приложению
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Настройка для отдачи статических файлов
    location /css/ {
        root /path/to/your/app/wwwroot/;
    }

    location /js/ {
        root /path/to/your/app/wwwroot/;
    }

    location /images/ {
        root /path/to/your/app/wwwroot/;
    }
}

```

#### Объяснение:

1. **location /:** Все запросы проксируются к ASP.NET приложению.
2. **location /css/:** Запросы к `/css/` обслуживаются напрямую из папки `wwwroot/css/` без участия ASP.NET.

#### Ключевые настройки:

- **root:** Указывает корневую папку для статических файлов.
- **proxy_pass:** Адрес ASP.NET приложения.

---

#### 3. Комбинированный подход

Nginx может отдавать только статические файлы, а остальные запросы проксировать к ASP.NET приложению. Это уменьшает нагрузку на приложение.

#### Пример:
```nginx
server {
    listen 80;
    server_name yourdomain.com;

    # Статические файлы
    location /static/ {
        alias /path/to/your/app/wwwroot/;
    }

    # Проксирование ASP.NET
    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

```

Запросы:

- `/static/css/styles.css` → отдача из `/path/to/your/app/wwwroot/css/styles.css`.
- Остальные запросы передаются на ASP.NET.

### Шаг 13. HTTPS

Для настройки HTTPS с использованием Nginx нужно установить SSL-сертификат, настроить сервер и проксировать запросы на ваше ASP.NET приложение. Вот пошаговое руководство:

---

#### 1. Получение SSL-сертификата

##### Вариант 1: Let's Encrypt (Бесплатный SSL)

1. Установите **Certbot**:
   
```cs
sudo apt update sudo apt install certbot python3-certbot-nginx
```

2. Запустите Certbot для автоматической настройки SSL:

```cs
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

3. Certbot автоматически добавит конфигурацию для HTTPS в ваш файл Nginx.

4. Проверьте и перезапустите Nginx:

```cs
sudo nginx -t sudo systemctl restart nginx
```

5. Настройте автоматическое обновление сертификатов:

```cs
sudo crontab -e
```

Добавьте строку:

```cs
0 0 * * * certbot renew --quiet
```


---

##### 2. Настройка Nginx для HTTPS

Пример конфигурации Nginx с HTTPS:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    # Перенаправление с HTTP на HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com www.yourdomain.com;

    # Пути к SSL-сертификатам
    ssl_certificate /etc/nginx/ssl/certificate.crt;
    ssl_certificate_key /etc/nginx/ssl/private.key;

    # SSL параметры
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers HIGH:!aNULL:!MD5;

    # Локации для проксирования и статических файлов
    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /static/ {
        alias /path/to/your/app/wwwroot/;
    }
}
```

---

##### 3. Улучшения безопасности

Добавьте заголовки для защиты HTTPS соединения:

```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header X-Content-Type-Options nosniff;
add_header X-Frame-Options DENY;
add_header X-XSS-Protection "1; mode=block";
```

---

##### 4. Проверка конфигурации

1. Убедитесь, что конфигурация Nginx корректна:

```cs
sudo nginx -t
```

2. Перезапустите Nginx:

```cs
sudo systemctl restart nginx
```


---

##### 5. Проверка HTTPS

1. Откройте ваш сайт по адресу `https://yourdomain.com`.
2. Используйте SSL Labs для проверки качества настройки HTTPS.