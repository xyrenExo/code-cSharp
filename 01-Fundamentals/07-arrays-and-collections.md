# Arrays and Collections

## Arrays

### Single-Dimensional Arrays
```csharp
// Declaration and initialization
int[] numbers = new int[5];
int[] numbers = new int[] { 1, 2, 3, 4, 5 };
int[] numbers = { 1, 2, 3, 4, 5 };

// Access and modify
numbers[0] = 10;
int first = numbers[0];
int length = numbers.Length;

// Iteration
for (int i = 0; i < numbers.Length; i++)
    Console.Write(numbers[i]);

foreach (var num in numbers)
    Console.Write(num);
```

### Multi-Dimensional Arrays
```csharp
// Rectangular arrays (fixed dimensions)
int[,] matrix = new int[3, 4];
int[,] matrix = { { 1, 2 }, { 3, 4 }, { 5, 6 } };
int value = matrix[1, 0];        // 3
int rows = matrix.GetLength(0);  // 3
int cols = matrix.GetLength(1);  // 2

// 3D array
int[,,] cube = new int[3, 3, 3];
```

### Jagged Arrays (Array of Arrays)
```csharp
int[][] jagged = new int[3][];
jagged[0] = new int[] { 1, 2 };
jagged[1] = new int[] { 3, 4, 5 };
jagged[2] = new int[] { 6 };

int value = jagged[1][2];  // 5
```

### Array Methods
```csharp
int[] arr = { 5, 2, 8, 1, 9 };
Array.Sort(arr);                      // 1, 2, 5, 8, 9
Array.Reverse(arr);                   // 9, 8, 5, 2, 1
int index = Array.IndexOf(arr, 8);    // 1
bool exists = Array.Exists(arr, x => x > 5);  // true
int found = Array.Find(arr, x => x > 5);      // 8
int[] slice = arr[1..3];              // Range operator (C# 8+)
```

## Generic Collections

### List<T>
```csharp
List<string> list = new List<string>();
list.Add("Apple");
list.AddRange(new[] { "Banana", "Cherry" });
list.Insert(1, "Blueberry");
list.Remove("Apple");
list.RemoveAt(0);
list.Sort();
bool has = list.Contains("Banana");
int count = list.Count;
string first = list[0];
string last = list[^1];             // Index from end (C# 8+)
```

### Dictionary<TKey, TValue>
```csharp
var dict = new Dictionary<string, int>();
dict.Add("Alice", 25);
dict["Bob"] = 30;

if (dict.TryGetValue("Alice", out int age))
    Console.WriteLine(age);

foreach (var kvp in dict)
    Console.WriteLine($"{kvp.Key}: {kvp.Value}");

foreach (var key in dict.Keys) { }
foreach (var value in dict.Values) { }
```

### HashSet<T>
```csharp
var set = new HashSet<int> { 1, 2, 3, 3, 4 };  // { 1, 2, 3, 4 }
set.Add(5);
bool added = set.Add(3);   // false (already exists)
bool contains = set.Contains(2);  // true
set.UnionWith(otherSet);
set.IntersectWith(otherSet);
set.ExceptWith(otherSet);
```

### Queue<T>
```csharp
var queue = new Queue<string>();
queue.Enqueue("First");
queue.Enqueue("Second");
string item = queue.Dequeue();      // "First"
string peek = queue.Peek();         // "Second" (without removing)
```

### Stack<T>
```csharp
var stack = new Stack<string>();
stack.Push("Bottom");
stack.Push("Top");
string item = stack.Pop();          // "Top"
string peek = stack.Peek();         // "Bottom"
```

### LinkedList<T>
```csharp
var linked = new LinkedList<string>();
linked.AddLast("Last");
linked.AddFirst("First");
var node = linked.Find("First");
linked.AddAfter(node, "Middle");
```

## Collection Performance

| Collection | Access | Search | Insert | Delete | Memory |
|-----------|--------|--------|--------|--------|--------|
| `T[]` | O(1) | O(n) | O(n) | O(n) | Low |
| `List<T>` | O(1) | O(n) | O(n) | O(n) | Low |
| `Dictionary<K,V>` | O(1) | O(1) | O(1) | O(1) | High |
| `HashSet<T>` | - | O(1) | O(1) | O(1) | High |
| `Queue<T>` | O(1) | O(n) | O(1) | O(1) | Low |
| `Stack<T>` | O(1) | O(n) | O(1) | O(1) | Low |
| `LinkedList<T>` | O(n) | O(n) | O(1) | O(1) | Medium |

## System.Array and Collection Interfaces

```csharp
// Interfaces
IEnumerable<T>    // Foreach support
ICollection<T>    // Add, Remove, Count
IList<T>          // Indexed access
IDictionary<K,V>  // Key-value access
ISet<T>           // Set operations
IReadOnlyList<T>  // Read-only indexed access
IReadOnlyDictionary<K,V> // Read-only dict access
```

## Immutable Collections
```csharp
using System.Collections.Immutable;

ImmutableArray<int> arr = ImmutableArray.Create(1, 2, 3);
ImmutableList<int> list = ImmutableList.Create(1, 2, 3);
ImmutableDictionary<string, int> dict = ImmutableDictionary.Create<string, int>();
```
