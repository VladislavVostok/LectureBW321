
Для работы с JWT-аутентификацией.
```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

Для работы с Entity Framework Core (ORM для взаимодействия с базой данных).
```bash
dotnet add package Microsoft.EntityFrameworkCore
```

Провайдер базы данных (например, SQL Server). Замените на провайдер, соответствующий вашей базе данных.
```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer

```

или

```bash
dotnet add package Pomelo.EntityFrameworkCore.MySql
```

Для выполнения миграций и управления схемой базы данных.
```bash
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

Для работы с Identity (управление пользователями, паролями, ролями и аутентификацией).
```bash
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
```

Для работы с JWT токенами, их генерацией и проверкой.
```bash
dotnet add package System.IdentityModel.Tokens.Jwt
```

Для преобразования между DTO и сущностями.
```bash
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
```


Модели Пользователя и его профиль

```cs
using Microsoft.AspNetCore.Identity;
using System;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

public class User : IdentityUser
{
    [Required]
    [MaxLength(100)]
    public override string UserName { get; set; }

    [Required]
    [MaxLength(100)]
    public override string Email { get; set; }

    [MaxLength(100)]
    public string FullName { get; set; }

    [MaxLength(100)]
    public string OTP { get; set; }

    [MaxLength(1000)]
    public string RefreshToken { get; set; }

    public virtual Profile Profile { get; set; }

    public override string ToString()
    {
        return Email;
    }

    public void InitializeDefaults()
    {
        var emailUsername = Email.Split('@')[0];

        if (string.IsNullOrEmpty(FullName))
            FullName = emailUsername;

        if (string.IsNullOrEmpty(UserName))
            UserName = emailUsername;
    }
}

public class Profile
{
    [Key]
    public int Id { get; set; }

    [Required]
    public string UserId { get; set; }

    [ForeignKey("UserId")]
    public virtual User User { get; set; }

    [MaxLength(100)]
    public string FullName { get; set; }

    [MaxLength(100)]
    public string Country { get; set; }

    public string About { get; set; }

    [MaxLength(255)]
    public string Image { get; set; } = "default-user.jpg";

    [Required]
    public DateTime DateCreated { get; set; } = DateTime.UtcNow;

    public override string ToString()
    {
        return !string.IsNullOrEmpty(FullName) ? FullName : User.FullName;
    }

    public void InitializeDefaults()
    {
        if (string.IsNullOrEmpty(FullName))
            FullName = User?.UserName;
    }
}

public class ApplicationDbContext : IdentityDbContext<User>
{
    public DbSet<Profile> Profiles { get; set; }

    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);

        builder.Entity<User>()
            .HasOne(u => u.Profile)
            .WithOne(p => p.User)
            .HasForeignKey<Profile>(p => p.UserId)
            .OnDelete(DeleteBehavior.Cascade);
    }
}

public class ProfileService
{
    private readonly ApplicationDbContext _context;

    public ProfileService(ApplicationDbContext context)
    {
        _context = context;
    }

    public void CreateProfileForUser(User user)
    {
        var profile = new Profile
        {
            UserId = user.Id,
            FullName = user.FullName,
            Image = "default-user.jpg",
            DateCreated = DateTime.UtcNow
        };

        profile.InitializeDefaults();

        _context.Profiles.Add(profile);
        _context.SaveChanges();
    }
}

```


Создание сервиса для работы с токеном

### Описание методов:

1. **`GenerateAccessToken(User user)`**:
    - Генерирует JWT токен на основе данных пользователя.
    - Обычно содержит информацию о `Claims` (e.g., email, роли, id).

2. **`GenerateRefreshToken()`**:
    - Создает случайный строковый токен для долгосрочной аутентификации (обычно хранится в базе данных).

3. **`GetPrincipalFromExpiredToken(string token)`**:
    - Извлекает информацию из истекшего JWT токена (полезно для валидации и обновления токена).

```cs
public interface ITokenService
{
    string GenerateAccessToken(User user);
    string GenerateRefreshToken();
    ClaimsPrincipal GetPrincipalFromExpiredToken(string token);
}

```


