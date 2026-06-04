# Minimal APIs

## Introduction (ASP.NET Core 6+)
Minimal APIs provide a simplified approach to building HTTP APIs with less ceremony than controllers.

## Basic Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.Run();
```

## Route Handlers

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddDbContext<AppDbContext>();
builder.Services.AddScoped<IUserService, UserService>();

var app = builder.Build();

// GET all
app.MapGet("/api/users", async (AppDbContext db) =>
    await db.Users.ToListAsync());

// GET by id
app.MapGet("/api/users/{id:int}", async (int id, AppDbContext db) =>
{
    var user = await db.Users.FindAsync(id);
    return user is not null ? Results.Ok(user) : Results.NotFound();
});

// POST
app.MapPost("/api/users", async (UserDto dto, AppDbContext db) =>
{
    var user = new User { Name = dto.Name, Email = dto.Email };
    db.Users.Add(user);
    await db.SaveChangesAsync();
    return Results.Created($"/api/users/{user.Id}", user);
});

// PUT
app.MapPut("/api/users/{id:int}", async (int id, UserDto dto, AppDbContext db) =>
{
    var user = await db.Users.FindAsync(id);
    if (user is null) return Results.NotFound();

    user.Name = dto.Name;
    user.Email = dto.Email;
    await db.SaveChangesAsync();
    return Results.NoContent();
});

// DELETE
app.MapDelete("/api/users/{id:int}", async (int id, AppDbContext db) =>
{
    var user = await db.Users.FindAsync(id);
    if (user is null) return Results.NotFound();

    db.Users.Remove(user);
    await db.SaveChangesAsync();
    return Results.NoContent();
});

app.Run();
```

## Route Parameters

```csharp
// Path parameters
app.MapGet("/api/users/{id:int}", (int id) => { });
app.MapGet("/api/users/{name:alpha}", (string name) => { });
app.MapGet("/api/users/{guid:guid}", (Guid guid) => { });

// Query parameters
app.MapGet("/api/users", (string? search, int page = 1, int size = 10) =>
{
    // search, page, size from query string
});

// Mixed
app.MapGet("/api/users/{id:int}/orders", (int id, string? status) => { });
```

## Dependency Injection

```csharp
// Via constructor (app level)
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddScoped<IUserService, UserService>();

var app = builder.Build();

app.MapGet("/api/users", async (IUserService userService) =>
    await userService.GetAllAsync());

// With DbContext (scoped)
app.MapGet("/api/users/{id:int}", async (int id, AppDbContext db) =>
    await db.Users.FindAsync(id));

// Custom service
app.MapGet("/api/report", async ([FromServices] IReportService reportService) =>
    await reportService.GenerateAsync());

app.Run();
```

## Results

```csharp
// Success
Results.Ok(data);           // 200
Results.Created("/url", data); // 201
Results.Accepted();         // 202
Results.NoContent();        // 204

// Client errors
Results.BadRequest();       // 400
Results.BadRequest(error);  // 400 with body
Results.NotFound();         // 404
Results.Conflict();         // 409
Results.ValidationProblem(errors); // 400 with validation

// Redirect
Results.Redirect("/login");         // 302
Results.RedirectPermanent("/new");  // 301

// File
Results.File(byteArray, "text/plain");
Results.File(stream, "image/png");
Results.File("path/to/file.pdf", "application/pdf");

// Content negotiation
Results.Json(data);
Results.Text("plain text");
Results.Content("<xml>data</xml>", "text/xml");
```

## Groups

```csharp
var users = app.MapGroup("/api/users")
    .WithTags("Users")
    .RequireAuthorization();

users.MapGet("/", GetAllUsers);
users.MapGet("/{id:int}", GetUserById);
users.MapPost("/", CreateUser);
users.MapPut("/{id:int}", UpdateUser);
users.MapDelete("/{id:int}", DeleteUser);

// Nested group with common prefix
var admin = app.MapGroup("/admin")
    .RequireAuthorization("Admin");

admin.MapGet("/dashboard", GetDashboard);
admin.MapGet("/audit", GetAuditLog);
```

## Filters

```csharp
// Logging filter
app.MapGet("/api/users", async (AppDbContext db) =>
    await db.Users.ToListAsync())
    .AddFilter(async (context, next) =>
    {
        Console.WriteLine($"Request: {context.Request.Method} {context.Request.Path}");
        await next();
        Console.WriteLine($"Response: {context.Response.StatusCode}");
    });

// Validation filter
app.MapPost("/api/users", async (UserDto dto, AppDbContext db) =>
{
    // Handler
})
.AddFilter(async (context, next) =>
{
    var dto = await context.Request.ReadFromJsonAsync<UserDto>();
    if (dto is null || string.IsNullOrEmpty(dto.Name))
    {
        context.Response.StatusCode = 400;
        await context.Response.WriteAsJsonAsync(new { error = "Name required" });
        return;
    }
    await next();
});
```

## OpenAPI / Swagger

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.MapGet("/api/users", async (AppDbContext db) =>
    await db.Users.ToListAsync())
    .WithName("GetAllUsers")
    .WithTags("Users")
    .WithOpenApi(operation =>
    {
        operation.Summary = "Get all users";
        operation.Description = "Returns a list of all registered users";
        return operation;
    });

app.Run();
```

## Minimal API vs Controller

| Feature | Minimal API | Controller |
|---------|-------------|------------|
| Setup | Less code | More code |
| Organization | Single file | Separate files |
| Filters | Basic | Full (IActionFilter, etc.) |
| Model binding | Automatic | Extensive attributes |
| Validation | Manual | Data annotations |
| Best for | Simple APIs, microservices | Complex APIs, MVC apps |
