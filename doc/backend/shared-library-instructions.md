next prompt: update services documents to use th shared messaging
make sure to select backend folder

# Shared Library Detailed Instructions

## Overview

The Shared Library (`MemeTokenHub.Shared`) is a .NET class library project that contains reusable code across all microservices (User, Token, Claim, Social, Payment, Notification). It eliminates code duplication and ensures consistency in DTOs, error handling, JWT utilities, and MongoDB configuration.

## Purpose and Responsibilities

The Shared Library provides:
- Common DTOs (Data Transfer Objects) for entities.
- Shared exception handling and custom exceptions.
- JWT token validation and generation helpers.
- Base repository class for MongoDB operations.
- Logging utilities and interfaces.
- Constants, enums, and configuration helpers.
- Extension methods for common operations.

## Project Setup

1. Create class library project:
   ```
   dotnet new classlib -n MemeTokenHub.Shared
   ```

2. Add NuGet packages:
   ```
   dotnet add package MongoDB.Driver
   dotnet add package System.IdentityModel.Tokens.Jwt
   dotnet add package Microsoft.Extensions.Logging
   dotnet add package Microsoft.Extensions.Configuration
   ```

3. Reference in all services:
   ```
   dotnet add reference ../MemeTokenHub.Shared/MemeTokenHub.Shared.csproj
   ```

## Core Components

### 1. DTOs (Folder: `Dtos`)

**User DTOs**
```csharp
public class UserDto
{
  public string UserId { get; set; }
  public string Username { get; set; }
  public string Email { get; set; }
  public string WalletAddress { get; set; }
  public string Role { get; set; } // "Authenticated", "Creator", etc.
  public DateTime CreatedAt { get; set; }
}

public class CreateUserDto
{
  public string Username { get; set; }
  public string Email { get; set; }
  public string WalletAddress { get; set; }
}

public class UpdateUserDto
{
  public string Username { get; set; }
  public string Bio { get; set; }
  public string AvatarUrl { get; set; }
}
```

**Token DTOs**
```csharp
public class TokenDto
{
  public string TokenId { get; set; }
  public string Name { get; set; }
  public string Symbol { get; set; }
  public string Chain { get; set; }
  public string ContractAddress { get; set; }
  public string Status { get; set; } // "Unclaimed", "Claimed"
  public decimal Price { get; set; }
  public DateTime CreatedAt { get; set; }
}

public class CreateTokenDto
{
  public string Name { get; set; }
  public string Symbol { get; set; }
  public string Description { get; set; }
  public string Chain { get; set; }
  public string ContractAddress { get; set; }
}
```

**Claim DTOs**
```csharp
public class ClaimDto
{
  public string ClaimId { get; set; }
  public string UserId { get; set; }
  public string TokenId { get; set; }
  public string Status { get; set; } // "Pending", "Approved", "Rejected"
  public string Description { get; set; }
  public DateTime SubmittedAt { get; set; }
}

public class CreateClaimDto
{
  public string TokenId { get; set; }
  public string Description { get; set; }
  public List<string> Attachments { get; set; }
}

public class ReviewClaimDto
{
  public string Status { get; set; } // "Approved", "Rejected"
  public string Notes { get; set; }
}
```

**Social DTOs**
```csharp
public class FollowDto
{
  public string FollowerId { get; set; }
  public string FollowingId { get; set; }
  public DateTime CreatedAt { get; set; }
}

public class ActivityDto
{
  public string ActivityId { get; set; }
  public string UserId { get; set; }
  public string Type { get; set; }
  public string Description { get; set; }
  public DateTime CreatedAt { get; set; }
}

public class ReputationDto
{
  public string UserId { get; set; }
  public int Score { get; set; }
  public List<string> Badges { get; set; }
}
```

**Payment DTOs**
```csharp
public class PaymentDto
{
  public string PaymentId { get; set; }
  public string UserId { get; set; }
  public string TokenId { get; set; }
  public decimal Amount { get; set; }
  public string Status { get; set; } // "Completed", "Failed"
  public DateTime CreatedAt { get; set; }
}

public class CreateCheckoutDto
{
  public string TokenId { get; set; }
  public decimal Amount { get; set; }
}
```

### 2. Exceptions (Folder: `Exceptions`)

