# Project Structure Documentation

## Complete Folder Structure
```
/StockMarketScreener
│
├── /src
│   ├── /Services
│   ├── /Data
│   ├── /Utilities
│   └── /Models
│
├── /tests
│   ├── /UnitTests
│   └── /IntegrationTests
│
└── /docs
```

## UML Class Mapping to C# Implementations
1. **Service Layer**
   - Description of each service and related classes.
   - Example:
     - `MarketDataService`
       - Responsibilities: Fetching market data, handling data transformations.
       - UML Class → C# Implementation: 
         ```csharp
         public class MarketDataService : IMarketDataService { ... }
         ```

2. **Data Access Layer**
   - Description of repository patterns and data models.
   - Example:
     - `MarketDataRepository`
       - Responsibilities: Data access, CRUD operations.
       - UML Class → C# Implementation:
         ```csharp
         public class MarketDataRepository : IMarketDataRepository { ... }
         ```

3. **Utilities**
   - Common utilities used across services.
   - Example:
     - `DateTimeUtility`
       - Responsibilities: Handling date and time operations.
       - UML Class → C# Implementation:
         ```csharp
         public static class DateTimeUtility { ... }
         ```

## Dependency Injection Configuration
- Setup for DI in the `Startup.cs` file:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddScoped<IMarketDataService, MarketDataService>();
    services.AddScoped<IMarketDataRepository, MarketDataRepository>();
}
```

## Usage Examples
- Example of usage in a controller:
```csharp
public class MarketController : ControllerBase
{
    private readonly IMarketDataService _marketDataService;

    public MarketController(IMarketDataService marketDataService)
    {
        _marketDataService = marketDataService;
    }

    public IActionResult GetMarketData()
    {
        var data = _marketDataService.GetMarketData();
        return Ok(data);
    }
}
```

