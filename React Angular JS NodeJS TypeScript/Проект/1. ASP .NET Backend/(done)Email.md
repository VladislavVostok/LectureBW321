## Подключение Email

### 1. Установите необходимые зависимости

В файле проекта (`.csproj`) или через NuGet добавьте библиотеку **MailKit** для отправки email.

```bash
dotnet add package MailKit
```

---

### 2. Настройте конфигурацию для Yandex SMTP

Добавьте секцию для email в `appsettings.json`:


```json
"EmailSettings": {
  "SMTPServer": "smtp.yandex.ru",
  "Port": 465,
  "SenderName": "vladislav",
  "SenderEmail": "vladislav.sage@yandex.ru",
  "Username": "vladislav.sage@yandex.ru",
  "Password": "zcryiolxwhwlpunv",
  "EnableSSL": true
}
```

> **Примечание:** Для Yandex вместо пароля рекомендуется использовать пароль приложения.

---

### 3. Создайте модель для хранения конфигурации

Добавьте класс `EmailSettings.cs` в ваш проект:

```cs
public class EmailSettings
{
    public string SMTPServer { get; set; }
    public int Port { get; set; }
    public string SenderName { get; set; }
    public string SenderEmail { get; set; }
    public string Username { get; set; }
    public string Password { get; set; }
    public bool EnableSSL { get; set; }
}
```

---

### 4. Зарегистрируйте `EmailSettings` в `Program.cs`

Добавьте настройки в контейнер DI:
```cs

builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("EmailSettings"));
builder.Services.AddTransient<IEmailService, EmailService>();

```

---

### 5. Реализуйте интерфейс для отправки писем

Создайте интерфейс `IEmailService.cs`:

```cs
public interface IEmailService
{
    Task SendEmailAsync(string toEmail, string subject, string body);
}
```

---

### 6. Реализуйте сервис для отправки email

Добавьте класс `EmailService.cs`:

```cs
using MailKit.Net.Smtp;
using MimeKit;
using Microsoft.Extensions.Options;

public class EmailService : IEmailService
{
    private readonly EmailSettings _emailSettings;

    public EmailService(IOptions<EmailSettings> emailSettings)
    {
        _emailSettings = emailSettings.Value;
    }

    public async Task SendEmailAsync(string toEmail, string subject, string body)
    {
        var emailMessage = new MimeMessage();

        emailMessage.From.Add(new MailboxAddress(_emailSettings.SenderName, _emailSettings.SenderEmail));
        emailMessage.To.Add(new MailboxAddress("", toEmail));
        emailMessage.Subject = subject;

        var bodyBuilder = new BodyBuilder { HtmlBody = body };
        emailMessage.Body = bodyBuilder.ToMessageBody();

        using var client = new SmtpClient();
        try
        {
            await client.ConnectAsync(_emailSettings.SMTPServer, _emailSettings.Port, _emailSettings.EnableSSL);
            await client.AuthenticateAsync(_emailSettings.Username, _emailSettings.Password);
            await client.SendAsync(emailMessage);
        }
        finally
        {
            await client.DisconnectAsync(true);
        }
    }
}

```

---

### 7. Добавьте тестовый метод в контроллер

Создайте или обновите существующий контроллер, например, `EmailController.cs`:

```cs
using Microsoft.AspNetCore.Mvc;

[HttpGet("password-reset-email/{email}")]
[AllowAnonymous]
public async Task<IActionResult> SendPasswordResetEmail(string email)
{
	var user = await _userManager.FindByEmailAsync(email);
	if (user != null)
	{
		user.OTP = GenerateRandomOtp(7);
		user.RefreshToken = _tokenService.GenerateRefreshToken(); // Simplified refresh token generation
		await _userManager.UpdateAsync(user);

		var link = $"http://localhost:5173/password-reset?otp={user.OTP}&uuid={user.Id}&refresh_token={user.RefreshToken}";
		// TODO: Configure Email Sending

		await _emailService.SendEmailAsync(user.Email, "Восстановление пароля", link);
		return Ok("Email sent successfully");

		Console.WriteLine($"Reset Link: {link}");
	}

	return Ok(new { message = "If the email exists, a reset link was sent." });
}

```

---

### 8. Протестируйте отправку email

1. Запустите приложение.
2. Отправьте POST-запрос через Postman или другое приложение к `/api/email/send` с телом:

```json
{
  "toEmail": "example@domain.com",
  "subject": "Test Email",
  "body": "This is a test email sent via Yandex SMTP in ASP.NET Core."
}

```