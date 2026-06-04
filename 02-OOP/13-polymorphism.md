# Polymorphism

## What is Polymorphism?
Polymorphism ("many forms") allows objects of different types to be treated as objects of a common base type, with each type providing its own implementation of shared methods.

## Compile-Time Polymorphism (Method Overloading)

### Method Overloading
```csharp
public class Calculator
{
    // Same name, different parameters
    public int Add(int a, int b) => a + b;

    public int Add(int a, int b, int c) => a + b + c;

    public double Add(double a, double b) => a + b;

    public decimal Add(decimal a, decimal b) => a + b;

    // Different parameter types
    public string Add(string a, string b) => $"{a}{b}";
}
```

### Operator Overloading
```csharp
public struct Point
{
    public int X { get; }
    public int Y { get; }

    public Point(int x, int y) => (X, Y) = (x, y);

    public static Point operator +(Point a, Point b)
        => new Point(a.X + b.X, a.Y + b.Y);

    public static Point operator -(Point a, Point b)
        => new Point(a.X - b.X, a.Y - b.Y);

    public static bool operator ==(Point a, Point b)
        => a.X == b.X && a.Y == b.Y;

    public static bool operator !=(Point a, Point b)
        => !(a == b);

    public override bool Equals(object obj)
        => obj is Point p && this == p;

    public override int GetHashCode() => HashCode.Combine(X, Y);
}
```

## Runtime Polymorphism (Method Overriding)

### Virtual and Override
```csharp
public class Employee
{
    public string Name { get; set; }

    public virtual decimal CalculatePay()
    {
        return 0;
    }

    public virtual void Work()
    {
        Console.WriteLine($"{Name} is working");
    }
}

public class SalariedEmployee : Employee
{
    public decimal Salary { get; set; }

    public override decimal CalculatePay()
    {
        return Salary / 12;  // Monthly salary
    }
}

public class HourlyEmployee : Employee
{
    public decimal HourlyRate { get; set; }
    public int HoursWorked { get; set; }

    public override decimal CalculatePay()
    {
        return HourlyRate * HoursWorked;
    }

    public override void Work()
    {
        Console.WriteLine($"{Name} is working hourly shift");
    }
}
```

### Polymorphic Behavior
```csharp
List<Employee> employees = new()
{
    new SalariedEmployee { Name = "Alice", Salary = 60000 },
    new HourlyEmployee { Name = "Bob", HourlyRate = 25, HoursWorked = 80 }
};

foreach (var employee in employees)
{
    employee.Work();  // Calls appropriate override
    Console.WriteLine($"Pay: {employee.CalculatePay():C}");
}
```

## Abstract Classes and Polymorphism
```csharp
public abstract class Shape
{
    public abstract double Area { get; }
    public abstract double Perimeter { get; }

    public virtual void Display()
    {
        Console.WriteLine($"Area: {Area}, Perimeter: {Perimeter}");
    }
}

public class Circle : Shape
{
    public double Radius { get; set; }

    public override double Area => Math.PI * Radius * Radius;
    public override double Perimeter => 2 * Math.PI * Radius;
}

public class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }

    public override double Area => Width * Height;
    public override double Perimeter => 2 * (Width + Height);
}
```

## Interface Polymorphism
```csharp
public interface ILogger
{
    void Log(string message);
}

public class ConsoleLogger : ILogger
{
    public void Log(string message)
        => Console.WriteLine($"Console: {message}");
}

public class FileLogger : ILogger
{
    public void Log(string message)
        => File.AppendAllText("log.txt", $"{message}\n");
}

public class Application
{
    private readonly ILogger _logger;

    public Application(ILogger logger)
    {
        _logger = logger;  // Any ILogger implementation
    }

    public void Run()
    {
        _logger.Log("Application started");
    }
}
```

## Polymorphism with Generics
```csharp
public interface IRepository<T>
{
    T GetById(int id);
    void Save(T entity);
}

public class UserRepository : IRepository<User>
{
    public User GetById(int id) { /* ... */ }
    public void Save(User entity) { /* ... */ }
}

// Covariance (out)
IEnumerable<object> objects = new List<string>();

// Contravariance (in)
Action<string> stringAction = (string s) => { };
Action<object> objectAction = (object o) => { };
stringAction = (Action<string>)objectAction;  // Contravariance
```

## Sealed Methods (Stopping Polymorphism)
```csharp
public class Base
{
    public virtual void DoSomething() { }
}

public class Derived : Base
{
    public sealed override void DoSomething()  // Stops further override
    {
        base.DoSomething();
    }
}

public class MoreDerived : Derived
{
    // Cannot override DoSomething - it's sealed
}
```

## Summary
| Type | When to Use |
|------|-------------|
| Method Overloading | Same operation, different inputs |
| Virtual/Override | Runtime polymorphic behavior |
| Abstract | Base class with incomplete implementation |
| Interface | Contract-based polymorphism |
| Generics | Type-parameterized polymorphism |
| Operator Overloading | Custom type math/comparison |
