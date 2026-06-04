# Control Flow

## Conditional Statements

### if, else if, else
```csharp
int score = 85;

if (score >= 90)
{
    Console.WriteLine("A");
}
else if (score >= 80)
{
    Console.WriteLine("B");
}
else
{
    Console.WriteLine("C or lower");
}
```

### switch Statement
```csharp
int day = 3;
switch (day)
{
    case 1:
        Console.WriteLine("Monday");
        break;
    case 2:
        Console.WriteLine("Tuesday");
        break;
    case 3:
    case 4:
        Console.WriteLine("Midweek");
        break;
    default:
        Console.WriteLine("Weekend");
        break;
}
```

### switch Expression (C# 8+)
```csharp
string dayName = day switch
{
    1 => "Monday",
    2 => "Tuesday",
    3 or 4 => "Midweek",
    >= 5 and <= 7 => "End of week",
    _ => "Invalid"
};
```

### switch with Pattern Matching (C# 7+)
```csharp
object obj = 42;
switch (obj)
{
    case int i when i > 0:
        Console.WriteLine($"Positive int: {i}");
        break;
    case string s:
        Console.WriteLine($"String: {s}");
        break;
    case null:
        Console.WriteLine("Null");
        break;
}
```

## Loops

### for Loop
```csharp
for (int i = 0; i < 5; i++)
{
    Console.WriteLine(i);  // 0, 1, 2, 3, 4
}

// Multiple variables
for (int i = 0, j = 10; i < j; i++, j--)
{
    Console.WriteLine($"{i}-{j}");
}

// Infinite loop
for (;;)
{
    // ...
}
```

### foreach Loop
```csharp
var numbers = new[] { 1, 2, 3, 4, 5 };
foreach (var num in numbers)
{
    Console.WriteLine(num);
}

// With index (C# 8+)
foreach (var (item, index) in numbers.Select((v, i) => (v, i)))
{
    Console.WriteLine($"{index}: {item}");
}

// Iterate over dictionary
foreach (var kvp in dictionary)
{
    Console.WriteLine($"{kvp.Key}: {kvp.Value}");
}
```

### while Loop
```csharp
int i = 0;
while (i < 5)
{
    Console.WriteLine(i);
    i++;
}

// Infinite loop
while (true)
{
    if (condition) break;
}
```

### do-while Loop
```csharp
int i = 0;
do
{
    Console.WriteLine(i);
    i++;
} while (i < 5);
// Always executes at least once
```

## Jump Statements

### break
```csharp
for (int i = 0; i < 10; i++)
{
    if (i == 5) break;      // Exit loop
    Console.WriteLine(i);    // 0, 1, 2, 3, 4
}
```

### continue
```csharp
for (int i = 0; i < 10; i++)
{
    if (i % 2 == 0) continue; // Skip even numbers
    Console.WriteLine(i);      // 1, 3, 5, 7, 9
}
```

### return
```csharp
int Sum(int a, int b)
{
    return a + b;            // Exit method with value
}

void Log(string message)
{
    if (string.IsNullOrEmpty(message)) return; // Exit method
    Console.WriteLine(message);
}
```

### goto
```csharp
// Generally avoided, but valid for nested loop exit
for (int i = 0; i < 10; i++)
{
    for (int j = 0; j < 10; j++)
    {
        if (i * j > 50) goto exit;
    }
}
exit:
Console.WriteLine("Exited nested loops");
```

## Pattern Matching in if (C# 7+)
```csharp
object value = "Hello";

if (value is string text)
{
    Console.WriteLine(text.Length);  // Pattern match with variable
}

if (value is int i && i > 0)
{
    Console.WriteLine("Positive int");
}
```

## Using Declaration (C# 8+)
```csharp
// Automatically disposed at end of scope
using var file = new StreamReader("file.txt");
string content = file.ReadToEnd();
```

## Summary
| Statement | Use Case |
|-----------|----------|
| `if/else` | Conditional branching |
| `switch` | Multiple discrete cases |
| `for` | Known iteration count |
| `foreach` | Iterating collections |
| `while` | Unknown iteration count |
| `do-while` | At least one execution |
| `break` | Exit loop early |
| `continue` | Skip to next iteration |