```cs
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Security.Cryptography;
using Microsoft.IdentityModel.Tokens;

public class TokenService : ITokenService
{
    private readonly IConfiguration _configuration;

    public TokenService(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public string GenerateAccessToken(User user)
    {
        var claims = new List<Claim>
        {
            new Claim(JwtRegisteredClaimNames.Sub, user.Id.ToString()),
            new Claim(JwtRegisteredClaimNames.Email, user.Email),
            new Claim(ClaimTypes.Name, user.FullName),
            new Claim(ClaimTypes.Role, "User") // Можно добавить роли из базы данных
        };

        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_configuration["Jwt:Key"]));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: _configuration["Jwt:Issuer"],
            audience: _configuration["Jwt:Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(30), // Время жизни токена
            signingCredentials: creds);

        return new JwtSecurityTokenHandler().WriteToken(token);
    }

    public string GenerateRefreshToken()
    {
        var randomNumber = new byte[32];
        using (var rng = RandomNumberGenerator.Create())
        {
            rng.GetBytes(randomNumber);
            return Convert.ToBase64String(randomNumber);
        }
    }

    public ClaimsPrincipal GetPrincipalFromExpiredToken(string token)
    {
        var tokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_configuration["Jwt:Key"])),
            ValidateIssuer = false,
            ValidateAudience = false,
            ValidateLifetime = false // Мы не проверяем время жизни, потому что токен истек
        };

        var tokenHandler = new JwtSecurityTokenHandler();
        var principal = tokenHandler.ValidateToken(token, tokenValidationParameters, out SecurityToken securityToken);

        if (securityToken is not JwtSecurityToken jwtSecurityToken ||
            !jwtSecurityToken.Header.Alg.Equals(SecurityAlgorithms.HmacSha256, StringComparison.InvariantCultureIgnoreCase))
        {
            throw new SecurityTokenException("Invalid token");
        }

        return principal;
    }
}

```

В файле `appsettings.json`:

```json
"Jwt": {
  "Key": "YourSuperSecretKeyHere",
  "Issuer": "YourApp",
  "Audience": "YourAppAudience"
}

```

Авторизация по токену

```cs
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly UserManager<User> _userManager;
    private readonly ITokenService _tokenService;

    public AuthController(UserManager<User> userManager, ITokenService tokenService)
    {
        _userManager = userManager;
        _tokenService = tokenService;
    }

    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginDto loginDto)
    {
        var user = await _userManager.FindByEmailAsync(loginDto.Email);
        if (user != null && await _userManager.CheckPasswordAsync(user, loginDto.Password))
        {
            var accessToken = _tokenService.GenerateAccessToken(user);
            var refreshToken = _tokenService.GenerateRefreshToken();

            // Сохраните refreshToken в базе данных или другой системе
            user.RefreshToken = refreshToken;
            await _userManager.UpdateAsync(user);

            return Ok(new
            {
                AccessToken = accessToken,
                RefreshToken = refreshToken
            });
        }

        return Unauthorized(new { message = "Invalid credentials" });
    }
}


```

Регистрация

```cs
[HttpPost("register")]
[AllowAnonymous]
public async Task<IActionResult> Register([FromBody] RegisterDto registerDto)
{
    var user = new User
    {
        Email = registerDto.Email,
        UserName = registerDto.UserName,
        FullName = registerDto.FullName
    };

    var result = await _userManager.CreateAsync(user, registerDto.Password);

    if (result.Succeeded)
    {
        return Ok(new { message = "User registered successfully" });
    }

    return BadRequest(result.Errors);
}

```


**OTP (One-Time Password)** — это **одноразовый пароль**, который используется для подтверждения или аутентификации пользователя. Он действителен только в течение ограниченного времени или для одной сессии и после этого становится недействительным.
### Где используется OTP:

1. **Двухфакторная аутентификация (2FA):**
    - Например, при входе в аккаунт вам отправляется OTP по SMS, email или через приложение (Google Authenticator, Authy).

2. **Подтверждение транзакций:**
    - OTP используется для подтверждения банковских операций, покупок и других транзакций.

3. **Сброс пароля:**
    - Одноразовый код отправляется пользователю для подтверждения права на сброс пароля.



### Виды OTP

1. **SMS или Email OTP:**
    - Отправляются на телефон или email пользователя.
    - Пример: "Ваш код подтверждения: 123456".

2. **Генерация в приложении:**
    - Используется приложение вроде Google Authenticator, которое генерирует временные OTP-коды.

