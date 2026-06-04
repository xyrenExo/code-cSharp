# Hello World and Basic Syntax

## Hello World Program

### Traditional C# (before .NET 6)
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

### Modern C# (.NET 6+ with top-level statements)
```csharp
Console.WriteLine("Hello, World!");
```

## Basic Syntax Elements

### Comments
```csharp
// Single-line comment

/*
   Multi-line comment
   spans multiple lines
*/

/// <summary>
/// XML documentation comment
/// </summary>
public void MyMethod() { }
```

### Entry Point
```csharp
// Traditional
static void Main(string[] args)
{
    // args contains command-line arguments
}

// Top-level statements (.NET 6+)
Console.WriteLine(args[0]); // args is implicitly available
```

### Namespaces
```csharp
using System;              // Import namespace
using System.Collections.Generic;

namespace MyApp.Models    // Declare namespace
{
    // types go here
}

// File-scoped namespace (C# 10+)
namespace MyApp.Models;
```

### Semicolons and Blocks
```csharp
int x = 5;                // Statement ends with ;

if (x > 0)                // Block with { }
{
    Console.WriteLine("Positive");
}
```

### Identifiers and Keywords
```csharp
int myVariable = 10;      // CamelCase for variables
int @class = 5;           // @ prefix to use reserved keywords
```

### Case Sensitivity
```csharp
int count = 10;
int Count = 20;           // Different variable (case-sensitive)
```

### Implicit Usings (C# 10+)
Generated based on SDK type:
- `Microsoft.NET.Sdk`: System, System.Collections.Generic, System.Linq, System.Threading, etc.
- `Microsoft.NET.Sdk.Web`: + ASP.NET Core namespaces

## Basic I/O
```csharp
// Output
Console.WriteLine("Hello");       // With newline
Console.Write("Hello");           // Without newline
Console.WriteLine($"Value: {x}"); // String interpolation

// Input
string input = Console.ReadLine();
int numericInput = int.Parse(Console.ReadLine());

// Formatted output
Console.WriteLine("{0} + {1} = {2}", a, b, a + b);
```

## Compilation and Execution
```bash
dotnet new console -n HelloWorld
cd HelloWorld
dotnet run
# Output: Hello, World!
```
