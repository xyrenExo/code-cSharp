# Variables and Data Types

## Value Types (Stack-allocated)

### Integral Types
| Type | Size | Range | Default |
|------|------|-------|---------|
| `byte` | 8-bit | 0 to 255 | 0 |
| `sbyte` | 8-bit | -128 to 127 | 0 |
| `short` | 16-bit | -32,768 to 32,767 | 0 |
| `ushort` | 16-bit | 0 to 65,535 | 0 |
| `int` | 32-bit | -2.1B to 2.1B | 0 |
| `uint` | 32-bit | 0 to 4.2B | 0 |
| `long` | 64-bit | -9.2E18 to 9.2E18 | 0 |
| `ulong` | 64-bit | 0 to 1.8E19 | 0 |
| `nint` | Platform | Platform-specific | 0 |
| `nuint` | Platform | Platform-specific | 0 |

### Floating-Point Types
| Type | Size | Precision | Range |
|------|------|-----------|-------|
| `float` | 32-bit | ~7 digits | ±1.5E-45 to ±3.4E38 |
| `double` | 64-bit | ~15-16 digits | ±5.0E-324 to ±1.7E308 |
| `decimal` | 128-bit | 28-29 digits | ±1.0E-28 to ±7.9E28 |

### Other Value Types
```csharp
bool isActive = true;        // true or false
char grade = 'A';            // 16-bit Unicode character
```

## Reference Types (Heap-allocated)

| Type | Example | Default |
|------|---------|---------|
| `string` | `"Hello"` | `null` |
| `object` | Base of all types | `null` |
| `dynamic` | Runtime-typed | `null` |
| `class` | User-defined | `null` |
| `interface` | Contract | `null` |
| `delegate` | Method reference | `null` |
| `record` | Immutable data | `null` |

## Variable Declaration

### Explicit Typing
```csharp
int age = 25;
string name = "Alice";
double price = 19.99;
bool isComplete = false;
```

### Implicit Typing (var)
```csharp
var count = 10;              // int
var name = "Bob";            // string
var items = new List<int>(); // List<int>

// Cannot use var for:
// - Method parameters
// - Class fields
// - Without initialization
```

### Constant and Read-only
```csharp
const double Pi = 3.14159;           // Compile-time constant
readonly int MaxValue = 100;         // Runtime constant (field only)
```

### Default Values
```csharp
int defaultInt = default;            // 0
bool defaultBool = default;          // false
string defaultString = default;      // null
var defaultValue = default(int);     // 0
```

## Numeric Literals

### Digit Separators and Suffixes
```csharp
int million = 1_000_000;
long big = 1_000_000_000_000L;
float f = 3.14f;
double d = 3.14;
decimal m = 19.99m;
uint ui = 100U;
ulong ul = 100UL;
```

## Nullable Value Types
```csharp
int? nullableInt = null;            // Nullable<int>
bool? flag = null;

// Checking null
if (nullableInt.HasValue)
{
    Console.WriteLine(nullableInt.Value);
}

// Null coalescing
int result = nullableInt ?? -1;

// Null-conditional
int? length = text?.Length;
```

## Strings
```csharp
string str1 = "Hello";
string str2 = @"Literal\nstring";     // Verbatim string (no escape)
string str3 = $"Value: {x}";          // Interpolated string
string str4 = """Raw string literal"""; // Raw string (C# 11)
string empty = string.Empty;
bool isNull = str1 is null;
int len = str1.Length;
```

## Type Information
```csharp
Type t = typeof(int);
bool isInt = someVar is int;
string typeName = someVar.GetType().Name;
```
