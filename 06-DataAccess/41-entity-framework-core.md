# Entity Framework Core

## Introduction
EF Core is a modern object-database mapper for .NET, supporting LINQ queries, change tracking, and migrations.

## Setup

### Install Packages
```bash
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

### DbContext
```csharp
public class AppDbContext : DbContext
{
    public DbSet<User> Users => Set<User>();
    public DbSet<Order> Orders => Set<Order>();
    public DbSet<Product> Products => Set<Product>();

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("Server=localhost;Database=MyDb;...");
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<User>(entity =>
        {
            entity.ToTable("Users");
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).HasMaxLength(100).IsRequired();
            entity.HasIndex(e => e.Email).IsUnique();
        });
    }
}
```

### Dependency Injection
```csharp
// Program.cs
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

## Entity Configuration

### Data Annotations
```csharp
[Table("Users")]
public class User
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int Id { get; set; }

    [Required]
    [MaxLength(100)]
    public string Name { get; set; } = string.Empty;

    [Required]
    [MaxLength(200)]
    [EmailAddress]
    public string Email { get; set; } = string.Empty;

    public DateTime CreatedAt { get; set; }

    public List<Order> Orders { get; set; } = new();
}
```

### Fluent API
```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<User>(entity =>
    {
        entity.Property(e => e.Name)
            .IsRequired()
            .HasMaxLength(100);

        entity.Property(e => e.Email)
            .IsRequired()
            .HasMaxLength(200);

        entity.HasIndex(e => e.Email)
            .IsUnique();

        entity.HasMany(e => e.Orders)
            .WithOne(e => e.User)
            .HasForeignKey(e => e.UserId)
            .OnDelete(DeleteBehavior.Cascade);
    });
}
```

## CRUD Operations

### Create
```csharp
var user = new User { Name = "Alice", Email = "alice@example.com" };
context.Users.Add(user);
await context.SaveChangesAsync();
// user.Id is now populated
```

### Read
```csharp
// All
var users = await context.Users.ToListAsync();

// Filtered
var activeUsers = await context.Users
    .Where(u => u.IsActive)
    .ToListAsync();

// Single
var user = await context.Users.FindAsync(1);
var user = await context.Users.FirstOrDefaultAsync(u => u.Email == email);
var user = await context.Users.SingleAsync(u => u.Id == 1);
```

### Update
```csharp
var user = await context.Users.FindAsync(1);
user.Name = "Updated Name";
await context.SaveChangesAsync();
```

### Delete
```csharp
var user = await context.Users.FindAsync(1);
context.Users.Remove(user);
await context.SaveChangesAsync();
```

## Relationships

```csharp
// One-to-Many
public class User
{
    public int Id { get; set; }
    public List<Order> Orders { get; set; } = new();
}

public class Order
{
    public int Id { get; set; }
    public int UserId { get; set; }
    public User User { get; set; } = null!;
}

// Many-to-Many (EF Core 5+)
public class Student
{
    public int Id { get; set; }
    public List<Course> Courses { get; set; } = new();
}

public class Course
{
    public int Id { get; set; }
    public List<Student> Students { get; set; } = new();
}
```

## Eager Loading

```csharp
// Include related data
var users = await context.Users
    .Include(u => u.Orders)
    .ThenInclude(o => o.Items)
    .ToListAsync();

// Filtered include
var users = await context.Users
    .Include(u => u.Orders.Where(o => o.IsActive))
    .ToListAsync();
```

## Migrations

```bash
# Create migration
dotnet ef migrations add InitialCreate

# Apply to database
dotnet ef database update

# Generate SQL script
dotnet ef migrations script

# Remove last migration
dotnet ef migrations remove

# Revert
dotnet ef database update PreviousMigrationName
```

## Query Performance

```csharp
// AsNoTracking (read-only queries)
var users = await context.Users
    .AsNoTracking()
    .Where(u => u.IsActive)
    .ToListAsync();

// Raw SQL
var users = await context.Users
    .FromSqlRaw("SELECT * FROM Users WHERE IsActive = {0}", true)
    .ToListAsync();

// Execute SQL
await context.Database.ExecuteSqlRawAsync("UPDATE Users SET IsActive = 0");

// Split queries (avoid cartesian explosion)
var users = await context.Users
    .Include(u => u.Orders)
    .AsSplitQuery()
    .ToListAsync();
```

## Logging

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(connectionString)
        .LogTo(Console.WriteLine, LogLevel.Information)
        .EnableSensitiveDataLogging()  // Development only!
        .EnableDetailedErrors();
}
```
