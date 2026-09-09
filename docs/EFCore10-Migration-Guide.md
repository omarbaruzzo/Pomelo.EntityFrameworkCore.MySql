# EF Core 10 Migration Guide for Microting MySQL Provider

This document outlines the breaking changes and migration path for upgrading to EF Core 10 with the Microting MySQL provider.

## Key Breaking Changes in EF Core 10

### 1. ExecuteUpdate API Changes

**Breaking Change**: The ExecuteUpdate API has changed from Expression-based to Action-based setters.

#### EF Core 9 and Earlier
```csharp
await context.Products
    .Where(p => p.CategoryId == 1)
    .ExecuteUpdateAsync(setters => setters
        .SetProperty(p => p.Price, p => p.Price * 1.1m)
        .SetProperty(p => p.LastModified, DateTime.UtcNow));
```

#### EF Core 10+
```csharp
await context.Products
    .Where(p => p.CategoryId == 1)
    .ExecuteUpdateAsync(p => new Product
    {
        Price = p.Price * 1.1m,
        LastModified = DateTime.UtcNow
    });
```

### 2. Migration Database Lock Interfaces

**Breaking Change**: Migration database lock interfaces have been introduced for better concurrency control.

#### New Interfaces in EF Core 10
- `IMigrationsDatabaseLock`
- `LockReleaseBehavior` enum

### 3. Query Expression Changes

**Breaking Change**: Several query expression methods have changed signatures or been removed.

## Migration Strategy

### Phase 1: Preparation ✅ Completed
1. ✅ Add conditional compilation support (`EFCORE10_OR_GREATER`, `EFCORE9_OR_GREATER`, `EFCORE8_OR_GREATER`)
2. ✅ Create `EFCoreCompatibilityHelper` class for version-agnostic patterns
3. ✅ Document breaking changes and migration patterns
4. ✅ Prepare conditional compilation patterns for affected code

### Phase 2: Implementation ✅ Completed
1. ✅ Update target framework to `net10.0`
2. ✅ Update EF Core packages to 10.0.x
3. ✅ Apply EF Core 10 API fixes
4. ✅ Update tests to the EF Core 10 API surface
5. ✅ Validate compatibility with existing applications

### Phase 3: Migration ✅ Completed (EF Core 10 is now the supported target)
1. Provide migration tools and scripts
2. Update documentation and examples
3. Create upgrade path for existing applications

## Conditional Compilation Patterns

The following patterns are recommended for handling version differences:

### Pattern 1: ExecuteUpdate with Single Property
```csharp
#if EFCORE10_OR_GREATER
await context.MyEntities
    .Where(e => e.Id == targetId)
    .ExecuteUpdateAsync(e => e.Name = newName);
#else
await context.MyEntities
    .Where(e => e.Id == targetId)
    .ExecuteUpdateAsync(setters => setters.SetProperty(e => e.Name, newName));
#endif
```

### Pattern 2: ExecuteUpdate with Multiple Properties
```csharp
#if EFCORE10_OR_GREATER
await context.MyEntities
    .Where(e => e.Active)
    .ExecuteUpdateAsync(e => new MyEntity
    {
        LastModified = DateTime.UtcNow,
        Status = "Updated"
    });
#else
await context.MyEntities
    .Where(e => e.Active)
    .ExecuteUpdateAsync(setters => setters
        .SetProperty(e => e.LastModified, DateTime.UtcNow)
        .SetProperty(e => e.Status, "Updated"));
#endif
```

### Pattern 3: Migration Lock Interfaces
```csharp
#if EFCORE10_OR_GREATER
protected override LockReleaseBehavior LockReleaseBehavior => LockReleaseBehavior.Immediate;

protected override IMigrationsDatabaseLock AcquireDatabaseLock()
{
    return new MySqlMigrationsDatabaseLock(/* parameters */);
}

protected override async Task<IMigrationsDatabaseLock> AcquireDatabaseLockAsync(CancellationToken cancellationToken = default)
{
    return await Task.FromResult(new MySqlMigrationsDatabaseLock(/* parameters */));
}
#endif
```

## Testing Strategy

### Unit Tests
- Create tests that validate behavior on EF Core 10
- Validate SQL generation for the EF Core 10 API surface

### Integration Tests
- Test migration scenarios
- Validate ExecuteUpdate operations
- Test database lock behavior

### Example Test Pattern
```csharp
[Fact]
public async Task ExecuteUpdate_UpdatesSingleProperty()
{
#if EFCORE10_OR_GREATER
    await context.Products
        .Where(p => p.Id == 1)
        .ExecuteUpdateAsync(p => p.Name = "Updated Name");
#else
    await context.Products
        .Where(p => p.Id == 1)
        .ExecuteUpdateAsync(setters => setters.SetProperty(p => p.Name, "Updated Name"));
#endif

    var product = await context.Products.FindAsync(1);
    Assert.Equal("Updated Name", product.Name);
}
```

## Compatibility Matrix

| EF Core Version | .NET Version | MySQL Provider Version | Status |
|----------------|--------------|------------------------|--------|
| 10.0.x         | .NET 10      | 10.0.x                | ✅ Current |
| 9.0.x          | .NET 9       | 9.0.x                 | Previous release |
| 8.0.x          | .NET 8       | 8.0.x                 | Previous release |

## Resources

- [EF Core 10 Breaking Changes](https://docs.microsoft.com/en-us/ef/core/what-is-new/ef-core-10.0/breaking-changes)
- [EF Core Migration Guide](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/)
- [Microting MySQL Provider Documentation](https://github.com/microting/Microting.EntityFrameworkCore.MySql)

## Next Steps

1. Keep the provider aligned with EF Core 10 patch releases
2. Maintain migration tooling for applications upgrading from EF Core 9
3. Keep documentation and examples up to date