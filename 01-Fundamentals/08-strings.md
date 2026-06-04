# Strings and String Manipulation

## String Basics
```csharp
string str1 = "Hello";
string str2 = @"C:\path\to\file";     // Verbatim: no escape sequences
string str3 = $"Value: {x}";          // Interpolated
string str4 = """<tag>value</tag>"""; // Raw string literal (C# 11)
string empty = "";
string nullStr = null;
```

## String Immutability
Strings are immutable - every operation creates a new string:
```csharp
string s = "Hello";
s += " World";   // Creates a new string, original "Hello" is garbage collected
```

## StringBuilder (for efficient string building)
```csharp
var sb = new StringBuilder();
sb.Append("Hello");
sb.Append(" ");
sb.Append("World");
sb.AppendLine("!");
sb.Insert(0, "Start: ");
sb.Replace("World", "C#");
string result = sb.ToString();

// StringBuilder vs string concatenation
// Bad (creates many intermediate strings):
string s = "";
for (int i = 0; i < 1000; i++)
    s += i.ToString();

// Good:
var sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
    sb.Append(i);
string result = sb.ToString();
```

## Common String Methods

### Search and Comparison
```csharp
string text = "Hello World";

bool contains = text.Contains("World");     // true
bool starts = text.StartsWith("Hello");     // true
bool ends = text.EndsWith("World");         // true
int index = text.IndexOf("World");          // 6
int lastIndex = text.LastIndexOf('o');      // 7
bool equals = text.Equals("hello", StringComparison.OrdinalIgnoreCase); // true
int compare = string.Compare("A", "B");     // -1
```

### Manipulation
```csharp
string upper = text.ToUpper();             // "HELLO WORLD"
string lower = text.ToLower();             // "hello world"
string trimmed = "  text  ".Trim();        // "text"
string trimmedStart = "  text".TrimStart();
string trimmedEnd = "text  ".TrimEnd();

string replaced = text.Replace("World", "C#");  // "Hello C#"
string removed = text.Remove(5);                // "Hello"
string inserted = text.Insert(5, " Beautiful"); // "Hello Beautiful World"

string padded = "5".PadLeft(3, '0');           // "005"
string[] parts = text.Split(' ');              // ["Hello", "World"]
```

### Substring and Slicing
```csharp
string sub = text.Substring(6);          // "World"
string sub2 = text.Substring(0, 5);      // "Hello"
string slice = text[6..];                // "World" (C# 8+)
string slice2 = text[..5];               // "Hello" (C# 8+)
string slice3 = text[^5..];              // "World" (C# 8+)
string slice4 = text[1..^1];             // "ello Worl" (C# 8+)
```

### Joining and Formatting
```csharp
string joined = string.Join(", ", "A", "B", "C");  // "A, B, C"
string concat = string.Concat("A", "B", "C");       // "ABC"

// Format
string formatted = string.Format("Name: {0}, Age: {1}", "Alice", 25);
string interpolated = $"Name: {name}, Age: {age}";

// Format specifiers
string price = $"Price: {19.99:C}";     // "$19.99"
string percent = $"Percent: {0.85:P}";  // "85.00%"
string padded = $"Value: {42:D5}";      // "00042"
string hex = $"Hex: {255:X}";            // "FF"
```

## String Interpolation Features
```csharp
int x = 10, y = 20;
string result = $"{x} + {y} = {x + y}";

// Alignment and formatting
string aligned = $"Left: {"text",-10} Right: {"text",10}";
string formatted = $"Pi: {Math.PI:F2}";   // "Pi: 3.14"

// Interpolated verbatim (C# 8+)
string path = $@"{folder}\file.txt";

// Raw string interpolation (C# 11)
int a = 1, b = 2;
string raw = $$"""Sum: {{a + b}}""";  // "Sum: 3"
```

## String Comparison Rules
```csharp
// Ordinal (byte-by-byte, fastest)
bool eq = string.Equals("a", "b", StringComparison.Ordinal);

// Ordinal ignore case
bool eq2 = string.Equals("A", "a", StringComparison.OrdinalIgnoreCase);

// Current culture (for user-facing text)
bool eq3 = string.Equals("straße", "strasse", StringComparison.CurrentCulture);

// Invariant culture (for programmatic comparison)
bool eq4 = string.Equals("a", "b", StringComparison.InvariantCulture);
```

## Span<char> and Memory<char>
```csharp
string text = "Hello World";
ReadOnlySpan<char> span = text.AsSpan();
ReadOnlySpan<char> slice = span.Slice(0, 5);  // No allocation!
```

## StringBuilder Pool
```csharp
using System.Text;
var pool = StringBuilderPool.Shared;
StringBuilder sb = pool.Get();
try
{
    sb.Append("work");
    string result = sb.ToString();
}
finally
{
    pool.Return(sb);
}
```
