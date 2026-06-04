# Delegates and Events

## Delegates

### Delegate Definition
A delegate is a type-safe function pointer - a reference to a method.

```csharp
// Declare a delegate type
public delegate void LogHandler(string message);

// Methods matching the delegate signature
public static void WriteToConsole(string msg)
    => Console.WriteLine($"Console: {msg}");

public static void WriteToFile(string msg)
    => File.AppendAllText("log.txt", $"{msg}\n");

// Usage
LogHandler logger = WriteToConsole;
logger("Hello World");  // Invokes WriteToConsole

logger = WriteToFile;
logger("Log entry");    // Now invokes WriteToFile
```

### Multicast Delegates
```csharp
LogHandler logger = WriteToConsole;
logger += WriteToFile;  // Add another target
logger("Test");         // Both methods called

logger -= WriteToConsole;  // Remove target
logger("Only file");       // Only WriteToFile called
```

### Built-in Delegate Types

#### Action (void return)
```csharp
Action greet = () => Console.WriteLine("Hello");
Action<string> log = msg => Console.WriteLine(msg);
Action<int, int> sum = (a, b) => Console.WriteLine(a + b);

// 0 to 16 parameters
Action action;
Action<T1> action1;
Action<T1, T2> action2;
// ... up to 16 type parameters
```

#### Func (has return value)
```csharp
Func<int> getNumber = () => 42;
Func<int, int> doubleIt = x => x * 2;
Func<int, int, int> add = (a, b) => a + b;

// Last type parameter is return type
Func<T1, TResult>     // 1 param + return
Func<T1, T2, TResult> // 2 params + return
```

#### Predicate (returns bool)
```csharp
Predicate<int> isPositive = x => x > 0;
Predicate<string> isEmpty = s => string.IsNullOrEmpty(s);

// Equivalent Func
Func<int, bool> same = x => x > 0;
```

### Anonymous Methods
```csharp
// Before lambdas (C# 2.0)
Func<int, int> square = delegate (int x) { return x * x; };

// Modern lambda equivalent
Func<int, int> square = x => x * x;
```

### Delegate as Parameter
```csharp
public List<T> Filter<T>(List<T> items, Func<T, bool> predicate)
{
    return items.Where(predicate).ToList();
}

// Usage
var numbers = new List<int> { 1, 2, 3, 4, 5 };
var evens = Filter(numbers, n => n % 2 == 0);
```

## Events

### Event Declaration and Usage
```csharp
public class Button
{
    // Event declaration (based on delegate)
    public event EventHandler? Clicked;
    public event EventHandler<KeyEventArgs>? KeyPressed;

    // Standard .NET event pattern
    public event EventHandler? TextChanged;

    protected virtual void OnClicked()
    {
        Clicked?.Invoke(this, EventArgs.Empty);
    }

    protected virtual void OnKeyPressed(char key)
    {
        KeyPressed?.Invoke(this, new KeyEventArgs(key));
    }

    // Simulate button press
    public void Press()
    {
        Console.WriteLine("Button pressed");
        OnClicked();
    }
}

// Custom EventArgs
public class KeyEventArgs : EventArgs
{
    public char Key { get; }
    public KeyEventArgs(char key) => Key = key;
}
```

### Subscribing to Events
```csharp
var button = new Button();

// Subscribe
button.Clicked += OnButtonClicked;
button.Clicked += (sender, e) => Console.WriteLine("Lambda handler");
button.Clicked += OnButtonClickedAsync;  // async void (use carefully)

// Unsubscribe
button.Clicked -= OnButtonClicked;

// Usage
button.Press();

// Handler methods
void OnButtonClicked(object? sender, EventArgs e)
{
    Console.WriteLine("Button was clicked!");
}

async void OnButtonClickedAsync(object? sender, EventArgs e)
{
    await Task.Delay(100);
    Console.WriteLine("Async handler");
}
```

### Custom Event Accessors
```csharp
public class CustomEventSource
{
    private EventHandler? _handlers;

    public event EventHandler? MyEvent
    {
        add
        {
            _handlers += value;
            Console.WriteLine("Handler added");
        }
        remove
        {
            _handlers -= value;
            Console.WriteLine("Handler removed");
        }
    }
}
```

### Event Best Practices
```csharp
public class ImprovedButton
{
    // Use EventHandler<T> for custom data
    public event EventHandler<ClickEventArgs>? Clicked;

    // Thread-safe invocation pattern
    protected virtual void OnClicked(ClickEventArgs e)
    {
        // Copy to local variable for thread safety
        var handler = Volatile.Read(ref Clicked);
        handler?.Invoke(this, e);
    }
}

// Make EventArgs subclass sealed with readonly properties
public sealed class ClickEventArgs : EventArgs
{
    public int X { get; }
    public int Y { get; }
    public ClickEventArgs(int x, int y) => (X, Y) = (x, y);
}
```

## Common .NET Event Patterns

### PropertyChange Notification
```csharp
public class ViewModel : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler? PropertyChanged;

    protected void OnPropertyChanged([CallerMemberName] string? propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    private string _name = string.Empty;
    public string Name
    {
        get => _name;
        set
        {
            if (_name != value)
            {
                _name = value;
                OnPropertyChanged();
            }
        }
    }
}
```

### Event vs Delegate
| Aspect | Event | Delegate |
|--------|-------|----------|
| Invocation | Only owner class can invoke | Any caller can invoke |
| External subscription | `+=`/`-=` only | `=` and assignment allowed |
| Interface members | Can declare | Cannot (use property) |
| Encapsulation | Protected | No special protection |
