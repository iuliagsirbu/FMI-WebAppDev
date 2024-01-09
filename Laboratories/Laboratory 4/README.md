# Laboratory 4

# Databases Pt. 2

## Adding the Database Relations

:bulb: Do a similar setup as in Laboratory 3, but with a different database to avoid problems with migrations.

Right-click on the folder `Models`, then add the following folders and files:

- `One-to-Many`:
  - `Model1.cs`
  - `Model2.cs`
- `Many-to-Many`:
  - `Model3.cs`
  - `Model4.cs`
- `One-to-One`:
  - `Model5.cs`
  - `Model6.cs`

## One-to-Many

In `Model1.cs`:

```csharp
using Lab4_24.Models.Base;

namespace Lab4_24.Models.One_to_Many
{
    public class Model1: BaseEntity
    {
        public string? Name { get; set; }

        // relation
        public ICollection<Model2> Models2 { get; set; } = default!;
    }
}
```

## :star: Some explanation

```csharp
public ICollection<Model2> Models2 { get; set; } = default!;
```

- This line establishes a one-to-many relationship between Model1 and Model2.
- `default!` initializes the `Models2` property. It is a way of saying "this will be initialized later, but it is not null." The ! operator is a null-forgiving operator in C# 8.0 or later, used to suppress the compiler warning about possible null reference.

In `Model2.cs`:

```csharp
using Lab4_24.Models.Base;

namespace Lab4_24.Models.One_to_Many
{
    public class Model2: BaseEntity
    {
        public string Description { get; set; } = default;

        // relation
        public Model1 Model1 { get; set; } = default;
        public Guid Model1Id { get; set; }
    }
}
```

## :star: Some other explanation

One-to-Many Relationship:

```csharp
public Model1 Model1 { get; set; } = default;
```

- This property establishes the "many" side of a one-to-many relationship between `Model2` and `Model1`. Each `Model2` instance is associated with one `Model1` instance.
- The =default; initializes `Model1` with the default value (which is null for reference types).

```csharp
public Guid Model1Id { get; set; }
```

- This property likely represents the foreign key in the database that links `Model2` to a specific instance of `Model1`.
- The type `Guid` suggests that the identifier used for `Model1` instances is of type `Guid` (Globally Unique Identifier).

## One-to-One

In `Model5.cs`:

```csharp
using Lab4_24.Models.Base;

namespace Lab4_24.Models.One_to_One
{
    public class Model5: BaseEntity
    {
        public int Size {  get; set; }

        // relation
        public Model6 Model6 { get; set; } = default!;
    }
}
```

In `Model6.cs`:

```csharp
using Lab4_24.Models.Base;

namespace Lab4_24.Models.One_to_One
{
    public class Model6: BaseEntity
    {
        public bool IsDeleted { get; set; }

        // relation
        public Model5 Model5 { get; set; } = default!;

        public Guid Model5Id {  get; set; }
    }
}
```

:star: The code is pretty self explanatory as it's similar to the previous relations.

## Many-to-Many

In the `Many-to-Many` folder add `ModelRelations.cs`.

In `Model3.cs`:

```csharp
using Lab4_24.Models.Base;

namespace Lab4_24.Models.Many_to_Many
{
    public class Model3: BaseEntity
    {
        public string? Name { get; set; }

        // relation

        // public ICollection<Model4> Models4 { get; set; }

        public ICollection<ModelsRelation> ModelsRelations { get; set; }
    }
}
```

In `Model4.cs`:

```csharp
using Lab4_24.Models.Base;

namespace Lab4_24.Models.Many_to_Many
{
    public class Model4: BaseEntity
    {
        public string? Name { get; set; }

        // relation

        public ICollection<ModelsRelation> ModelsRelations { get; set; }
    }
}
```

## :bulb: In many-to-many relationships, a join entity is often used to represent the table that holds the foreign keys to the two entities that form the many-to-many relationship.

In `ModelsRelation.cs`:

```csharp
namespace Lab4_24.Models.Many_to_Many
{
    public class ModelsRelation
    {
        public Guid Model3Id { get; set; }
        public Model3 Model3 { get; set; }

        public Guid Model4Id { get; set; }
        public Model4 Model4 { get; set; }
    }
}
```

## Making the connection with the Context

In `Lab4Context.cs`:

```csharp
using Lab4_24.Models;
using Lab4_24.Models.Many_to_Many;
using Lab4_24.Models.One_to_Many;
using Lab4_24.Models.One_to_One;
using Microsoft.EntityFrameworkCore;
using Microsoft.Identity.Client;

namespace Lab4_24.Data
{
    public class Lab4Context: DbContext
    {
        public DbSet<Student> Students { get; set; }

        // One to Many
        public DbSet<Model1> Models1 { get; set; }
        public DbSet<Model2> Models2 { get; set; }

        // One to One
        public DbSet<Model5> Models5 { get; set; }
        public DbSet<Model6> Models6 { get; set; }

        // Many to Many
        public DbSet<Model3> Models3 { get; set; }
        public DbSet<Model4> Models4 { get; set; }
        public DbSet<ModelsRelation> ModelsRelations { get; set; }


        public Lab4Context(DbContextOptions<Lab4Context> options) : base(options) { }
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            // One to Many
            modelBuilder.Entity<Model1>().HasMany(m1 => m1.Models2).WithOne(m2 => m2.Model1);

            // modelBuilder.Entity<Model2>().HasOne(m2 => m2.Model1).WithMany(m1 => m1.Models2);

            // One to One
            modelBuilder.Entity<Model5>().HasOne(m5 => m5.Model6).WithOne(m6 => m6.Model5).HasForeignKey<Model6>(m6 => m6.Model5Id);

            base.OnModelCreating(modelBuilder);

            // Many to Many
            modelBuilder.Entity<ModelsRelation>().HasKey(mr => new { mr.Model3Id, mr.Model4Id });

            modelBuilder.Entity<ModelsRelation>()
                        .HasOne(mr => mr.Model3)
                        .WithMany(m3 => m3.ModelsRelations)
                        .HasForeignKey(mr => mr.Model3Id);

            modelBuilder.Entity<ModelsRelation>()
                        .HasOne(mr => mr.Model4)
                        .WithMany(m4 => m4.ModelsRelations)
                        .HasForeignKey(mr => mr.Model4Id);

            base.OnModelCreating(modelBuilder);
        }
    }
}
```

## :bulb: Before proceeding with the migrations, make sure to build the project to see if it has any errors.

The migrations are just like in Laboratory 3.

---

In the `Controllers` folder add a new `API Controller - Empty` with the name `DatabaseController.cs`.

In `DatabaseController.cs`:

```csharp
using Lab4_24.Data;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace Lab4_24.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class DatabaseController : ControllerBase
    {
        public readonly Lab4Context _lab4Context;
        public DatabaseController(Lab4Context lab4Context)
        {
            _lab4Context = lab4Context;
        }

        [HttpGet("model1")]
        public async Task<IActionResult> GetModel1()
        {
            return Ok(await _lab4Context.Models1.ToListAsync());
        }
    }
}
```