```csharp
public class MemeTokenHubException : Exception
{
  public string Code { get; set; }
  
  public MemeTokenHubException(string message, string code = "UNKNOWN_ERROR") 
    : base(message)
  {
    Code = code;
  }
}

public class NotFoundException : MemeTokenHubException
{
  public NotFoundException(string resource, string id)
    : base($"{resource} with id {id} not found", "NOT_FOUND") { }
}

public class UnauthorizedException : MemeTokenHubException
{
  public UnauthorizedException(string message = "Unauthorized")
    : base(message, "UNAUTHORIZED") { }
}

public class ForbiddenException : MemeTokenHubException
{
  public ForbiddenException(string message = "Access forbidden")
    : base(message, "FORBIDDEN") { }
}

public class ValidationException : MemeTokenHubException
{
  public List<string> Errors { get; set; }
  
  public ValidationException(List<string> errors)
    : base("Validation failed", "VALIDATION_ERROR")
  {
    Errors = errors;
  }
}
```

### 3. Constants and Enums (Folder: `Constants`)

```csharp
public static class UserRoles
{
  public const string Anonymous = "Anonymous";
  public const string Authenticated = "Authenticated";
  public const string Creator = "Creator";
  public const string Collector = "Collector";
  public const string Moderator = "Moderator";
}

public static class ClaimStatus
{
  public const string Pending = "Pending";
  public const string Approved = "Approved";
  public const string Rejected = "Rejected";
}

public static class TokenStatus
{
  public const string Unclaimed = "Unclaimed";
  public const string Claimed = "Claimed";
  public const string Featured = "Featured";
}

public static class PaymentStatus
{
  public const string Pending = "Pending";
  public const string Completed = "Completed";
  public const string Failed = "Failed";
}

public static class ActivityType
{
  public const string ClaimSubmitted = "ClaimSubmitted";
  public const string TokenPublished = "TokenPublished";
  public const string UserFollowed = "UserFollowed";
}
```

### 4. JWT Utilities (Folder: `Auth`)

```csharp
public interface ITokenService
{
  string GenerateToken(string userId, string role, int expirationMinutes = 60);
  ClaimsPrincipal ValidateToken(string token);
}

public class JwtTokenService : ITokenService
{
  private readonly IConfiguration _configuration;
  
  public JwtTokenService(IConfiguration configuration)
  {
    _configuration = configuration;
  }
  
  public string GenerateToken(string userId, string role, int expirationMinutes = 60)
  {
    var key = new SymmetricSecurityKey(
      Encoding.UTF8.GetBytes(_configuration["Jwt:SecretKey"]));
    var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
    
    var claims = new[]
    {
      new Claim(ClaimTypes.NameIdentifier, userId),
      new Claim(ClaimTypes.Role, role)
    };
    
    var token = new JwtSecurityToken(
      issuer: _configuration["Jwt:Issuer"],
      audience: _configuration["Jwt:Audience"],
      claims: claims,
      expires: DateTime.UtcNow.AddMinutes(expirationMinutes),
      signingCredentials: creds
    );
    
    return new JwtSecurityTokenHandler().WriteToken(token);
  }
  
  public ClaimsPrincipal ValidateToken(string token)
  {
    var key = new SymmetricSecurityKey(
      Encoding.UTF8.GetBytes(_configuration["Jwt:SecretKey"]));
    
    try
    {
      var principal = new JwtSecurityTokenHandler().ValidateToken(token,
        new TokenValidationParameters
        {
          ValidateIssuerSigningKey = true,
          IssuerSigningKey = key,
          ValidateIssuer = true,
          ValidIssuer = _configuration["Jwt:Issuer"],
          ValidateAudience = true,
          ValidAudience = _configuration["Jwt:Audience"],
          ValidateLifetime = true,
          ClockSkew = TimeSpan.FromMinutes(2)
        }, out SecurityToken validatedToken);
      
      return principal;
    }
    catch
    {
      throw new UnauthorizedException("Invalid token");
    }
  }
}
```

### 5. Shared JWT Authentication Registration

Add a shared registration helper so each protected service can wire up the same JWT validation flow.

```csharp
public static class AuthExtensions
{
  public static IServiceCollection AddJwtAuthentication(
    this IServiceCollection services, IConfiguration configuration)
  {
    var secret = configuration["Jwt:SecretKey"];
    var issuer = configuration["Jwt:Issuer"];
    var audience = configuration["Jwt:Audience"];

    var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(secret));

    services.AddAuthentication(options =>
    {
      options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
      options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
    })
    .AddJwtBearer(options =>
    {
      options.TokenValidationParameters = new TokenValidationParameters
      {
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = key,
        ValidateIssuer = true,
        ValidIssuer = issuer,
        ValidateAudience = true,
        ValidAudience = audience,
        ValidateLifetime = true,
        ClockSkew = TimeSpan.FromMinutes(2)
      };
    });

    services.AddScoped<ITokenService, JwtTokenService>();

    return services;
  }
}
```

