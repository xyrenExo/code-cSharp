# Inheritance

## Basic Inheritance
```csharp
// Base class (parent)
public class Animal
{
    public string Name { get; set; }
    public int Age { get; set; }

    public Animal(string name)
    {
        Name = name;
    }

    public void Eat()
    {
        Console.WriteLine($"{Name} is eating.");
    }

    public virtual void MakeSound()
    {
        Console.WriteLine("Animal makes a sound");
    }
}

// Derived class (child)
public class Dog : Animal
{
    public Dog(string name) : base(name) { }

    public void Bark()
    {
        Console.WriteLine("Woof!");
    }

    public override void MakeSound()
    {
        Console.WriteLine("Woof! Woof!");
    }
}
```

## `base` Keyword
```csharp
public class Vehicle
{
    public string Brand { get; }
    public int Year { get; }

    public Vehicle(string brand, int year)
    {
        Brand = brand;
        Year = year;
    }

    public virtual void DisplayInfo()
    {
        Console.WriteLine($"{Brand} ({Year})");
    }
}

public class Car : Vehicle
{
    public int Doors { get; }

    public Car(string brand, int year, int doors) 
        : base(brand, year)  // Call base constructor
    {
        Doors = doors;
    }

    public override void DisplayInfo()
    {
        base.DisplayInfo();  // Call base method
        Console.WriteLine($"Doors: {Doors}");
    }
}
```

## Method Hiding (new keyword)
```csharp
public class Base
{
    public void Display()
    {
        Console.WriteLine("Base Display");
    }
}

public class Derived : Base
{
    public new void Display()  // Hides base method intentionally
    {
        Console.WriteLine("Derived Display");
    }
}

// Usage
Base obj = new Derived();
obj.Display();           // "Base Display" (base version)

Derived derived = new Derived();
derived.Display();       // "Derived Display" (derived version)
```

## Sealed Classes and Methods
```csharp
public sealed class FinalClass
{
    // Cannot be inherited
}

public class BaseClass
{
    public virtual void Method() { }
}

public class DerivedClass : BaseClass
{
    public sealed override void Method()  // Cannot be overridden further
    {
        base.Method();
    }
}
```

## Multiple Inheritance (via Interfaces)
C# does not support multiple class inheritance, but supports multiple interface inheritance:
```csharp
public interface IWalkable
{
    void Walk();
}

public interface ISwimmable
{
    void Swim();
}

public class Duck : IWalkable, ISwimmable
{
    public void Walk() => Console.WriteLine("Duck walks");
    public void Swim() => Console.WriteLine("Duck swims");
}
```

## Constructor Inheritance Chain
```csharp
public class A
{
    public A() => Console.WriteLine("A");
}

public class B : A
{
    public B() => Console.WriteLine("B");
}

public class C : B
{
    public C() => Console.WriteLine("C");
}

// new C() outputs:
// A
// B
// C
```

## Polymorphism with Inheritance
```csharp
List<Animal> animals = new()
{
    new Dog("Rex"),
    new Cat("Whiskers"),
    new Animal("Generic")
};

foreach (var animal in animals)
{
    animal.MakeSound();  // Calls appropriate override
}
// Output:
// Woof! Woof!
// Meow!
// Animal makes a sound
```

## is and as with Inheritance
```csharp
Animal animal = new Dog("Buddy");

if (animal is Dog dog)
{
    dog.Bark();  // Safe downcast
}

Dog? anotherDog = animal as Dog;
if (anotherDog is not null)
{
    anotherDog.Bark();
}
```

## Inheritance and Access Modifiers
```csharp
public class Base
{
    public int Public;          // Everyone
    private int Private;        // Only Base
    protected int Protected;    // Base + Derived
    internal int Internal;      // Same assembly
    protected internal int ProtectedInternal; // Same assembly + Derived
    private protected int PrivateProtected;  // Same assembly derived classes only
}
```

## Inheritance Rules
- A class can only inherit from one base class
- Inheritance is transitive (C inherits from B inherits from A)
- Constructors are not inherited
- `static` classes cannot be inherited
- `struct` cannot inherit from classes (except `ValueType`)
- All classes implicitly inherit from `object`

## Virtual Member Inheritance
```csharp
public class Shape
{
    // Virtual - can be overridden
    public virtual double Area() => 0;

    // Abstract - must be overridden (class must also be abstract)
    // public abstract double Area();

    // Non-virtual - cannot be overridden
    public string GetName() => "Shape";
}
```
