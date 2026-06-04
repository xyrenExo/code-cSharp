# Structs vs Classes

## Key Differences

| Feature | Class (Reference Type) | Struct (Value Type) |
|---------|----------------------|-------------------|
| Memory | Heap (GC-managed) | Stack/inline |
| Assignment | Copies reference | Copies value |
| Default value | `null` | All fields zeroed |
| Inheritance | Single class + multiple interfaces | Only interfaces |
| Constructor | Can define any constructor | Must set all fields (or use primary) |
| Destructor | Yes | No |
| Parameterless constructor | Default provided | C# 10+: can define |
| `this` | Reference type | Value type |
| Mutability | Usually mutable | Should be immutable |
| Performance | Allocation + GC cost | Stack allocation |

## Class Example
```csharp
public class PointClass
{
    public int X { get; set; }
    public int Y { get; set; }

    public PointClass(int x, int y) => (X, Y) = (x, y);
}

// Reference semantics
var p1 = new PointClass(1, 2);
var p2 = p1;          // Both point to SAME object
p2.X = 100;
Console.WriteLine(p1.X);  // 100 (changed!)
```

## Struct Example
```csharp
public struct PointStruct
{
    public int X { get; set; }
    public int Y { get; set; }

    public PointStruct(int x, int y) => (X, Y) = (x, y);
}

// Value semantics
var p1 = new PointStruct(1, 2);
var p2 = p1;          // Independent COPY
p2.X = 100;
Console.WriteLine(p1.X);  // 1 (unchanged!)
```

## Readonly Struct (C# 7.2+)
```csharp
public readonly struct Coordinate
{
    // All fields MUST be readonly
    public double Latitude { get; }
    public double Longitude { get; }

    public Coordinate(double lat, double lon) => (Latitude, Longitude) = (lat, lon);

    // Cannot have property setters
    // Cannot have field modifiers other than readonly
    // Compiler can optimize - no defensive copies
}
```

## Ref Struct (C# 7.2+)
```csharp
public ref struct SpanWrapper
{
    private readonly Span<byte> _buffer;

    public SpanWrapper(Span<byte> buffer) => _buffer = buffer;

    // Ref structs:
    // - Cannot be boxed
    // - Cannot be a field of a class
    // - Cannot be used in async methods
    // - Cannot be used in iterators
    // - Can only be on stack
}
```

## Record Struct (C# 10+)
```csharp
public readonly record struct Measurement(double Value, string Unit);

// Value-based equality
var m1 = new Measurement(10, "kg");
var m2 = new Measurement(10, "kg");
Console.WriteLine(m1 == m2);  // true

// With deconstruction
var (value, unit) = m1;

// Non-destructive mutation
var m3 = m1 with { Value = 20 };
```

## When to Use Struct

### Good Candidates for Struct
```csharp
// Small, immutable data (≤ 16 bytes)
public readonly struct PhoneNumber
{
    public string CountryCode { get; }
    public string Number { get; }

    public PhoneNumber(string countryCode, string number)
        => (CountryCode, Number) = (countryCode, number);
}

// Frequently allocated in arrays/lists
public readonly struct RgbColor
{
    public byte R { get; }
    public byte G { get; }
    public byte B { get; }

    public RgbColor(byte r, byte g, byte b) => (R, G, B) = (r, g, b);
}
```

### Poor Candidates for Struct
```csharp
// Too large (> 16 bytes) - expensive to copy
public struct LargeData  // BAD for struct
{
    public string Name;
    public string Description;
    public decimal Price;
    public DateTime CreatedAt;
    public List<string> Tags;
}

// Mutating state
public struct MutableStruct  // BAD - mutable structs cause bugs
{
    public int Value;
    public void Increment() => Value++;  // This modifies 'this' copy!
}
```

## Performance Considerations

### Stack vs Heap
```csharp
// Struct - stack allocated
void ProcessPoint()
{
    var point = new PointStruct(10, 20);  // On stack
    // No GC pressure
}

// Class - heap allocated
void ProcessPointClass()
{
    var point = new PointClass(10, 20);  // On heap
    // Creates GC pressure
}
```

### Array Performance
```csharp
// Struct array: contiguous memory, cache-friendly
PointStruct[] structArray = new PointStruct[1000];

// Class array: scattered objects, pointer chasing
PointClass[] classArray = new PointClass[1000];
for (int i = 0; i < 1000; i++)
    classArray[i] = new PointClass(i, i);
```

## Boxing/Unboxing
```csharp
// Struct boxing (avoids when possible)
int i = 42;
object boxed = i;         // Boxing: allocates on heap
int unboxed = (int)boxed; // Unboxing

// Using generic collections avoids boxing
List<int> numbers = new();    // No boxing
ArrayList oldList = new();    // Boxing! Avoid
```

## Guidelines

### Choose Struct When:
- Represents a single value (like a number, point, color)
- Small size (≤ 16 bytes recommended)
- Immutable
- Frequently created and short-lived
- Doesn't need polymorphism
- Used in arrays (better locality)

### Choose Class When:
- Larger than 16 bytes
- Needs inheritance/polymorphism
- Needs identity (reference equality)
- Needs mutable state with shared references
- Needs nullability as a reference
- Needs destructor/finalizer