3. **Токены:**
    - Специальные устройства (например, банковские токены), генерирующие одноразовые пароли.

4. **Алгоритмические OTP:**
    - Пароль генерируется на основе определенного алгоритма (например, временного или событийного).



### Алгоритмы генерации OTP

1. **HOTP (HMAC-Based One-Time Password):**
    - Основан на событии: пароль изменяется при выполнении определенного действия (например, входе).

2. **TOTP (Time-Based One-Time Password):**
    - Основан на времени: пароль действителен только в течение заданного временного окна (обычно 30 или 60 секунд).


Изменение пароля по Email

```cs
[HttpGet("password-reset-email/{email}")]
[AllowAnonymous]
public async Task<IActionResult> SendPasswordResetEmail(string email)
{
    var user = await _userManager.FindByEmailAsync(email);
    if (user != null)
    {
        user.OTP = GenerateRandomOtp(7);
        user.RefreshToken = Guid.NewGuid().ToString(); // Simplified refresh token generation
        await _userManager.UpdateAsync(user);

        var link = $"http://localhost:5173/create-new-password?otp={user.OTP}&uuid={user.Id}&refresh_token={user.RefreshToken}";
        // TODO: Configure Email Sending
        Console.WriteLine($"Reset Link: {link}");
    }

    return Ok(new { message = "If the email exists, a reset link was sent." });
}

```

```cs
private string GenerateRandomOtp(int length)
{
    var random = new Random();
    return new string(Enumerable.Range(0, length).Select(_ => random.Next(0, 10).ToString()[0]).ToArray());
}

```


Изменение пароля по OTP

```cs
[HttpPost("password-reset")]
[AllowAnonymous]
public async Task<IActionResult> ResetPassword([FromBody] PasswordResetDto resetDto)
{
    var user = await _userManager.FindByIdAsync(resetDto.Uuid);
    if (user != null && user.OTP == resetDto.OTP)
    {
        var result = await _userManager.RemovePasswordAsync(user);
        if (result.Succeeded)
        {
            await _userManager.AddPasswordAsync(user, resetDto.NewPassword);
            user.OTP = null; // Clear the OTP
            await _userManager.UpdateAsync(user);
            return Ok(new { message = "Password changed successfully" });
        }
    }

    return NotFound(new { message = "Invalid OTP or user not found" });
}

```

Изменение пароля (при условии что пользователь Авторизован)

```cs
[HttpPost("change-password")]
[Authorize]
public async Task<IActionResult> ChangePassword([FromBody] ChangePasswordDto changeDto)
{
    var user = await _userManager.FindByIdAsync(changeDto.UserId);
    if (user != null)
    {
        var result = await _userManager.ChangePasswordAsync(user, changeDto.OldPassword, changeDto.NewPassword);
        if (result.Succeeded)
        {
            return Ok(new { message = "Password changed successfully", icon = "success" });
        }

        return BadRequest(new { message = "Old password is incorrect", icon = "warning" });
    }

    return NotFound(new { message = "User does not exist", icon = "error" });
}
```


Классы DTO

**DTO (Data Transfer Object)** — это объект, который используется для передачи данных между слоями приложения или между приложениями. DTO создаются, чтобы:

1. **Изолировать слои приложения**: DTO отделяет бизнес-логику от внешнего интерфейса, предоставляя только те данные, которые нужны для конкретной операции.
    
2. **Оптимизировать передачу данных**: В отличие от полноценных моделей, DTO содержат только необходимые поля, что уменьшает объем передаваемой информации и упрощает сериализацию/десериализацию.
    
3. **Защитить внутренние данные**: DTO позволяют не раскрывать детали внутренней реализации (например, бизнес-логику или структуру базы данных) внешним системам.

```cs
public class LoginDto
{
    public string Email { get; set; }
    public string Password { get; set; }
}

public class RegisterDto
{
    public string Email { get; set; }
    public string UserName { get; set; }
    public string FullName { get; set; }
    public string Password { get; set; }
}

public class PasswordResetDto
{
    public string Uuid { get; set; }
    public string OTP { get; set; }
    public string NewPassword { get; set; }
}

public class ChangePasswordDto
{
    public string UserId { get; set; }
    public string OldPassword { get; set; }
    public string NewPassword { get; set; }
}

```


