# Dapper

## Introduction
Dapper is a lightweight, fast micro-ORM by Stack Overflow that extends IDbConnection with extension methods.

## Setup
```bash
dotnet add package Dapper
```

## Basic Queries

```csharp
using var connection = new SqlConnection(connectionString);

// Query multiple rows
var users = await connection.QueryAsync<User>(
    "SELECT Id, Name, Email FROM Users WHERE IsActive = @IsActive",
    new { IsActive = true });

// Query single row
var user = await connection.QueryFirstOrDefaultAsync<User>(
    "SELECT * FROM Users WHERE Id = @Id",
    new { Id = 1 });

// Query scalar value
int count = await connection.ExecuteScalarAsync<int>(
    "SELECT COUNT(*) FROM Users");
```

## Execute (INSERT, UPDATE, DELETE)

```csharp
// INSERT
int affected = await connection.ExecuteAsync(
    "INSERT INTO Users (Name, Email) VALUES (@Name, @Email)",
    new { Name = "Alice", Email = "alice@example.com" });

// INSERT with identity retrieval
int id = await connection.QuerySingleAsync<int>(
    @"INSERT INTO Users (Name, Email) OUTPUT INSERTED.Id VALUES (@Name, @Email)",
    new { Name = "Bob", Email = "bob@example.com" });

// UPDATE
await connection.ExecuteAsync(
    "UPDATE Users SET Name = @Name WHERE Id = @Id",
    new { Id = 1, Name = "Updated" });

// DELETE
await connection.ExecuteAsync(
    "DELETE FROM Users WHERE Id = @Id",
    new { Id = 1 });
```

## Multiple Results

```csharp
using var multi = await connection.QueryMultipleAsync(
    "SELECT * FROM Users; SELECT * FROM Orders;");

var users = await multi.ReadAsync<User>();
var orders = await multi.ReadAsync<Order>();
```

## Stored Procedures

```csharp
var user = await connection.QueryFirstOrDefaultAsync<User>(
    "usp_GetUserById",
    new { UserId = 1 },
    commandType: CommandType.StoredProcedure);

// With output parameters
var parameters = new DynamicParameters();
parameters.Add("@UserId", 1);
parameters.Add("@Result", dbType: DbType.Int32, direction: ParameterDirection.Output);

await connection.ExecuteAsync("usp_ProcessUser", parameters,
    commandType: CommandType.StoredProcedure);

int result = parameters.Get<int>("@Result");
```

## Complex Mappings

```csharp
// One-to-many
var sql = @"SELECT o.Id, o.OrderDate, oi.Id, oi.ProductName, oi.Quantity
            FROM Orders o
            JOIN OrderItems oi ON o.Id = oi.OrderId";

var orderDict = new Dictionary<int, Order>();

var orders = await connection.QueryAsync<Order, OrderItem, Order>(
    sql,
    (order, item) =>
    {
        if (!orderDict.TryGetValue(order.Id, out var existingOrder))
        {
            existingOrder = order;
            existingOrder.Items = new List<OrderItem>();
            orderDict.Add(order.Id, existingOrder);
        }
        existingOrder.Items.Add(item);
        return existingOrder;
    },
    splitOn: "Id"
);

var result = orderDict.Values;
```

## Transactions

```csharp
using var connection = new SqlConnection(connectionString);
await connection.OpenAsync();
using var transaction = connection.BeginTransaction();

try
{
    await connection.ExecuteAsync(
        "UPDATE Accounts SET Balance = Balance - 100 WHERE Id = @Id",
        new { Id = 1 }, transaction);

    await connection.ExecuteAsync(
        "UPDATE Accounts SET Balance = Balance + 100 WHERE Id = @Id",
        new { Id = 2 }, transaction);

    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

## Bulk Operations

```csharp
// Insert multiple
var users = new List<User>
{
    new() { Name = "Alice", Email = "alice@example.com" },
    new() { Name = "Bob", Email = "bob@example.com" }
};

await connection.ExecuteAsync(
    "INSERT INTO Users (Name, Email) VALUES (@Name, @Email)",
    users);  // Single command, multiple parameter sets
```

## Type Handlers

```csharp
// Custom type handler
public class DapperDateTimeHandler : SqlMapper.TypeHandler<DateTime>
{
    public override void SetValue(IDbDataParameter parameter, DateTime value)
    {
        parameter.Value = value;
    }

    public override DateTime Parse(object value)
    {
        return DateTime.SpecifyKind((DateTime)value, DateTimeKind.Utc);
    }
}

// Register
SqlMapper.AddTypeHandler(new DapperDateTimeHandler());
```

## Performance Tips

```csharp
// Use async methods
await connection.QueryAsync<T>();
await connection.ExecuteAsync();

// Use buffered vs unbuffered
var allData = await connection.QueryAsync<T>(sql);  // buffered (default)
var stream = await connection.QueryAsync<T>(sql, buffered: false);  // unbuffered

// Dynamic for flexible queries
var result = await connection.QueryAsync("SELECT * FROM Users");
foreach (IDictionary<string, object> row in result)
{
    Console.WriteLine(row["Name"]);
}
```

## Comparison: EF Core vs Dapper
| Feature | EF Core | Dapper |
|---------|---------|--------|
| Performance | Slower (overhead) | Fast |
| Setup | Complex | Minimal |
| Change tracking | Yes | No |
| Migrations | Yes | No |
| LINQ | Yes | Raw SQL |
| Lazy loading | Yes | No |
| Complex mapping | Automatic | Manual |
| Best for | Complex domains | Performance-critical |
