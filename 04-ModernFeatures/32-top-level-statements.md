# Top-Level Statements

## Introduction (C# 10 / .NET 6+)
Top-level statements allow you to write the main program logic without the ceremony of a class and `Main` method.

### Traditional Program.cs
```csharp
using System;

namespace MyApp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, World!");
        }
    }
}
```

### With Top-Level Statements
```csharp
// Program.cs - entire file
Console.WriteLine("Hello, World!");
```

## Accessing Command-Line Arguments
```csharp
// args is implicitly available
if (args.Length > 0)
{
    Console.WriteLine($"Arguments: {string.Join(", ", args)}");
}
else
{
    Console.WriteLine("No arguments provided");
}
```

## Returning Exit Codes
```csharp
// Return int directly
if (args.Length == 0)
{
    Console.WriteLine("Usage: myapp <name>");
    return 1;
}

Console.WriteLine($"Hello, {args[0]}");
return 0;
```

## Async Top-Level Statements
```csharp
// await is available directly
Console.WriteLine("Downloading...");
string data = await new HttpClient().GetStringAsync("https://example.com");
Console.WriteLine($"Downloaded {data.Length} characters");
```

## Declaring Local Functions
```csharp
// Local functions are permitted
Console.WriteLine(Calculate(5, 3));

static int Calculate(int a, int b)
{
    return Add(a, b) * Subtract(a, b);
}

static int Add(int a, int b) => a + b;
static int Subtract(int a, int b) => a - b;
```

## Mixing with Other Type Declarations
```csharp
// Top-level statements FIRST, then type declarations
Console.WriteLine(new Person("Alice").Greet());

public record Person(string Name)
{
    public string Greet() => $"Hello, {Name}!";
}
```

## Using Statements and Global Usings
```csharp
// Usings at top are implicitly global to the file
using System.Net.Http;
using System.Text.Json;

// These usings are automatically included (.NET 6+):
// System, System.Collections.Generic, System.IO,
// System.Linq, System.Net.Http, System.Threading,
// System.Threading.Tasks
```

## Scoping Rules
```csharp
// Top-level statements are in the generated <Main>$ method
// They share a common scope

int x = 10;
int y = 20;

// Can reference earlier variables
Console.WriteLine(x + y);

// Cannot have duplicate names
// int x = 30;  // Error!
```

## Multiple Files Limitation
```csharp
// Only ONE file can have top-level statements per project
// Usually Program.cs - other files must have full type declarations

// Program.cs:
Console.WriteLine("Entry point");

// Other.cs:
public class Helper
{
    public static void DoSomething() { }
}
```

## How It Works
The compiler generates the equivalent of:

```csharp
using System;
using System.Threading.Tasks;

[CompilerGenerated]
internal class Program
{
    private static void Main(string[] args)
    {
        // Your top-level statements go here
        Console.WriteLine("Hello, World!");
    }
}
```

## Practical Examples

### Simple Console App
```csharp
// Program.cs
Console.Write("Enter your name: ");
var name = Console.ReadLine();
Console.WriteLine($"Hello, {name}!");
Console.WriteLine("Press any key to exit...");
Console.ReadKey();
```

### Web Scraper
```csharp
using System.Text.RegularExpressions;

var client = new HttpClient();
string html = await client.GetStringAsync("https://example.com");
var titles = Regex.Matches(html, @"<title>(.*?)</title>", RegexOptions.IgnoreCase);

foreach (Match title in titles)
{
    Console.WriteLine(title.Groups[1].Value);
}
```

### File Processor
```csharp
string directory = args.Length > 0 ? args[0] : Environment.CurrentDirectory;
var files = Directory.GetFiles(directory, "*.txt");

foreach (var file in files)
{
    string content = File.ReadAllText(file);
    int wordCount = content.Split(' ', StringSplitOptions.RemoveEmptyEntries).Length;
    Console.WriteLine($"{Path.GetFileName(file)}: {wordCount} words");
}

Console.WriteLine($"Processed {files.Length} files");
```
