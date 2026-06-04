# Attributes

## Introduction
Attributes provide metadata about code elements (classes, methods, properties, etc.) that can be queried at runtime via reflection.

## Built-in Attributes

### Common Attributes
```csharp
// Compiler information
[Obsolete("Use NewMethod instead", error: true)]
public void OldMethod() { }

// Serialization
[Serializable]
public class Data { }

[NonSerialized]
private string _tempData;

// JSON serialization (System.Text.Json)
[JsonPropertyName("user_name")]
public string UserName { get; set; }

[JsonIgnore]
public string InternalField { get; set; }

// XML serialization
[XmlRoot("order")]
public class Order
{
    [XmlAttribute("id")]
    public int Id { get; set; }

    [XmlElement("item")]
    public List<string> Items { get; set; }

    [XmlIgnore]
    public decimal InternalTotal { get; set; }
}

// Validation (Data Annotations)
[Required]
[StringLength(100, MinimumLength = 3)]
[Range(1, 100)]
[EmailAddress]
[Phone]
[CreditCard]
[RegularExpression(@"^[a-zA-Z]+$")]
public string Property { get; set; }
```

### Caller Info Attributes
```csharp
public void Log(
    string message,
    [CallerMemberName] string member = "",
    [CallerFilePath] string filePath = "",
    [CallerLineNumber] int lineNumber = 0)
{
    Console.WriteLine($"{member} ({filePath}:{lineNumber}): {message}");
}

// Usage (compiler fills in values automatically)
Log("Something happened");
```

### Conditional Attribute
```csharp
#define DEBUG

[Conditional("DEBUG")]
public void DebugOnlyMethod()
{
    Console.WriteLine("Debug message");
}

// Call is only compiled when DEBUG is defined
DebugOnlyMethod();
```

## Custom Attributes

### Defining Custom Attributes
```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, AllowMultiple = true)]
public class AuthorAttribute : Attribute
{
    public string Name { get; }
    public string? Email { get; set; }
    public string? Version { get; set; }

    public AuthorAttribute(string name)
    {
        Name = name;
    }
}

// Usage
[Author("Alice", Email = "alice@example.com", Version = "1.0")]
[Author("Bob", Version = "2.0")]
public class SampleClass
{
    [Author("Charlie")]
    public void Method() { }
}
```

### AttributeUsage
```csharp
[AttributeUsage(
    AttributeTargets.Class |      // Where it can be applied
    AttributeTargets.Struct |
    AttributeTargets.Method |
    AttributeTargets.Property |
    AttributeTargets.Field |
    AttributeTargets.Parameter |
    AttributeTargets.ReturnValue |
    AttributeTargets.All,
    AllowMultiple = true,          // Can be applied multiple times
    Inherited = true               // Inherited by derived classes
)]
public class MyCustomAttribute : Attribute { }
```

### Constructor vs Property Parameters
```csharp
public class ConfigAttribute : Attribute
{
    // Positional: passed through constructor
    public string Key { get; }
    public object DefaultValue { get; }

    // Named: set via property
    public string Description { get; set; }
    public bool Required { get; set; }

    public ConfigAttribute(string key, object defaultValue)
    {
        Key = key;
        DefaultValue = defaultValue;
    }
}

// Usage
[Config("app:setting", 42, Description = "Application setting", Required = true)]
public int AppSetting { get; set; }
```

## Reading Attributes at Runtime

```csharp
// Check if attribute exists
if (Attribute.IsDefined(typeof(MyClass), typeof(SerializableAttribute)))
{
    Console.WriteLine("MyClass is serializable");
}

// Get single attribute
var author = typeof(MyClass).GetCustomAttribute<AuthorAttribute>();
Console.WriteLine($"Author: {author?.Name}");

// Get multiple attributes
var authors = typeof(MyClass).GetCustomAttributes<AuthorAttribute>();
foreach (var author in authors)
{
    Console.WriteLine($"{author.Name} (v{author.Version})");
}

// Method attributes
var method = typeof(MyClass).GetMethod("MyMethod");
var obsolete = method?.GetCustomAttribute<ObsoleteAttribute>();
if (obsolete is not null)
{
    Console.WriteLine($"Method is obsolete: {obsolete.Message}");
}
```

## Validation with Attributes
```csharp
public static class Validator
{
    public static bool Validate(object obj)
    {
        var properties = obj.GetType().GetProperties();
        foreach (var prop in properties)
        {
            var value = prop.GetValue(obj);

            // Required check
            if (Attribute.IsDefined(prop, typeof(RequiredAttribute)))
            {
                if (value is null || (value is string s && string.IsNullOrEmpty(s)))
                {
                    Console.WriteLine($"{prop.Name} is required");
                    return false;
                }
            }

            // Range check
            var range = prop.GetCustomAttribute<RangeAttribute>();
            if (range is not null && value is int intValue)
            {
                if (intValue < (int)range.Minimum || intValue > (int)range.Maximum)
                {
                    Console.WriteLine($"{prop.Name} out of range");
                    return false;
                }
            }
        }
        return true;
    }
}
```

## Common .NET Attributes Reference
| Attribute | Purpose |
|-----------|---------|
| `[Serializable]` | Type can be serialized |
| `[Obsolete]` | Marks deprecated code |
| `[Conditional]` | Conditional compilation |
| `[CallerMemberName]` | Gets caller's method name |
| `[CallerFilePath]` | Gets caller's source file |
| `[CallerLineNumber]` | Gets caller's line number |
| `[DebuggerDisplay]` | Custom debug display |
| `[DebuggerStepThrough]` | Skip debugging |
| `[Flags]` | Enum is bit field |
| `[DllImport]` | P/Invoke declaration |
| `[JsonPropertyName]` | JSON property name |
| `[JsonIgnore]` | Skip JSON serialization |
| `[Required]` | Validation required |
| `[StringLength]` | String length validation |
| `[Range]` | Numeric range validation |
| `[DisplayName]` | Display name for UI |
| `[Description]` | Description for UI |
| `[TestClass]` | Test class (MSTest) |
| `[TestMethod]` | Test method (MSTest) |
| `[Fact]` | Test method (xUnit) |
| `[Theory]` | Parameterized test (xUnit) |
