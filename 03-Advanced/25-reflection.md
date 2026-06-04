# Reflection

## Getting Type Information

```csharp
// Three ways to get a Type
Type type1 = typeof(string);
Type type2 = "hello".GetType();
Type type3 = Type.GetType("System.String");

// Type properties
Console.WriteLine(type1.Name);          // String
Console.WriteLine(type1.FullName);      // System.String
Console.WriteLine(type1.Namespace);     // System
Console.WriteLine(type1.IsClass);       // true
Console.WriteLine(type1.IsValueType);   // false
Console.WriteLine(type1.IsAbstract);    // false
Console.WriteLine(type1.IsSealed);      // true
Console.WriteLine(type1.IsGenericType); // false
Console.WriteLine(type1.BaseType?.Name); // Object

// Assembly information
Assembly assembly = type1.Assembly;
Console.WriteLine(assembly.FullName);
```

## Inspecting Members

```csharp
public class Person
{
    public string Name { get; set; }
    private int _age;
    public const int MaxAge = 150;

    public Person() { }
    public Person(string name) => Name = name;

    public void Greet() => Console.WriteLine($"Hello, {Name}");
    private void InternalMethod() { }
    public static Person Create() => new();
}

// Reflection inspection
Type type = typeof(Person);

// Properties
foreach (var prop in type.GetProperties(BindingFlags.Public | BindingFlags.Instance))
{
    Console.WriteLine($"Property: {prop.Name}, Type: {prop.PropertyType.Name}");
}

// Methods
foreach (var method in type.GetMethods(BindingFlags.Public | BindingFlags.Instance))
{
    Console.WriteLine($"Method: {method.Name}, Return: {method.ReturnType.Name}");
}

// Fields
foreach (var field in type.GetFields(BindingFlags.Public | BindingFlags.Instance | BindingFlags.NonPublic))
{
    Console.WriteLine($"Field: {field.Name}, Type: {field.FieldType.Name}");
}

// Constructors
foreach (var ctor in type.GetConstructors())
{
    Console.WriteLine($"Constructor with {ctor.GetParameters().Length} params");
}
```

## Dynamic Invocation

### Creating Instances
```csharp
Type type = typeof(Person);

// Using Activator
Person? person1 = (Person?)Activator.CreateInstance(type);
Person? person2 = (Person?)Activator.CreateInstance(type, "Alice");

// With generic
Person person3 = Activator.CreateInstance<Person>();

// With constructor info
ConstructorInfo? ctor = type.GetConstructor(new[] { typeof(string) });
Person? person4 = (Person?)ctor?.Invoke(new object[] { "Bob" });
```

### Invoking Methods
```csharp
Person person = new("Charlie");
Type type = typeof(Person);

// Invoke public instance method
MethodInfo? greetMethod = type.GetMethod("Greet");
greetMethod?.Invoke(person, null);  // "Hello, Charlie"

// Invoke static method
MethodInfo? createMethod = type.GetMethod("Create", BindingFlags.Public | BindingFlags.Static);
Person? newPerson = (Person?)createMethod?.Invoke(null, null);
```

### Getting and Setting Properties
```csharp
Person person = new();
Type type = typeof(Person);

// Get property info
PropertyInfo? nameProperty = type.GetProperty("Name");

// Set value
nameProperty?.SetValue(person, "Diana");

// Get value
string? name = (string?)nameProperty?.GetValue(person);
Console.WriteLine(name);  // "Diana"
```

### Accessing Non-Public Members
```csharp
Person person = new();
Type type = typeof(Person);

// Private field
FieldInfo? ageField = type.GetField("_age",
    BindingFlags.NonPublic | BindingFlags.Instance);
ageField?.SetValue(person, 25);
int age = (int)ageField?.GetValue(person);

// Private method
MethodInfo? internalMethod = type.GetMethod("InternalMethod",
    BindingFlags.NonPublic | BindingFlags.Instance);
internalMethod?.Invoke(person, null);
```

## Working with Attributes
```csharp
Type type = typeof(MyClass);

// Check if attribute exists
bool hasAttr = Attribute.IsDefined(type, typeof(SerializableAttribute));

// Get attribute
var attr = type.GetCustomAttribute<SerializableAttribute>();

// Get all attributes
var attrs = type.GetCustomAttributes(true);

// Property attributes
PropertyInfo prop = type.GetProperty("Name");
var propAttrs = prop.GetCustomAttributes<JsonPropertyNameAttribute>();
```

## Generic Types with Reflection
```csharp
// Get open generic type
Type openList = typeof(List<>);

// Create closed generic type
Type stringList = openList.MakeGenericType(typeof(string));

// Create instance
var list = (List<string>)Activator.CreateInstance(stringList)!;
list.Add("Hello");

// Generic method
MethodInfo? method = typeof(MyClass).GetMethod("GenericMethod");
MethodInfo? closed = method?.MakeGenericMethod(typeof(int));
closed?.Invoke(obj, new object[] { 42 });
```

## Loading Assemblies
```csharp
// Load from file
Assembly assembly = Assembly.LoadFrom("MyLibrary.dll");

// Load by name
Assembly assembly = Assembly.Load("System.Text.Json");

// Get all types
Type[] types = assembly.GetExportedTypes();

// Create instance from assembly
Type? type = assembly.GetType("MyLibrary.MyClass");
var instance = Activator.CreateInstance(type);
```

## Performance Considerations
```csharp
// Reflection is slow; cache results

// Bad: Reflect every time
for (int i = 0; i < 10000; i++)
{
    var prop = typeof(Person).GetProperty("Name");
    prop.SetValue(person, "Name" + i);
}

// Good: Cache the PropertyInfo
var cachedProp = typeof(Person).GetProperty("Name");
for (int i = 0; i < 10000; i++)
{
    cachedProp.SetValue(person, "Name" + i);
}

// Best: Use delegates (compiled reflection)
var setter = CreateSetter<Person, string>(cachedProp);
for (int i = 0; i < 10000; i++)
{
    setter(person, "Name" + i);
}

static Action<T, TValue> CreateSetter<T, TValue>(PropertyInfo prop)
{
    var method = prop.GetSetMethod();
    return (Action<T, TValue>)Delegate.CreateDelegate(typeof(Action<T, TValue>), method);
}
```

## Common Reflection Uses

### Object Mapper
```csharp
public static TTarget Map<TSource, TTarget>(TSource source)
    where TTarget : new()
{
    var target = new TTarget();
    var sourceProps = typeof(TSource).GetProperties();
    var targetProps = typeof(TTarget).GetProperties();

    foreach (var sourceProp in sourceProps)
    {
        var targetProp = targetProps.FirstOrDefault(p =>
            p.Name == sourceProp.Name && p.PropertyType == sourceProp.PropertyType);
        if (targetProp is not null && targetProp.CanWrite)
        {
            targetProp.SetValue(target, sourceProp.GetValue(source));
        }
    }
    return target;
}
```

### Plugin System
```csharp
public IEnumerable<IPlugin> LoadPlugins(string directory)
{
    var plugins = new List<IPlugin>();
    foreach (var dll in Directory.GetFiles(directory, "*.dll"))
    {
        var assembly = Assembly.LoadFrom(dll);
        var pluginTypes = assembly.GetTypes()
            .Where(t => typeof(IPlugin).IsAssignableFrom(t) && !t.IsInterface);
        foreach (var type in pluginTypes)
        {
            plugins.Add((IPlugin)Activator.CreateInstance(type));
        }
    }
    return plugins;
}
```
