# Input/Output Operations

## Console I/O

### Basic Console I/O
```csharp
// Output
Console.Write("Enter your name: ");
Console.WriteLine($"Hello, {name}!");

// Input
string name = Console.ReadLine();
int key = Console.Read();               // Reads single character code
ConsoleKeyInfo keyInfo = Console.ReadKey();  // Reads key press

// Formatted output
Console.WriteLine("Name: {0}, Age: {1}", name, age);
Console.WriteLine($"Name: {name}, Age: {age}");
```

### Console Color and Positioning
```csharp
Console.ForegroundColor = ConsoleColor.Green;
Console.BackgroundColor = ConsoleColor.Black;
Console.WriteLine("Green text");

Console.CursorLeft = 10;
Console.CursorTop = 5;
Console.Write("Positioned text");

Console.Clear();
Console.Beep();  // Plays beep sound
```

## File I/O

### Reading Files
```csharp
// Read all at once
string content = File.ReadAllText("file.txt");
string[] lines = File.ReadAllLines("file.txt");
byte[] bytes = File.ReadAllBytes("file.bin");

// Read line by line (efficient for large files)
foreach (string line in File.ReadLines("file.txt"))
{
    Process(line);
}

// Using StreamReader
using (var reader = new StreamReader("file.txt"))
{
    string line;
    while ((line = reader.ReadLine()) is not null)
    {
        Process(line);
    }
}
```

### Writing Files
```csharp
// Write all at once
File.WriteAllText("file.txt", "content");
File.WriteAllLines("file.txt", new[] { "line1", "line2" });
File.WriteAllBytes("file.bin", byteArray);

// Append
File.AppendAllText("file.txt", "additional content");
File.AppendAllLines("file.txt", new[] { "line1", "line2" });

// Using StreamWriter
using (var writer = new StreamWriter("file.txt", append: true))
{
    writer.WriteLine("New line");
    writer.Write("Text without newline");
}
```

### File and Directory Operations
```csharp
// File operations
bool exists = File.Exists("file.txt");
File.Copy("source.txt", "dest.txt", overwrite: true);
File.Move("source.txt", "dest.txt");
File.Delete("file.txt");
FileInfo info = new FileInfo("file.txt");
long size = info.Length;
DateTime modified = info.LastWriteTime;

// Directory operations
Directory.CreateDirectory(@"C:\Data\SubDir");
bool dirExists = Directory.Exists(@"C:\Data");
string[] files = Directory.GetFiles(@"C:\Data", "*.txt");
string[] dirs = Directory.GetDirectories(@"C:\Data");
Directory.Delete(@"C:\Data", recursive: true);

// Enumerate (better performance for large directories)
foreach (string file in Directory.EnumerateFiles(@"C:\Data", "*.*",
    SearchOption.AllDirectories))
{
    Console.WriteLine(file);
}
```

### Path Operations
```csharp
string path = @"C:\Data\folder\file.txt";

string dir = Path.GetDirectoryName(path);     // C:\Data\folder
string file = Path.GetFileName(path);         // file.txt
string nameNoExt = Path.GetFileNameWithoutExtension(path); // file
string ext = Path.GetExtension(path);         // .txt
string root = Path.GetPathRoot(path);         // C:\

string combined = Path.Combine("folder", "sub", "file.txt");
string tempPath = Path.GetTempPath();
string tempFile = Path.GetTempFileName();

char separator = Path.DirectorySeparatorChar;  // \ on Windows, / on Linux
char altSeparator = Path.AltDirectorySeparatorChar;
```

## Stream-Based I/O

### Stream Hierarchy
```
Stream (abstract)
├── FileStream
├── MemoryStream
├── BufferedStream
├── NetworkStream
├── CryptoStream
├── GZipStream / DeflateStream
└── PipeStream
```

### FileStream
```csharp
using var fs = new FileStream("data.bin", FileMode.OpenOrCreate, FileAccess.ReadWrite);

// Write
byte[] data = Encoding.UTF8.GetBytes("Hello");
fs.Write(data, 0, data.Length);

// Read
byte[] buffer = new byte[1024];
int bytesRead = fs.Read(buffer, 0, buffer.Length);

// Seeking
fs.Seek(0, SeekOrigin.Begin);
long position = fs.Position;
long length = fs.Length;
```

### MemoryStream
```csharp
using var ms = new MemoryStream();
byte[] data = Encoding.UTF8.GetBytes("Hello");
ms.Write(data, 0, data.Length);
ms.Position = 0;
byte[] result = ms.ToArray();
```

### Readers and Writers
```csharp
// Binary
using var writer = new BinaryWriter(File.Open("data.bin", FileMode.Create));
writer.Write(42);              // int
writer.Write(3.14);            // double
writer.Write("Hello");         // string

using var reader = new BinaryReader(File.OpenRead("data.bin"));
int i = reader.ReadInt32();
double d = reader.ReadDouble();
string s = reader.ReadString();
```

## Async I/O
```csharp
// File
string content = await File.ReadAllTextAsync("file.txt");
await File.WriteAllTextAsync("file.txt", "content");

// Stream
using var stream = new FileStream("file.txt", FileMode.Open);
byte[] buffer = new byte[1024];
int bytesRead = await stream.ReadAsync(buffer, 0, buffer.Length);
await stream.WriteAsync(buffer, 0, bytesRead);

// Console
string? input = await Console.In.ReadLineAsync();
await Console.Out.WriteLineAsync("Async output");
```

## JSON Serialization (System.Text.Json)
```csharp
using System.Text.Json;

// Serialize
string json = JsonSerializer.Serialize(new { Name = "Alice", Age = 30 });
var options = new JsonSerializerOptions { WriteIndented = true };
string prettyJson = JsonSerializer.Serialize(obj, options);

// Deserialize
var person = JsonSerializer.Deserialize<Person>(json);
var dict = JsonSerializer.Deserialize<Dictionary<string, object>>(json);

// Async
await JsonSerializer.SerializeAsync(stream, obj);
var person = await JsonSerializer.DeserializeAsync<Person>(stream);
```

## Environment and Special Folders
```csharp
string desktop = Environment.GetFolderPath(Environment.SpecialFolder.Desktop);
string documents = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
string appData = Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData);

string currentDir = Environment.CurrentDirectory;
string machineName = Environment.MachineName;
string userName = Environment.UserName;
string osVersion = Environment.OSVersion.ToString();
string[] args = Environment.GetCommandLineArgs();
```
