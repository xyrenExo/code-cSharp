# Operator Overloading

## Overloadable Operators

| Category | Operators |
|----------|-----------|
| Unary | `+`, `-`, `!`, `~`, `++`, `--`, `true`, `false` |
| Binary | `+`, `-`, `*`, `/`, `%`, `&`, `\|`, `^`, `<<`, `>>` |
| Comparison | `==`, `!=`, `<`, `>`, `<=`, `>=` |
| Compound | `+=`, `-=`, etc. (overridden by binary operators) |

## Non-Overloadable Operators
| Operator | Reason |
|----------|--------|
| `&&`, `\|\|` | Short-circuit (built-in) |
| `[]` | Use indexers instead |
| `()` | Use methods/invocation |
| `=` | Assignment (built-in) |
| `.` | Member access (always) |
| `?:` | Ternary (built-in) |
| `?.`, `??` | Null-conditional (built-in) |
| `is`, `as`, `typeof` | Type testing (built-in) |
| `->` | Pointer (unsafe context) |

## Basic Overload Example
```csharp
public readonly struct Vector2D
{
    public double X { get; }
    public double Y { get; }

    public Vector2D(double x, double y) => (X, Y) = (x, y);

    // Binary operators
    public static Vector2D operator +(Vector2D a, Vector2D b)
        => new(a.X + b.X, a.Y + b.Y);

    public static Vector2D operator -(Vector2D a, Vector2D b)
        => new(a.X - b.X, a.Y - b.Y);

    public static Vector2D operator *(Vector2D a, double scalar)
        => new(a.X * scalar, a.Y * scalar);

    public static Vector2D operator *(double scalar, Vector2D a)
        => a * scalar;  // Commutative

    // Unary operators
    public static Vector2D operator -(Vector2D v) => new(-v.X, -v.Y);

    public static Vector2D operator ++(Vector2D v) => new(v.X + 1, v.Y + 1);

    // Comparison operators (must be in pairs)
    public static bool operator ==(Vector2D a, Vector2D b)
        => a.X == b.X && a.Y == b.Y;

    public static bool operator !=(Vector2D a, Vector2D b)
        => !(a == b);

    // Must override Equals and GetHashCode with ==/!=
    public override bool Equals(object? obj)
        => obj is Vector2D other && this == other;

    public override int GetHashCode() => HashCode.Combine(X, Y);

    public override string ToString() => $"({X}, {Y})";
}
```

## Full Example: Complex Number
```csharp
public readonly struct Complex
{
    public double Real { get; }
    public double Imaginary { get; }

    public Complex(double real, double imaginary)
        => (Real, Imaginary) = (real, imaginary);

    // Arithmetic
    public static Complex operator +(Complex a, Complex b)
        => new(a.Real + b.Real, a.Imaginary + b.Imaginary);

    public static Complex operator -(Complex a, Complex b)
        => new(a.Real - b.Real, a.Imaginary - b.Imaginary);

    public static Complex operator *(Complex a, Complex b)
        => new(
            a.Real * b.Real - a.Imaginary * b.Imaginary,
            a.Real * b.Imaginary + a.Imaginary * b.Real
        );

    public static Complex operator /(Complex a, Complex b)
    {
        double denominator = b.Real * b.Real + b.Imaginary * b.Imaginary;
        return new Complex(
            (a.Real * b.Real + a.Imaginary * b.Imaginary) / denominator,
            (a.Imaginary * b.Real - a.Real * b.Imaginary) / denominator
        );
    }

    // Unary
    public static Complex operator -(Complex c) => new(-c.Real, -c.Imaginary);

    // Conversion
    public static implicit operator Complex(double value) => new(value, 0);
    public static explicit operator double(Complex c) => c.Real;

    // Comparison
    public static bool operator ==(Complex a, Complex b)
        => a.Real == b.Real && a.Imaginary == b.Imaginary;

    public static bool operator !=(Complex a, Complex b) => !(a == b);

    public override bool Equals(object? obj)
        => obj is Complex other && this == other;

    public override int GetHashCode() => HashCode.Combine(Real, Imaginary);

    public override string ToString()
        => $"{Real}{(Imaginary >= 0 ? "+" : "")}{Imaginary}i";
}
```

## true/false Operator Overloading
```csharp
public struct BoolWrapper
{
    public bool Value { get; }

    public BoolWrapper(bool value) => Value = value;

    // Enables use in if, while, etc.
    public static bool operator true(BoolWrapper w) => w.Value;
    public static bool operator false(BoolWrapper w) => !w.Value;

    // Also enables && and || (via ?: rules)
    public static BoolWrapper operator &(BoolWrapper a, BoolWrapper b)
        => new(a.Value & b.Value);

    public static BoolWrapper operator |(BoolWrapper a, BoolWrapper b)
        => new(a.Value | b.Value);
}
```

## Conversion Operators

### Implicit Conversion (Safe, No Data Loss)
```csharp
public readonly struct Fraction
{
    public int Numerator { get; }
    public int Denominator { get; }

    public Fraction(int numerator, int denominator)
        => (Numerator, Denominator) = (numerator, denominator);

    // Implicit: from int to Fraction
    public static implicit operator Fraction(int value)
        => new(value, 1);
}

// Usage
Fraction f = 5;  // Implicit conversion
```

### Explicit Conversion (Potential Data Loss)
```csharp
public readonly struct Angle
{
    public double Radians { get; }

    public Angle(double radians) => Radians = radians;

    public static explicit operator double(Angle a) => a.Radians;

    // From degrees
    public static explicit operator Angle(double degrees)
        => new(degrees * Math.PI / 180.0);
}

// Usage
Angle a = (Angle)180.0;  // Explicit conversion
double rad = (double)a;  // Explicit conversion
```

## Equality Pattern (Required with ==/!=)
```csharp
public readonly struct Time : IEquatable<Time>
{
    public int Hours { get; }
    public int Minutes { get; }

    public Time(int hours, int minutes) => (Hours, Minutes) = (hours, minutes);

    // Operator pair
    public static bool operator ==(Time a, Time b)
        => a.Hours == b.Hours && a.Minutes == b.Minutes;

    public static bool operator !=(Time a, Time b) => !(a == b);

    // IEquatable<T>
    public bool Equals(Time other) => this == other;

    // object overrides
    public override bool Equals(object? obj) => obj is Time other && Equals(other);
    public override int GetHashCode() => HashCode.Combine(Hours, Minutes);
}
```

## Operator Overloading Guidelines
- Only overload when it's intuitive and expected
- Maintain the original operator semantics
- Always overload `==` and `!=` together
- Override `Equals` and `GetHashCode` with `==`/`!=`
- Provide `IEquatable<T>` for value types
- Keep conversion operators logical and expected
- Prefer methods when semantics are unclear
