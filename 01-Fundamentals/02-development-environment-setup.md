# Setting Up Development Environment

## .NET SDK Installation

### Windows
1. Download from [dotnet.microsoft.com](https://dotnet.microsoft.com/download)
2. Run the installer
3. Verify installation:
```bash
dotnet --version
dotnet --list-sdks
```

### macOS
```bash
brew install dotnet
```
Or download from the official website.

### Linux (Ubuntu/Debian)
```bash
wget https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install dotnet-sdk-8.0
```

## IDE Options

### Visual Studio (Windows)
- Full-featured IDE
- Community edition is free
- Built-in debugging, testing, profiling
- Project templates and scaffolding

### Visual Studio Code
- Lightweight, cross-platform
- Install C# extension (OmniSharp)
- Install .NET Extension Pack
- Terminal integration for `dotnet` CLI

### JetBrains Rider
- Cross-platform, paid
- Advanced refactoring and navigation
- Built-in tools for Unity, ASP.NET, etc.

## Essential .NET CLI Commands

```bash
# Create new projects
dotnet new console -n MyApp
dotnet new webapi -n MyApi
dotnet new classlib -n MyLib
dotnet new xunit -n MyTests
dotnet new mvc -n MyMvcApp
dotnet new blazorwasm -n MyBlazor

# Build and run
dotnet build
dotnet run
dotnet run --project MyApp

# Testing
dotnet test
dotnet test --collect:"XPlat Code Coverage"

# Publish
dotnet publish -c Release -o ./publish
dotnet publish -c Release --self-contained true -r win-x64

# NuGet packages
dotnet add package Newtonsoft.Json
dotnet list package
dotnet remove package Newtonsoft.Json

# Solution management
dotnet new sln -n MySolution
dotnet sln add MyApp/MyApp.csproj
dotnet sln list
```

## Project Structure
```
MyApp/
├── MyApp.csproj          # Project file (MSBuild XML)
├── Program.cs             # Entry point
├── Models/                # Data models
├── Services/              # Business logic
├── Controllers/           # API controllers (ASP.NET)
├── Data/                  # Database context, migrations
├── Properties/            # launchSettings.json
└── appsettings.json       # Configuration
```

## .csproj File Example
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
</Project>
```

## Debugging Basics
- Set breakpoints (F9)
- Step over (F10), step into (F11)
- Watch variables, call stack
- Immediate window for quick expressions
- Debug.WriteLine() for output
