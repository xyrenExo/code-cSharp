# LINQ (Language Integrated Query)

## Introduction
LINQ provides a consistent way to query data from various sources (collections, databases, XML, etc.).

## Two Syntaxes

### Method Syntax (Fluent)
```csharp
var result = numbers
    .Where(n => n > 5)
    .OrderBy(n => n)
    .Select(n => n * 2);
```

### Query Syntax (SQL-like)
```csharp
var result = from n in numbers
             where n > 5
             orderby n
             select n * 2;

// Both produce the same result; method syntax is more common
```

## Filtering

### Where
```csharp
var adults = people.Where(p => p.Age >= 18);

// Indexed filter
var everyOther = people.Where((p, index) => index % 2 == 0);

// OfType - filter by type
var strings = mixedList.OfType<string>();
```

### Distinct
```csharp
var unique = numbers.Distinct();
var uniqueByAge = people.DistinctBy(p => p.Age);  // .NET 6+
```

## Projection

### Select
```csharp
var names = people.Select(p => p.Name);
var anon = people.Select(p => new { p.Name, p.Age });
var indexed = people.Select((p, i) => $"{i}: {p.Name}");

// SelectMany (flatten nested collections)
var allOrders = customers.SelectMany(c => c.Orders);
var words = sentences.SelectMany(s => s.Split(' '));
```

### SelectMany
```csharp
// Flatten nested collections
var students = new[]
{
    new Student("Alice", new[] { "Math", "Physics" }),
    new Student("Bob", new[] { "English", "History" })
};

var allSubjects = students.SelectMany(s => s.Subjects);
// Result: "Math", "Physics", "English", "History"

// With parent element
var studentSubjects = students.SelectMany(
    s => s.Subjects,
    (student, subject) => $"{student.Name}: {subject}"
);
```

## Sorting

```csharp
var sorted = people.OrderBy(p => p.Name);
var sortedDesc = people.OrderByDescending(p => p.Age);
var multiSort = people
    .OrderBy(p => p.Department)
    .ThenBy(p => p.Name);

var reversed = numbers.Reverse();
```

## Partitioning

```csharp
var first3 = numbers.Take(3);
var skip2 = numbers.Skip(2);
var page = numbers.Skip(10).Take(10);

// Predicate-based
var until = numbers.TakeWhile(n => n < 10);
var after = numbers.SkipWhile(n => n < 10);
```

## Aggregation

```csharp
int count = numbers.Count();
int sum = numbers.Sum();
double avg = numbers.Average();
int min = numbers.Min();
int max = numbers.Max();

// With selector
int totalAge = people.Sum(p => p.Age);
double avgAge = people.Average(p => p.Age);

// Custom aggregation
var product = numbers.Aggregate((acc, n) => acc * n);
var csv = numbers.Aggregate("", (acc, n) => $"{acc},{n}").TrimStart(',');
```

## Quantifiers

```csharp
bool any = numbers.Any();              // Any elements?
bool anyEven = numbers.Any(n => n % 2 == 0);
bool allEven = numbers.All(n => n % 2 == 0);
bool contains = numbers.Contains(5);
```

## Set Operations

```csharp
var union = set1.Union(set2);
var intersect = set1.Intersect(set2);
var except = set1.Except(set2);
var concat = set1.Concat(set2);  // Duplicates kept

// With comparers
var union = set1.Union(set2, new PersonComparer());
```

## Element Operations

```csharp
int first = numbers.First();          // Throws if empty
int? firstOrNull = numbers.FirstOrDefault();
int last = numbers.Last();
int? lastOrNull = numbers.LastOrDefault();
int single = numbers.Single(n => n == 5);  // Exactly one match
int? singleOrNull = numbers.SingleOrDefault(n => n == 5);

// With defaults
int result = numbers.FirstOrDefault(-1);  // -1 if empty (.NET 6+)
```

## Grouping

```csharp
var grouped = people.GroupBy(p => p.Department);

foreach (var group in grouped)
{
    Console.WriteLine($"Dept: {group.Key}, Count: {group.Count()}");
    foreach (var person in group)
        Console.WriteLine($"  {person.Name}");
}

// With element projection
var summary = people
    .GroupBy(p => p.Department)
    .Select(g => new
    {
        Department = g.Key,
        Count = g.Count(),
        AverageAge = g.Average(p => p.Age)
    });

// GroupBy with multiple keys
var grouped = people.GroupBy(p => new { p.Department, p.City });
```

## Joining

```csharp
var orders = new List<Order>();
var customers = new List<Customer>();

// Inner join
var query = customers.Join(
    orders,
    customer => customer.Id,
    order => order.CustomerId,
    (customer, order) => new { customer.Name, order.Total }
);

// Group join (left join)
var customerOrders = customers.GroupJoin(
    orders,
    customer => customer.Id,
    order => order.CustomerId,
    (customer, customerOrders) => new
    {
        customer.Name,
        OrderCount = customerOrders.Count()
    }
);

// Zip
var numbers = new[] { 1, 2, 3 };
var words = new[] { "one", "two", "three" };
var zipped = numbers.Zip(words, (n, w) => $"{n}: {w}");
// "1: one", "2: two", "3: three"
```

## LINQ to Objects vs Other Providers

```csharp
// LINQ to Objects (in-memory collections)
var result = list.Where(x => x > 5);

// LINQ to SQL / EF Core (database)
var users = db.Users.Where(u => u.Age > 18);  // Translated to SQL

// LINQ to XML
var elements = doc.Descendants("item")
    .Where(x => x.Attribute("id").Value == "123");

// Parallel LINQ (PLINQ)
var result = list.AsParallel().Where(x => ExpensiveCheck(x));
```

## Deferred vs Immediate Execution

```csharp
// Deferred (lazy) - query defined, not executed
var query = numbers.Where(n => n > 5);

// Execution when enumerated
foreach (var n in query) { }
var list = query.ToList();
var array = query.ToArray();

// Immediate execution
var count = query.Count();
var first = query.First();
var any = query.Any();

// Changes to source affect deferred queries
numbers.Add(10);  // query would include 10
```

## Conversion Methods

```csharp
var list = query.ToList();
var array = query.ToArray();
var dict = people.ToDictionary(p => p.Id);
var lookup = people.ToLookup(p => p.Department);
var hashSet = query.ToHashSet();
```

## LINQ with Empty Sequences

```csharp
var empty = Enumerable.Empty<int>();
var range = Enumerable.Range(1, 10);      // 1, 2, ..., 10
var repeat = Enumerable.Repeat("A", 5);    // "A", "A", ...
```

## Query Expression Full Examples
```csharp
var query = from p in people
            where p.Age >= 18 && p.City == "New York"
            orderby p.Name descending
            select p;

var joinQuery = from c in customers
                join o in orders on c.Id equals o.CustomerId
                select new { c.Name, o.Total };

var groupQuery = from p in people
                 group p by p.Department into deptGroup
                 select new
                 {
                     Department = deptGroup.Key,
                     Count = deptGroup.Count()
                 };
```
