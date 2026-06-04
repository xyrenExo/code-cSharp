# Dependency Injection

## Introduction
Dependency Injection (DI) is a design pattern where dependencies are provided to a class rather than created internally. ASP.NET Core has built-in DI.

## Service Lifetimes

| Lifetime | Description | Instance Per |
|----------|-------------|--------------|
| **Singleton** | One instance | Whole application |
| **Scoped** | One instance per request | HTTP request |
| **Transient** | New instance each time | Every injection |

```csharp
// Singleton
builder.Services.AddSingleton<IUserService, UserService>();

// Scoped
builder.Services.AddScoped<IUserService, UserService>();

// Transient
builder.Services.AddTransient<IUserService, UserService>();
```

## Service Registration

```csharp
var builder = WebApplication.CreateBuilder(args);

// Interface + implementation
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddScoped<IUserService, UserService>();

// Concrete type
builder.Services.AddScoped<UserService>();

// Factory
builder.Services.AddScoped<IUserService>(sp =>
{
    var repo = sp.GetRequiredService<IUserRepository>();
    return new UserService(repo, "config");
});

// Instance (singleton)
builder.Services.AddSingleton<IUserService>(new UserService(new UserRepository(), "cfg"));

// Multiple implementations
builder.Services.AddScoped<IDataExporter, CsvExporter>();
builder.Services.AddScoped<IDataExporter, JsonExporter>();
```

## Constructor Injection

```csharp
public class UserService : IUserService
{
    private readonly IUserRepository _repository;
    private readonly ILogger<UserService> _logger;
    private readonly AppDbContext _context;

    public UserService(
        IUserRepository repository,
        ILogger<UserService> logger,
        AppDbContext context)
    {
        _repository = repository;
        _logger = logger;
        _context = context;
    }

    public async Task<User> GetUserAsync(int id)
    {
        _logger.LogInformation("Getting user {Id}", id);
        return await _repository.GetByIdAsync(id);
    }
}
```

## Method Injection

```csharp
[ApiController]
[Route("api/users")]
public class UsersController : ControllerBase
{
    [HttpGet("{id}")]
    public async Task<ActionResult<User>> GetUser(
        int id,
        [FromServices] IUserService userService)  // Method injection
    {
        return await userService.GetUserAsync(id);
    }
}
```

## Keyed Services (.NET 8+)

```csharp
// Registration
builder.Services.AddKeyedScoped<IDataExporter, CsvExporter>("csv");
builder.Services.AddKeyedScoped<IDataExporter, JsonExporter>("json");

// Injection
public class ExportService
{
    public async Task ExportAsync(string format, object data)
    {
        var exporter = format switch
        {
            "csv" => HttpContext.RequestServices.GetRequiredKeyedService<IDataExporter>("csv"),
            "json" => HttpContext.RequestServices.GetRequiredKeyedService<IDataExporter>("json"),
            _ => throw new NotSupportedException()
        };

        await exporter.ExportAsync(data);
    }
}

// Constructor injection with [FromKeyedServices]
public class DataController(
    [FromKeyedServices("csv")] IDataExporter csvExporter)
{
}
```

## Service Locator (Anti-Pattern, Avoid)

```csharp
// BAD - service locator (don't do this)
public class UserController
{
    public async Task<User> GetUser(int id)
    {
        var service = HttpContext.RequestServices
            .GetService<IUserService>();  // Service locator!
        return await service.GetUserAsync(id);
    }
}

// GOOD - constructor injection
public class UserController(IUserService userService)
{
    public async Task<User> GetUser(int id)
        => await userService.GetUserAsync(id);
}
```

## Lifetime Scoping

```csharp
// Create custom scope
using (var scope = serviceProvider.CreateScope())
{
    var scopedService = scope.ServiceProvider
        .GetRequiredService<IScopedService>();

    await scopedService.ProcessAsync();
}
```

## ASP.NET Core Built-in Services

```csharp
var builder = WebApplication.CreateBuilder(args);

// Already registered:
// - ILogger<T>
// - IConfiguration
// - IWebHostEnvironment
// - IHttpContextAccessor
// - IHttpClientFactory

// Example
public class MyService
{
    public MyService(
        ILogger<MyService> logger,
        IConfiguration config,
        IWebHostEnvironment env,
        IHttpContextAccessor httpContext)
    {
    }
}
```

## Testing with DI

```csharp
public class UserServiceTests
{
    [Fact]
    public async Task GetUser_Returns_User()
    {
        // Arrange
        var mockRepo = new Mock<IUserRepository>();
        mockRepo.Setup(r => r.GetByIdAsync(1))
            .ReturnsAsync(new User { Id = 1, Name = "Test" });

        var logger = new Mock<ILogger<UserService>>();

        var service = new UserService(mockRepo.Object, logger.Object);

        // Act
        var user = await service.GetUserAsync(1);

        // Assert
        Assert.Equal("Test", user.Name);
    }
}
```

## Options Pattern

```csharp
// appsettings.json
{
  "AppConfig": {
    "AppName": "MyApp",
    "MaxItems": 100,
    "FeatureFlags": {
      "NewFeature": true
    }
  }
}

// Options class
public class AppOptions
{
    public const string SectionName = "AppConfig";
    public string AppName { get; set; } = string.Empty;
    public int MaxItems { get; set; }
    public FeatureFlagsOptions FeatureFlags { get; set; } = new();
}

public class FeatureFlagsOptions
{
    public bool NewFeature { get; set; }
}

// Registration
builder.Services.Configure<AppOptions>(builder.Configuration.GetSection("AppConfig"));

// Usage
public class MyService
{
    private readonly AppOptions _options;
    public MyService(IOptions<AppOptions> options) => _options = options.Value;
    public MyService(IOptionsSnapshot<AppOptions> options) => _options = options.Value; // Scoped
    public MyService(IOptionsMonitor<AppOptions> options) => _options = options.CurrentValue; // Singleton + reload
}
```
