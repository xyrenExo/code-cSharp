# Type Conversion and Casting

## Implicit Conversions (Safe, No Data Loss)
```csharp
// Numeric widening
int i = 100;
long l = i;              // int → long
float f = i;             // int → float
double d = i;            // int → double
decimal m = i;           // int → decimal

// Derived to base (reference types)
string s = "Hello";
object obj = s;

// Value type to nullable
int? nullable = i;
```

## Explicit Conversions (Cast, Potential Data Loss)
```csharp
double d = 3.14;
int i = (int)d;              // 3 (truncation)

long l = 1234567890123L;
int narrowed = (int)l;       // Data loss possible

object obj = "Hello";
string s = (string)obj;      // Must be string type
```

## Conversion Methods

### Parse and TryParse
```csharp
string input = "123";

int num1 = int.Parse(input);                // Throws on failure
bool success = int.TryParse(input, out int num2);  // Safe

// Other types
long l = long.Parse("123456");
double d = double.Parse("3.14");
bool b = bool.Parse("true");
DateTime dt = DateTime.Parse("2024-01-01");
```

### Convert Class
```csharp
int i = Convert.ToInt32("123");             // null → 0
double d = Convert.ToDouble("3.14");
string s = Convert.ToString(123);
bool b = Convert.ToBoolean("true");
byte[] bytes = Convert.FromBase64String("base64string");
string base64 = Convert.ToBase64String(bytes);
```

### ToString
```csharp
int i = 123;
string s1 = i.ToString();                  // "123"
string s2 = i.ToString("X");               // "7B" (hex)
string s3 = 3.14.ToString("F2");           // "3.14"
string s4 = 0.5.ToString("P0");            // "50%"
```

## Type Testing Operators

### is Operator
```csharp
object obj = "Hello";

if (obj is string) { }                     // Type check only
if (obj is string s) { }                   // Pattern match with variable
if (obj is string s && s.Length > 0) { }   // With condition

// C# 7+ pattern matching
if (obj is int i && i > 0)
    Console.WriteLine($"Positive int: {i}");

// negation (C# 9+)
if (obj is not null) { }
```

### as Operator
```csharp
object obj = "Hello";
string s = obj as string;                  // "Hello" or null
int? num = obj as int?;                    // null (not an int)

// Safe to use without try-catch
if (s is not null)
    Console.WriteLine(s.Length);
```

## Boxing and Unboxing

### Boxing (Value → Reference, Allocation)
```csharp
int value = 42;
object boxed = value;       // Boxing: value copied to heap
```

### Unboxing (Reference → Value)
```csharp
object boxed = 42;
int value = (int)boxed;     // Unboxing: must be exact type
```

### Performance Impact
```csharp
// Avoid boxing in collections:
ArrayList list = new ArrayList();   // Old, boxes values
list.Add(42);                       // Boxing!
int val = (int)list[0];           // Unboxing!

// Use generics instead:
List<int> list = new List<int>();   // No boxing
list.Add(42);
int val = list[0];
```

## Best Practices

### Safe Conversion Pattern
```csharp
// Prefer TryParse over Parse
if (int.TryParse(input, out int result))
    UseValue(result);
else
    HandleError();

// Prefer pattern matching over as + null check
if (obj is string s)
    UseString(s);

// Prefer Convert for unknown types
object val = GetValue();
int i = Convert.ToInt32(val);  // Safe for many types
```

## Conversion Table
| From | To | Method |
|------|----|--------|
| `string` → `int` | `int.Parse()`, `int.TryParse()` |
| `string` → `double` | `double.Parse()`, `double.TryParse()` |
| `string` → `DateTime` | `DateTime.Parse()`, `DateTime.TryParse()` |
| `object` → `string` | `(string)` cast, `as string`, `.ToString()` |
| `int` → `string` | `.ToString()`, `Convert.ToString()` |
| `byte[]` → `string` | `Encoding.UTF8.GetString()` |
| `string` → `byte[]` | `Encoding.UTF8.GetBytes()` |
| Any type | `Convert.ToXxx()` |
| Any compatible type | `(TargetType)` cast |
