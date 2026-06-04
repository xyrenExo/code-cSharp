# Operators and Expressions

## Arithmetic Operators
```csharp
int a = 10, b = 3;
int sum = a + b;           // 13 (addition)
int diff = a - b;          // 7 (subtraction)
int product = a * b;       // 30 (multiplication)
int quotient = a / b;      // 3 (integer division)
int remainder = a % b;     // 1 (modulus)

// Floating-point division
double div = 10.0 / 3.0;   // 3.333...
```

## Assignment Operators
```csharp
int x = 5;
x += 3;     // x = 8
x -= 2;     // x = 6
x *= 4;     // x = 24
x /= 3;     // x = 8
x %= 5;     // x = 3
x <<= 2;    // x = 12 (left shift)
x >>= 1;    // x = 6 (right shift)
x &= 2;     // x = 2 (bitwise AND)
x |= 4;     // x = 6 (bitwise OR)
x ^= 2;     // x = 4 (bitwise XOR)
```

## Comparison Operators
```csharp
int a = 5, b = 10;
bool eq = a == b;     // false
bool neq = a != b;    // true
bool lt = a < b;      // true
bool gt = a > b;      // false
bool lte = a <= b;    // true
bool gte = a >= b;    // false
```

## Logical Operators
```csharp
bool a = true, b = false;
bool and = a && b;     // false (short-circuiting)
bool or = a || b;      // true (short-circuiting)
bool not = !a;         // false

// Non-short-circuiting versions
bool and2 = a & b;     // false (evaluates both)
bool or2 = a | b;      // true (evaluates both)
bool xor = a ^ b;      // true (XOR)
```

## Bitwise Operators
```csharp
uint a = 0b1100;        // 12
uint b = 0b1010;        // 10
uint and = a & b;       // 0b1000 (8)
uint or = a | b;        // 0b1110 (14)
uint xor = a ^ b;       // 0b0110 (6)
uint not = ~a;          // Inverts all bits
uint left = a << 2;     // 0b110000 (48)
uint right = a >> 2;    // 0b0011 (3)
```

## Null-Conditional Operators
```csharp
string text = null;
int? length = text?.Length;           // null (no exception)
char? first = text?[0];               // null

// Null-conditional with coalescing
int result = text?.Length ?? 0;       // 0

// Null-conditional invocation
Action action = null;
action?.Invoke();                     // No exception
```

## Null Coalescing Operator
```csharp
string name = null;
string display = name ?? "Default";   // "Default"

// Null coalescing assignment (C# 8+)
name ??= "Assigned";                  // Only assigns if null
```

## Conditional (Ternary) Operator
```csharp
int age = 20;
string status = age >= 18 ? "Adult" : "Minor";

// Nested ternary (use sparingly)
string result = score >= 90 ? "A" : score >= 80 ? "B" : "C";
```

## Type Testing and Casting Operators
```csharp
object obj = "Hello";
bool isString = obj is string;              // true
bool isPattern = obj is string s;           // true, s = "Hello"
string casted = (string)obj;                // Explicit cast
string? safe = obj as string;               // "Hello" or null
```

## sizeof and typeof Operators
```csharp
int size = sizeof(int);                    // 4
Type type = typeof(string);                // System.String
```

## Switch Expression (C# 8+)
```csharp
string grade = score switch
{
    >= 90 => "A",
    >= 80 => "B",
    >= 70 => "C",
    _ => "F"                               // Default case
};
```

## Operator Precedence (Highest to Lowest)
| Category | Operators |
|----------|-----------|
| Primary | `x.y`, `f(x)`, `a[x]`, `x++`, `x--`, `new`, `typeof`, `checked`, `unchecked` |
| Unary | `+`, `-`, `!`, `~`, `++x`, `--x`, `(T)x`, `await`, `&`, `*` |
| Multiplicative | `*`, `/`, `%` |
| Additive | `+`, `-` |
| Shift | `<<`, `>>` |
| Relational | `<`, `>`, `<=`, `>=`, `is`, `as` |
| Equality | `==`, `!=` |
| Bitwise AND | `&` |
| Bitwise XOR | `^` |
| Bitwise OR | `|` |
| Conditional AND | `&&` |
| Conditional OR | `||` |
| Null Coalescing | `??` |
| Ternary | `?:` |
| Assignment | `=`, `+=`, `-=`, etc. |
