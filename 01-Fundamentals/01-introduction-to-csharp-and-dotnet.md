# Introduction to C# and .NET

## What is C#?
C# (pronounced "C sharp") is a modern, object-oriented, type-safe programming language developed by Microsoft. It runs on the .NET platform and is used to build a wide variety of applications.

## Key Characteristics
- **Object-Oriented**: Supports encapsulation, inheritance, and polymorphism
- **Type-Safe**: Ensures type safety at compile time
- **Cross-Platform**: Runs on Windows, Linux, macOS via .NET Core/.NET 5+
- **Garbage Collected**: Automatic memory management
- **Functional**: Supports functional programming features (LINQ, lambdas)
- **Async-Native**: Built-in async/await for asynchronous programming

## The .NET Platform
.NET is the runtime and framework that C# applications run on:

| Component | Description |
|-----------|-------------|
| **CLR (Common Language Runtime)** | Executes IL code, manages memory, handles exceptions |
| **BCL (Base Class Library)** | Provides built-in types, collections, I/O, networking |
| **FCL (Framework Class Library)** | Extended library for web, data, UI, etc. |

## C# Version History
| Version | Released | Key Features |
|---------|----------|--------------|
| C# 1.0  | 2002     | Basic OOP, classes, structs, interfaces |
| C# 2.0  | 2005     | Generics, nullable types, anonymous methods |
| C# 3.0  | 2007     | LINQ, lambda expressions, extension methods |
| C# 4.0  | 2010     | Dynamic binding, named/optional parameters |
| C# 5.0  | 2012     | Async/await |
| C# 6.0  | 2015     | String interpolation, null-conditional operators |
| C# 7.0+ | 2017+    | Pattern matching, tuples, local functions, records |
| C# 10   | 2021     | Global usings, file-scoped namespaces, record structs |
| C# 11   | 2022     | Raw string literals, required members, generic math |
| C# 12   | 2023     | Primary constructors, collection expressions, aliases |

## Application Types
- **Desktop Apps**: WPF, WinForms, MAUI
- **Web Apps**: ASP.NET Core, Blazor
- **Mobile Apps**: .NET MAUI, Xamarin
- **Cloud/Server**: Azure Functions, AWS Lambda
- **Game Development**: Unity
- **Microservices**: ASP.NET Core
- **IoT**: .NET IoT Libraries

## Compilation Process
```
C# Source Code → C# Compiler → IL (Intermediate Language) → CLR → Native Code
```

## Key Terminology
- **IL (Intermediate Language)**: Compiled C# code, platform-agnostic
- **JIT (Just-In-Time)**: Converts IL to native code at runtime
- **AOT (Ahead-Of-Time)**: Pre-compiles to native code (Native AOT)
- **GC (Garbage Collector)**: Automatically manages memory
- **CTS (Common Type System)**: Defines all data types in .NET
- **CLS (Common Language Specification)**: Rules for cross-language interoperability