### 6. MongoDB Base Repository (Folder: `Data`)

```csharp
public interface IBaseRepository<T> where T : class
{
  Task<T> GetByIdAsync(string id);
  Task<IEnumerable<T>> GetAllAsync();
  Task<T> CreateAsync(T entity);
  Task<T> UpdateAsync(string id, T entity);
  Task DeleteAsync(string id);
}

public class BaseRepository<T> : IBaseRepository<T> where T : class
{
  protected readonly IMongoCollection<T> _collection;
  
  public BaseRepository(IMongoDatabase database, string collectionName)
  {
    _collection = database.GetCollection<T>(collectionName);
  }
  
  public async Task<T> GetByIdAsync(string id)
  {
    var objectId = ObjectId.Parse(id);
    return await _collection.Find(Builders<T>.Filter.Eq("_id", objectId))
      .FirstOrDefaultAsync();
  }
  
  public async Task<IEnumerable<T>> GetAllAsync()
  {
    return await _collection.Find(_ => true).ToListAsync();
  }
  
  public async Task<T> CreateAsync(T entity)
  {
    await _collection.InsertOneAsync(entity);
    return entity;
  }
  
  public async Task<T> UpdateAsync(string id, T entity)
  {
    var objectId = ObjectId.Parse(id);
    await _collection.ReplaceOneAsync(
      Builders<T>.Filter.Eq("_id", objectId), entity);
    return entity;
  }
  
  public async Task DeleteAsync(string id)
  {
    var objectId = ObjectId.Parse(id);
    await _collection.DeleteOneAsync(Builders<T>.Filter.Eq("_id", objectId));
  }
}
```

This shared library contains the common MongoDB infrastructure and generic CRUD repository base so protected services can reuse the same connection and data access patterns.

Service-specific repositories should extend `BaseRepository<T>` and add domain-specific queries or indexes in each microservice.

### 6. Event Contracts and Messaging Helpers (Folder: `Messaging`)

```csharp
public interface IEventMessage
{
  string EventType { get; }
  DateTime OccurredAt { get; }
}

public class ClaimApprovedEvent : IEventMessage
{
  public string EventType => "ClaimApproved";
  public DateTime OccurredAt { get; set; } = DateTime.UtcNow;
  public string ClaimId { get; set; }
  public string UserId { get; set; }
  public string TokenId { get; set; }
  public string Status { get; set; }
}

public class UserUpdatedEvent : IEventMessage
{
  public string EventType => "UserUpdated";
  public DateTime OccurredAt { get; set; } = DateTime.UtcNow;
  public string UserId { get; set; }
  public string Username { get; set; }
  public string Role { get; set; }
}
```

```csharp
public static class MessagingExtensions
{
  public static IServiceCollection AddServiceBusMessaging(
    this IServiceCollection services, IConfiguration configuration)
  {
    services.AddSingleton<ServiceBusClient>(provider =>
      new ServiceBusClient(configuration["ServiceBus:ConnectionString"]));

    return services;
  }
}
```

This shared messaging layer should include event contract DTOs, standard topic/queue names, and common helper registration for Azure Service Bus clients. Actual event handlers and domain routing logic remain in each service.

### 7. MongoDB Configuration (Folder: `Configuration`)

```csharp
public interface IMongoConnection
{
  IMongoDatabase GetDatabase(string databaseName);
}

public class MongoConnection : IMongoConnection
{
  private readonly IMongoClient _client;
  
  public MongoConnection(string connectionString)
  {
    _client = new MongoClient(connectionString);
  }
  
  public IMongoDatabase GetDatabase(string databaseName)
  {
    return _client.GetDatabase(databaseName);
  }
}

public static class MongoConfiguration
{
  public static IServiceCollection AddMongoDb(
    this IServiceCollection services, string connectionString, string databaseName)
  {
    services.AddSingleton<IMongoConnection>(
      new MongoConnection(connectionString));
    services.AddScoped(provider =>
    {
      var connection = provider.GetRequiredService<IMongoConnection>();
      return connection.GetDatabase(databaseName);
    });
    
    return services;
  }
}
```

