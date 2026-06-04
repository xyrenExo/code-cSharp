# Global Usings and Implicit Usings

## Global Usings (C# 10+)

### Defining Global Usings
```csharp
// GlobalUsings.cs
global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading.Tasks;
global using static System.Math;  // Static imports
global using MyAlias = MyApp.SomeNamespace;  // Aliases
```

### Scoped to the Project
```csharp
// Any file in the project - no need to add using statements
var list = new List<int> { 1, 2, 3 };
var query = list.Where(x => x > 1);  // LINQ available globally
var task = Task.Run(() => { });       // Task available globally
```

## Implicit Usings (.NET 6+)

### How It Works
When `<ImplicitUsings>enable</ImplicitUsings>` is in the `.csproj`, the compiler generates global usings based on the SDK type:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
</Project>
```

### SDK-Specific Implicit Usings

**Microsoft.NET.Sdk** (Console, Class Library):
```csharp
global using System;
global using System.Collections.Generic;
global using System.IO;
global using System.Linq;
global using System.Net.Http;
global using System.Threading;
global using System.Threading.Tasks;
```

**Microsoft.NET.Sdk.Web** (ASP.NET):
Everything from Microsoft.NET.Sdk plus:
```csharp
global using System.Net.Http.Json;
global using Microsoft.AspNetCore.Builder;
global using Microsoft.AspNetCore.Hosting;
global using Microsoft.AspNetCore.Http;
global using Microsoft.AspNetCore.Routing;
global using Microsoft.Extensions.Configuration;
global using Microsoft.Extensions.DependencyInjection;
global using Microsoft.Extensions.Hosting;
global using Microsoft.Extensions.Logging;
```

### Controlling Implicit Usings
```csharp
// In .csproj - remove specific implicit usings
<PropertyGroup>
    <ImplicitUsings>enable</ImplicitUsings>
    <UsingRemove>System.Linq;System.Net.Http</UsingRemove>
</PropertyGroup>

// Add additional implicit usings
<ItemGroup>
    <Using Include="System.Text.Json" />
    <Using Include="System.Text.RegularExpressions" />
</ItemGroup>
```

## GlobalUsings.cs vs ImplicitUsings

| Feature | GlobalUsings.cs | ImplicitUsings |
|---------|----------------|----------------|
| Control | Manual per project | Automatic per SDK |
| Visibility | Project-wide | Project-wide |
| Customization | All `using` variations | SDK + csproj overrides |
| Version | C# 10+ | .NET 6+ |
| Static imports | Yes | No |
| Aliases | Yes | No |

## Best Practices

### Organizing Global Usings
```csharp
// GlobalUsings.cs - Keep it organized
// .NET namespaces
global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading;
global using System.Threading.Tasks;

// Third-party libraries
global using Newtonsoft.Json;
global using FluentValidation;

// Project namespaces
global using MyApp.Common;
global using MyApp.Models;

// Aliases
global using JsonDocument = System.Text.Json.JsonDocument;
```

### When to Use Global Usings
- Namespaces used across most files (>80%)
- Framework-level namespaces (System.*, Microsoft.*)
- Project-wide common namespaces
- Well-known third-party libraries

### When NOT to Use Global Usings
- Rarely used namespaces
- Namespaces causing ambiguity conflicts
- Experimental/conditional features

## Potential Issues

### Namespace Pollution
```csharp
// If multiple global usings have conflicting types:
global using System.Text.Json;
global using Newtonsoft.Json;

// Usage requires full qualification:
System.Text.Json.JsonSerializer.Serialize(obj);  // Must specify
```

### Maintenance
```csharp
// Avoid adding unnecessary global usings
// Files should still explicitly import rarely-used namespaces

// Bad global usings:
global using System.Xml;  // Only used in 2 files
global using System.Drawing;  // Only used in 1 file

// Better: add only in files that need them
```
