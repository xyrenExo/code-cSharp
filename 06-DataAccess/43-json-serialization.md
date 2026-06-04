# JSON Serialization

## System.Text.Json (Modern)

### Basic Serialization/Deserialization
```csharp
using System.Text.Json;

// Serialize
string json = JsonSerializer.Serialize(person);
string prettyJson = JsonSerializer.Serialize(person, new JsonSerializerOptions
{
    WriteIndented = true
});

// Deserialize
var person = JsonSerializer.Deserialize<Person>(json);
var person = JsonSerializer.Deserialize<Person>(json, new JsonSerializerOptions
{
    PropertyNameCaseInsensitive = true
});
```

### Async
```csharp
// Serialize to stream
await JsonSerializer.SerializeAsync(stream, person, options);

// Deserialize from stream
var person = await JsonSerializer.DeserializeAsync<Person>(stream, options);
```

### JsonSerializerOptions
```csharp
var options = new JsonSerializerOptions
{
    WriteIndented = true,
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    PropertyNameCaseInsensitive = true,
    DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull,
    IncludeFields = true,
    IgnoreReadOnlyProperties = true,
    NumberHandling = JsonNumberHandling.AllowReadingFromString,
    ReferenceHandler = ReferenceHandler.IgnoreCycles,
    Encoder = System.Text.Encodings.Web.JavaScriptEncoder.UnsafeRelaxedJsonEscaping
};
```

### Attributes
```csharp
public class Person
{
    [JsonPropertyName("person_name")]
    public string Name { get; set; }

    [JsonIgnore]
    public string InternalField { get; set; }

    [JsonInclude]
    internal string InternalProperty { get; set; }

    [JsonConverter(typeof(CustomDateConverter))]
    public DateTime BirthDate { get; set; }

    [JsonNumberHandling(JsonNumberHandling.AllowReadingFromString)]
    public int Age { get; set; }
}
```

### Custom Converters
```csharp
public class CustomDateConverter : JsonConverter<DateTime>
{
    private readonly string _format = "yyyy-MM-dd";

    public override DateTime Read(ref Utf8JsonReader reader, Type typeToConvert,
        JsonSerializerOptions options)
    {
        return DateTime.ParseExact(reader.GetString()!, _format, null);
    }

    public override void Write(Utf8JsonWriter writer, DateTime value,
        JsonSerializerOptions options)
    {
        writer.WriteStringValue(value.ToString(_format));
    }
}
```

### Polymorphic Serialization
```csharp
[JsonDerivedType(typeof(Cat), nameof(Cat))]
[JsonDerivedType(typeof(Dog), nameof(Dog))]
public class Animal { }

public class Cat : Animal { public bool IsIndoor { get; set; } }
public class Dog : Animal { public string Breed { get; set; } }

// Serialization includes type discriminator
var json = JsonSerializer.Serialize<Animal>(new Dog { Breed = "Lab" });
// { "$type": "Dog", "Breed": "Lab" }
```

## Newtonsoft.Json (Json.NET)

### Setup
```bash
dotnet add package Newtonsoft.Json
```

### Basic Usage
```csharp
using Newtonsoft.Json;

// Serialize
string json = JsonConvert.SerializeObject(person, Formatting.Indented);

// Deserialize
var person = JsonConvert.DeserializeObject<Person>(json);
```

### Settings
```csharp
var settings = new JsonSerializerSettings
{
    Formatting = Formatting.Indented,
    NullValueHandling = NullValueHandling.Ignore,
    MissingMemberHandling = MissingMemberHandling.Error,
    ReferenceLoopHandling = ReferenceLoopHandling.Ignore,
    ContractResolver = new CamelCasePropertyNamesContractResolver(),
    DateFormatString = "yyyy-MM-dd",
    Converters = new List<JsonConverter> { new StringEnumConverter() }
};
```

### Attributes
```csharp
public class Person
{
    [JsonProperty("person_name")]
    public string Name { get; set; }

    [JsonIgnore]
    public string InternalField { get; set; }

    [JsonRequired]
    public string Email { get; set; }

    [JsonConverter(typeof(IsoDateTimeConverter))]
    public DateTime BirthDate { get; set; }

    [OnDeserialized]
    internal void OnDeserialized(StreamingContext context)
    {
        // Post-deserialization logic
    }
}
```

## JSON Documents

### JsonDocument (System.Text.Json)
```csharp
using System.Text.Json;

string json = @"{ ""name"": ""Alice"", ""age"": 30 }";

using JsonDocument doc = JsonDocument.Parse(json);
JsonElement root = doc.RootElement;

string name = root.GetProperty("name").GetString()!;
int age = root.GetProperty("age").GetInt32();

// Iterate
foreach (var property in root.EnumerateObject())
{
    Console.WriteLine($"{property.Name}: {property.Value}");
}

// Array
JsonElement items = root.GetProperty("items");
foreach (var item in items.EnumerateArray())
{
    Console.WriteLine(item.GetString());
}
```

### JsonNode (System.Text.Json, .NET 6+)
```csharp
// Mutable JSON
dynamic json = new { name = "Alice", age = 30 };
JsonNode node = JsonSerializer.SerializeToNode(json);
node["age"] = 31;
node["email"] = "alice@example.com";

string jsonString = node.ToJsonString(new JsonSerializerOptions
{
    WriteIndented = true
});
```

## Source Generators (.NET 6+)

```csharp
// Performance optimization - no runtime reflection
[JsonSerializable(typeof(Person))]
[JsonSerializable(typeof(List<Person>))]
public partial class JsonContext : JsonSerializerContext
{
}

// Usage
string json = JsonSerializer.Serialize(person, JsonContext.Default.Person);
var person = JsonSerializer.Deserialize(json, JsonContext.Default.Person);
```

## HttpClient Extensions

```csharp
// System.Text.Json
var user = await httpClient.GetFromJsonAsync<User>("api/users/1");
await httpClient.PostAsJsonAsync("api/users", user);
await httpClient.PutAsJsonAsync("api/users/1", user);

// With options
var options = new JsonSerializerOptions
{
    PropertyNameCaseInsensitive = true
};
var user = await httpClient.GetFromJsonAsync<User>("api/users/1", options);
```
