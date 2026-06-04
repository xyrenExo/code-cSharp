# ASP.NET Core Web API

## Creating a Web API

```bash
dotnet new webapi -n MyApi
cd MyApi
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```

## Program.cs (Minimal Host)

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllers();
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Middleware pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

## Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly AppDbContext _context;

    public UsersController(AppDbContext context)
    {
        _context = context;
    }

    // GET: api/users
    [HttpGet]
    public async Task<ActionResult<List<User>>> GetAll()
    {
        return await _context.Users.ToListAsync();
    }

    // GET: api/users/5
    [HttpGet("{id}")]
    public async Task<ActionResult<User>> GetById(int id)
    {
        var user = await _context.Users.FindAsync(id);
        if (user is null)
            return NotFound();
        return user;
    }

    // POST: api/users
    [HttpPost]
    public async Task<ActionResult<User>> Create(UserDto dto)
    {
        var user = new User { Name = dto.Name, Email = dto.Email };
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        return CreatedAtAction(nameof(GetById), new { id = user.Id }, user);
    }

    // PUT: api/users/5
    [HttpPut("{id}")]
    public async Task<IActionResult> Update(int id, UserDto dto)
    {
        var user = await _context.Users.FindAsync(id);
        if (user is null)
            return NotFound();

        user.Name = dto.Name;
        user.Email = dto.Email;
        await _context.SaveChangesAsync();

        return NoContent();
    }

    // DELETE: api/users/5
    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        var user = await _context.Users.FindAsync(id);
        if (user is null)
            return NotFound();

        _context.Users.Remove(user);
        await _context.SaveChangesAsync();

        return NoContent();
    }
}
```

## Routing

```csharp
[Route("api/[controller]")]
[Route("api/v{version:apiVersion}/[controller]")]

// Attribute routing
[HttpGet]
[HttpGet("{id}")]
[HttpPost]
[HttpPut("{id}")]
[HttpPatch("{id}")]
[HttpDelete("{id}")]

// Route constraints
[HttpGet("{id:int}")]
[HttpGet("{name:alpha}")]
[HttpGet("{guid:guid}")]
[HttpGet("{date:datetime}")]
```

## Model Binding

```csharp
// From route
[HttpGet("{id}")]
public ActionResult Get([FromRoute] int id)

// From query
[HttpGet]
public ActionResult Get([FromQuery] string name, [FromQuery] int page = 1)

// From body
[HttpPost]
public ActionResult Create([FromBody] UserDto dto)

// From header
[HttpGet]
public ActionResult Get([FromHeader(Name = "X-API-Key")] string apiKey)

// From form
[HttpPost]
public ActionResult Upload([FromForm] IFormFile file)

// Mixed
[HttpPut("{id}")]
public ActionResult Update(int id, [FromBody] UserDto dto)
```

## DTOs and Validation

```csharp
public record UserDto
{
    [Required]
    [StringLength(100, MinimumLength = 2)]
    public string Name { get; init; } = string.Empty;

    [Required]
    [EmailAddress]
    public string Email { get; init; } = string.Empty;

    [Range(0, 150)]
    public int Age { get; init; }
}

// Validation with FluentValidation
public class UserDtoValidator : AbstractValidator<UserDto>
{
    public UserDtoValidator()
    {
        RuleFor(x => x.Name).NotEmpty().Length(2, 100);
        RuleFor(x => x.Email).NotEmpty().EmailAddress();
        RuleFor(x => x.Age).InclusiveBetween(0, 150);
    }
}
```

## Response Types

```csharp
[HttpGet]
public ActionResult<List<User>> GetAll()  // 200 OK

[HttpGet("{id}")]
public ActionResult<User> GetById(int id)  // 200 or 404

[HttpPost]
public ActionResult<User> Create(UserDto dto)  // 201 Created

[HttpPut("{id}")]
public IActionResult Update(int id, UserDto dto)  // 204 NoContent

[HttpDelete("{id}")]
public IActionResult Delete(int id)  // 204 NoContent

// Custom status codes
return Ok(data);          // 200
return Created("url", data);  // 201
return NoContent();       // 204
return BadRequest();      // 400
return Unauthorized();    // 401
return NotFound();        // 404
return Conflict();        // 409
return Problem("Error");  // 500
```

## Error Handling

```csharp
// Global exception handler
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        context.Response.StatusCode = 500;
        context.Response.ContentType = "application/json";

        var error = context.Features.Get<IExceptionHandlerFeature>();
        if (error is not null)
        {
            await context.Response.WriteAsJsonAsync(new
            {
                error = "An error occurred",
                detail = app.Environment.IsDevelopment() ? error.Error.Message : null
            });
        }
    });
});
```

## Middleware

```csharp
// Custom middleware
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;

    public RequestLoggingMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context)
    {
        Console.WriteLine($"Request: {context.Request.Method} {context.Request.Path}");
        await _next(context);
        Console.WriteLine($"Response: {context.Response.StatusCode}");
    }
}

// Register
app.UseMiddleware<RequestLoggingMiddleware>();
```
