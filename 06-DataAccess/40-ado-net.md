# ADO.NET

## Introduction
ADO.NET is the foundational data access technology in .NET, providing direct access to databases.

## Connection

```csharp
// Connection string
string connectionString = "Server=localhost;Database=MyDb;User Id=sa;Password=pass;TrustServerCertificate=true;";

// SQL Server
using var connection = new SqlConnection(connectionString);
connection.Open();

// Connection pooling (automatic with default settings)
// Pooling increases performance by reusing connections
```

## Command (CRUD Operations)

### SELECT
```csharp
using var connection = new SqlConnection(connectionString);
await connection.OpenAsync();

var sql = "SELECT Id, Name, Email FROM Users WHERE IsActive = @IsActive";
using var command = new SqlCommand(sql, connection);
command.Parameters.AddWithValue("@IsActive", true);

using var reader = await command.ExecuteReaderAsync();
while (await reader.ReadAsync())
{
    int id = reader.GetInt32(0);
    string name = reader.GetString(1);
    string email = reader.IsDBNull(2) ? null : reader.GetString(2);

    var user = new User
    {
        Id = id,
        Name = name,
        Email = email
    };
    users.Add(user);
}
```

### INSERT
```csharp
var sql = @"INSERT INTO Users (Name, Email, CreatedAt)
            OUTPUT INSERTED.Id
            VALUES (@Name, @Email, @CreatedAt)";

using var command = new SqlCommand(sql, connection);
command.Parameters.AddWithValue("@Name", "Alice");
command.Parameters.AddWithValue("@Email", "alice@example.com");
command.Parameters.AddWithValue("@CreatedAt", DateTime.UtcNow);

int newId = (int)await command.ExecuteScalarAsync();
```

### UPDATE
```csharp
var sql = "UPDATE Users SET Name = @Name, Email = @Email WHERE Id = @Id";

using var command = new SqlCommand(sql, connection);
command.Parameters.AddWithValue("@Id", 1);
command.Parameters.AddWithValue("@Name", "Alice Updated");
command.Parameters.AddWithValue("@Email", "alice@updated.com");

int affected = await command.ExecuteNonQueryAsync();
```

### DELETE
```csharp
var sql = "DELETE FROM Users WHERE Id = @Id";
using var command = new SqlCommand(sql, connection);
command.Parameters.AddWithValue("@Id", 1);

int affected = await command.ExecuteNonQueryAsync();
```

## Stored Procedures

```csharp
using var command = new SqlCommand("usp_GetUserById", connection)
{
    CommandType = CommandType.StoredProcedure
};

command.Parameters.AddWithValue("@UserId", 1);

// Output parameter
var outputParam = new SqlParameter("@Result", SqlDbType.Int)
{
    Direction = ParameterDirection.Output
};
command.Parameters.Add(outputParam);

using var reader = await command.ExecuteReaderAsync();
int result = (int)outputParam.Value;
```

## Transactions

```csharp
using var transaction = connection.BeginTransaction();
try
{
    var cmd1 = new SqlCommand("UPDATE Accounts SET Balance = Balance - 100 WHERE Id = 1", connection, transaction);
    await cmd1.ExecuteNonQueryAsync();

    var cmd2 = new SqlCommand("UPDATE Accounts SET Balance = Balance + 100 WHERE Id = 2", connection, transaction);
    await cmd2.ExecuteNonQueryAsync();

    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

## Async Operations

```csharp
await connection.OpenAsync();
await command.ExecuteNonQueryAsync();
await command.ExecuteReaderAsync();
await command.ExecuteScalarAsync();
await reader.ReadAsync();
```

## DataSet and DataTable

```csharp
// Disconnected data: fill, modify, update back
var dataSet = new DataSet();
using var adapter = new SqlDataAdapter("SELECT * FROM Users", connection);
await adapter.FillAsync(dataSet, "Users");

// Modify
var table = dataSet.Tables["Users"];
table.Rows[0]["Name"] = "Updated";

// Update back
using var updateAdapter = new SqlCommandBuilder(adapter);
await adapter.UpdateAsync(dataSet, "Users");
```

## Dapper (Micro ORM)

```csharp
using var connection = new SqlConnection(connectionString);

// Query
var users = await connection.QueryAsync<User>(
    "SELECT * FROM Users WHERE IsActive = @IsActive",
    new { IsActive = true });

// Single
var user = await connection.QueryFirstOrDefaultAsync<User>(
    "SELECT * FROM Users WHERE Id = @Id",
    new { Id = 1 });

// Execute
await connection.ExecuteAsync(
    "UPDATE Users SET Name = @Name WHERE Id = @Id",
    new { Name = "Alice", Id = 1 });
```

## Best Practices

```csharp
// ALWAYS use parameterized queries (avoid SQL injection)
// BAD:
var sql = $"SELECT * FROM Users WHERE Name = '{name}'";  // SQL injection!

// GOOD:
var sql = "SELECT * FROM Users WHERE Name = @Name";
command.Parameters.AddWithValue("@Name", name);

// Use 'using' for all ADO.NET objects
// ConfigureAwait(false) for library code
// Use connection pooling (default)
// Close connections as soon as possible
```
