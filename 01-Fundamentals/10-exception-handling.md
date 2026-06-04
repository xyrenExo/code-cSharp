# Exception Handling

## Try-Catch-Finally
```csharp
try
{
    // Code that may throw
    int result = Divide(10, 0);
}
catch (DivideByZeroException ex)
{
    // Handle specific exception
    Console.WriteLine($"Cannot divide by zero: {ex.Message}");
}
catch (Exception ex)
{
    // Handle all other exceptions
    Console.WriteLine($"Error: {ex.Message}");
    throw;  // Re-throw preserving stack trace
}
finally
{
    // Always executes (cleanup)
    CloseResources();
}
```

## Custom Exceptions
```csharp
public class InvalidAgeException : Exception
{
    public InvalidAgeException() { }

    public InvalidAgeException(string message)
        : base(message) { }

    public InvalidAgeException(string message, Exception inner)
        : base(message, inner) { }

    public int Age { get; set; }
}

// Usage
throw new InvalidAgeException("Age cannot be negative")
{
    Age = -5
};
```

## Exception Properties
```csharp
try
{
    throw new ArgumentNullException("paramName");
}
catch (Exception ex)
{
    Console.WriteLine(ex.Message);       // Human-readable description
    Console.WriteLine(ex.StackTrace);    // Call stack
    Console.WriteLine(ex.InnerException); // Nested exception
    Console.WriteLine(ex.Source);        // App that caused error
    Console.WriteLine(ex.HResult);       // HRESULT code
    Console.WriteLine(ex.TargetSite);    // Method that threw
}
```

## Common Exception Types
| Exception | When Thrown |
|-----------|-------------|
| `ArgumentNullException` | Null argument passed |
| `ArgumentOutOfRangeException` | Argument outside valid range |
| `ArgumentException` | Invalid argument |
| `InvalidOperationException` | Object state doesn't allow operation |
| `NullReferenceException` | Accessing null object's member |
| `IndexOutOfRangeException` | Array index out of bounds |
| `DivideByZeroException` | Division by zero |
| `FileNotFoundException` | File not found |
| `FormatException` | Invalid format for conversion |
| `TimeoutException` | Operation timed out |
| `NotImplementedException` | Method not implemented |
| `UnauthorizedAccessException` | Access denied |

## Throwing Exceptions
```csharp
void ValidateAge(int age)
{
    if (age < 0)
        throw new ArgumentOutOfRangeException(
            nameof(age), "Age cannot be negative");
    if (age > 150)
        throw new ArgumentOutOfRangeException(
            nameof(age), "Age exceeds maximum");
}

// Throw helper (C# 7+)
public string Name
{
    get => name;
    set => name = value ?? throw new ArgumentNullException(nameof(value));
}
```

## Exception Filtering (C# 6+)
```csharp
try
{
    DoSomething();
}
catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.NotFound)
{
    // Only catch 404 errors
}
catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.Forbidden)
{
    // Only catch 403 errors
}
```

## Using and Dispose Pattern
```csharp
// Safe resource management
using (var file = new StreamReader("file.txt"))
{
    string content = file.ReadToEnd();
}  // Automatically disposed

// Using declaration (C# 8+)
using var connection = new SqlConnection(connectionString);
connection.Open();
// Disposed at end of scope
```

## Global Exception Handling
```csharp
// Console app
AppDomain.CurrentDomain.UnhandledException += (sender, args) =>
{
    Exception ex = (Exception)args.ExceptionObject;
    Console.WriteLine($"Unhandled: {ex.Message}");
};

// ASP.NET Core middleware
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        context.Response.StatusCode = 500;
        await context.Response.WriteAsync("An error occurred");
    });
});
```

## Best Practices

### When to Catch
```csharp
// Good: Handle expected exceptions
try
{
    await file.ReadAsync();
}
catch (FileNotFoundException)
{
    // Handle missing file
}

// Bad: Swallowing exceptions
try { DoSomething(); }
catch { }  // Never do this!

// Bad: Catching Exception unnecessarily
try { DoSomething(); }
catch (Exception ex)
{
    Log(ex);
    throw;  // OK if rethrowing
}
```

### When to Throw
- Invalid arguments: `ArgumentNullException`, `ArgumentOutOfRangeException`
- Invalid state: `InvalidOperationException`
- Not supported: `NotSupportedException`
- Not implemented: `NotImplementedException`

### Performance Considerations
```csharp
// Avoid exceptions for control flow
// Bad:
int ParseInt(string s)
{
    try { return int.Parse(s); }
    catch { return 0; }
}

// Good:
bool TryParseInt(string s, out int result)
{
    return int.TryParse(s, out result);
}
```