### 7. Logging (Folder: `Logging`)

```csharp
public interface IAppLogger
{
  void LogInfo(string message);
  void LogWarning(string message);
  void LogError(string message, Exception ex = null);
  void LogDebug(string message);
}

public class AppLogger : IAppLogger
{
  private readonly ILogger<AppLogger> _logger;
  
  public AppLogger(ILogger<AppLogger> logger)
  {
    _logger = logger;
  }
  
  public void LogInfo(string message) => _logger.LogInformation(message);
  public void LogWarning(string message) => _logger.LogWarning(message);
  public void LogError(string message, Exception ex = null) => 
    _logger.LogError(ex, message);
  public void LogDebug(string message) => _logger.LogDebug(message);
}
```

### 8. Extension Methods (Folder: `Extensions`)

```csharp
public static class ServiceExtensions
{
  public static IServiceCollection AddSharedServices(
    this IServiceCollection services, IConfiguration configuration)
  {
    services.AddScoped<ITokenService, JwtTokenService>();
    services.AddScoped<IAppLogger, AppLogger>();
    
    return services;
  }
}

public static class AuthExtensions
{
  public static IServiceCollection AddJwtAuthentication(
    this IServiceCollection services, IConfiguration configuration)
  {
    var secret = configuration["Jwt:SecretKey"];
    var issuer = configuration["Jwt:Issuer"];
    var audience = configuration["Jwt:Audience"];

    var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(secret));

    services.AddAuthentication(options =>
    {
      options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
      options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
    })
    .AddJwtBearer(options =>
    {
      options.TokenValidationParameters = new TokenValidationParameters
      {
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = key,
        ValidateIssuer = true,
        ValidIssuer = issuer,
        ValidateAudience = true,
        ValidAudience = audience,
        ValidateLifetime = true,
        ClockSkew = TimeSpan.FromMinutes(2)
      };
    });

    services.AddScoped<ITokenService, JwtTokenService>();
    return services;
  }
}

public static class StringExtensions
{
  public static bool IsValidEmail(this string email)
  {
    try
    {
      var addr = new System.Net.Mail.MailAddress(email);
      return addr.Address == email;
    }
    catch
    {
      return false;
    }
  }
}
```

## File Structure

```
MemeTokenHub.Shared/
├── Dtos/
│   ├── UserDto.cs
│   ├── TokenDto.cs
│   ├── ClaimDto.cs
│   ├── SocialDto.cs
│   └── PaymentDto.cs
├── Exceptions/
│   ├── MemeTokenHubException.cs
│   ├── NotFoundException.cs
│   ├── UnauthorizedException.cs
│   └── ValidationException.cs
├── Constants/
│   ├── UserRoles.cs
│   ├── ClaimStatus.cs
│   └── PaymentStatus.cs
├── Auth/
│   ├── ITokenService.cs
│   └── JwtTokenService.cs
├── Data/
│   ├── IBaseRepository.cs
│   └── BaseRepository.cs
├── Configuration/
│   ├── IMongoConnection.cs
│   ├── MongoConnection.cs
│   └── MongoConfiguration.cs
├── Logging/
│   ├── IAppLogger.cs
│   └── AppLogger.cs
├── Extensions/
│   ├── ServiceExtensions.cs
│   └── StringExtensions.cs
└── MemeTokenHub.Shared.csproj
```

## Usage in Services

Each service references the shared library and uses its components:

```csharp
// Startup in a protected service (Startup.cs or Program.cs)
services.AddSharedServices(configuration);
services.AddJwtAuthentication(configuration);
services.AddMongoDb(connectionString, "MemeTokenHubDB");

app.UseAuthentication();
app.UseAuthorization();

// In controller
[ApiController]
[Route("api/[controller]")]
public class UserController : ControllerBase
{
  private readonly IUserRepository _repository;
  private readonly IAppLogger _logger;
  
  public UserController(IUserRepository repository, IAppLogger logger)
  {
    _repository = repository;
    _logger = logger;
  }
}
```

## Benefits

- **DRY Principle**: No code duplication across services.
- **Consistency**: All services use the same DTOs and exceptions.
- **Maintainability**: Centralized configuration and utilities.
- **Testing**: Shared utilities can be tested once.
- **Scalability**: Easy to add new shared components.

This spec enables an AI agent to generate the complete shared library.