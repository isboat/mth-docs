# User Service Detailed Instructions

## Overview

The User Service is a core microservice in the Meme Token Hub backend architecture. It manages user profiles, authentication, roles, and related data. Based on the project overview and component diagram, this service supports the authenticated user experience, role-based access, and integration with social features.

## Purpose and Responsibilities

The User Service handles:
- User profile management (creation, updates, retrieval).
- Role assignment and management (anonymous visitor, authenticated user, creator/KOL, collector/dev, moderator/approver).
- Authentication integration with Privy (frontend handles login, but service validates and stores user data).
- Wallet verification for on-chain credentials.
- User search and public profile access.
- Preferences and settings storage.

It ensures secure, role-based access for features like claim submission, profile editing, and moderation.

## Tech Stack

- **Framework**: ASP.NET Core API (Web API project).
- **Database**: MongoDB (document-based for flexible user data).
- **Authentication**: JWT tokens; Privy token exchange for frontend auth.
- **Communication**: REST APIs for synchronous calls; Azure Service Bus for publishing user update events (uses shared event contracts from shared library).
- **Deployment**: Azure App Service or cloud VM.

## Data Models

### User Document (MongoDB Collection: Users)

```json
{
  "_id": "ObjectId",
  "userId": "string", // Unique identifier (e.g., from Privy or generated)
  "username": "string", // Display name
  "email": "string", // Optional, for notifications
  "walletAddress": "string", // Blockchain wallet for verification
  "role": "enum", // "Anonymous", "Authenticated", "Creator", "Collector", "Moderator"
  "profile": {
    "bio": "string",
    "avatarUrl": "string",
    "socialLinks": ["string"], // Twitter, etc.
    "reputationScore": "number",
    "badges": ["string"] // e.g., "Top Creator"
  },
  "preferences": {
    "notificationsEnabled": "boolean",
    "theme": "string"
  },
  "createdAt": "DateTime",
  "updatedAt": "DateTime",
  "isVerified": "boolean" // Wallet/on-chain verification
}
```

### Role Enum
- Anonymous: Public browsing only.
- Authenticated: Basic user features.
- Creator: Publish tokens, manage pages.
- Collector: Discover, claim, save.
- Moderator: Review claims, approve validations.

## API Endpoints

All endpoints use RESTful conventions. Base URL: `/api/users`

### Authentication
- Use JWT in Authorization header for protected endpoints.
- API Gateway handles initial auth; service validates roles.

## Privy Token Exchange Flow

When a user logs in via Privy on the frontend, a token exchange is needed:

1. **Frontend**: User logs in with Privy, receives Privy auth token/session.
2. **Frontend**: Calls `POST /api/users/auth/exchange` with the Privy token.
3. **User Service**: Verifies the Privy token (with Privy API).
4. **User Service**: Checks if user exists in MongoDB; creates if new.
5. **User Service**: Issues a JWT token for the frontend to use with all services.
6. **Frontend**: Stores JWT token for authenticated requests.

### Endpoints

1. **GET /api/users/{userId}**
   - Description: Retrieve user profile (public or private based on auth).
   - Params: userId (path).
   - Response: User document (exclude sensitive data for public views).
   - Auth: Optional (public profiles); required for private.

2. **POST /api/users/auth/exchange**
   - Description: Exchange Privy token for backend JWT.
   - Body: { "privyToken": "string" }.
   - Response: { "jwtToken": "string", "user": UserDto }.
   - Auth: None (Privy token validates).
   - Flow: Frontend calls after Privy login; service verifies with Privy API and issues JWT.

3. **POST /api/users**
   - Description: Create new user profile (triggered after Privy login).
   - Body: User data (userId, username, email, walletAddress).
   - Response: Created user document.
   - Auth: Required (from Gateway).

3. **PUT /api/users/{userId}**
   - Description: Update user profile.
   - Params: userId (path).
   - Body: Updated fields (profile, preferences).
   - Response: Updated user document.
   - Auth: Required (user or admin).

4. **DELETE /api/users/{userId}**
   - Description: Deactivate user account.
   - Params: userId (path).
   - Response: Success message.
   - Auth: Required (user or admin).

5. **GET /api/users/search**
   - Description: Search users by username or wallet.
   - Query Params: query (string), limit (int).
   - Response: Array of user summaries.
   - Auth: Optional.

6. **POST /api/users/{userId}/verify-wallet**
   - Description: Verify wallet address (on-chain check).
   - Params: userId (path).
   - Body: Signature or proof.
   - Response: Verification status.
   - Auth: Required.

7. **PUT /api/users/{userId}/role**
   - Description: Assign or update role (admin/moderator only).
   - Params: userId (path).
   - Body: { "role": "string" }.
   - Response: Updated user.
   - Auth: Admin/Moderator.

## Database Schema and Operations

- **Collection**: Users (MongoDB).
- **Indexes**: userId (unique), username (unique), walletAddress (unique), role.
- **Operations**:
  - CRUD via MongoDB.Driver in ASP.NET Core.
  - Use async methods for performance.
  - Migrations: Handle schema updates via code (e.g., add new fields).

## Integration with Other Services

- **API Gateway**: Routes requests; provides JWT validation.
- **Social Service**: Queries user data for followers, leaderboards; listens for `UserUpdatedEvent` from shared library.
- **Claim Service**: Checks user roles for claim access.
- **Notification Service**: Listens for `UserUpdatedEvent` from shared library for notifications (via Azure Service Bus).
- **Frontend**: Privy auth triggers user creation; service provides profile data.

## Implementation Guidance

### ASP.NET Core Setup
1. Create Web API project: `dotnet new webapi -n UserService`.
2. Add packages: `Microsoft.AspNetCore.Authentication.JwtBearer`, `MongoDB.Driver`, `Azure.Messaging.ServiceBus`.
3. Configure MongoDB in `appsettings.json`: Connection string.
4. Add shared JWT authentication, shared services, and messaging:
   - `services.AddSharedServices(configuration);`
   - `services.AddJwtAuthentication(configuration);`
   - `services.AddMongoDb(connectionString, "MemeTokenHubDB");`
   - `services.AddServiceBusMessaging(configuration);`
   - `app.UseAuthentication();`
   - `app.UseAuthorization();`
5. Implement controllers, services, and event publishers for `UserUpdatedEvent`.

### Key Classes
- `UserController`: Handles API requests., publishes `UserUpdatedEvent` from shared library.
- `UserRepository`: MongoDB interactions.
- `UserEventPublisher`: Publishes `UserUpdatedEvent` (from shared library) to Azure Service Bus on profile updatevalidation).
- `UserRepository`: MongoDB interactions.
- `UserModel`: C# class for User document.

### Privy Integration and Token Exchange

1. **Install Privy SDK**: Add NuGet package for Privy API interaction (if available) or use HTTP client.

2. **Privy Configuration** (appsettings.json):
   ```json
   {
     "Privy": {
       "ApiKey": "your-privy-api-key",
       "ApiUrl": "https://api.privy.io"
     }
   }
   ```

3. **Implement PrivyService**:
   ```csharp
   public interface IPrivyService
   {
     Task<PrivyUser> VerifyTokenAsync(string privyToken);
   }
   
   public class PrivyService : IPrivyService
   {
     private readonly HttpClient _httpClient;
     private readonly IConfiguration _configuration;
     
     public PrivyService(HttpClient httpClient, IConfiguration configuration)
     {
       _httpClient = httpClient;
       _configuration = configuration;
     }
     
     public async Task<PrivyUser> VerifyTokenAsync(string privyToken)
     {
       // Call Privy API to verify token
       var request = new HttpRequestMessage(HttpMethod.Get, 
         $"{_configuration["Privy:ApiUrl"]}/verify");
       request.Headers.Add("Authorization", $"Bearer {_configuration["Privy:ApiKey"]}");
       request.Content = new StringContent($"{{\"token\": \"{privyToken}\"}}", 
         Encoding.UTF8, "application/json");
       
       var response = await _httpClient.SendAsync(request);
       if (!response.IsSuccessStatusCode)
         throw new UnauthorizedException("Invalid Privy token");
       
       var json = await response.Content.ReadAsStringAsync();
       return JsonConvert.DeserializeObject<PrivyUser>(json);
     }
   }
   ```

4. **Token Exchange in Controller**:
   ```csharp
   [HttpPost("auth/exchange")]
   public async Task<IActionResult> ExchangeToken([FromBody] ExchangeTokenRequest request)
   {
     try
     {
       // Verify Privy token
       var privyUser = await _privyService.VerifyTokenAsync(request.PrivyToken);
       
       // Check/create user in MongoDB
       var user = await _userService.GetOrCreateUserAsync(privyUser.Id, privyUser);
       
       // Issue JWT token
       var jwtToken = _tokenService.GenerateToken(user.UserId, user.Role);
       
       return Ok(new { 
         jwtToken = jwtToken, 
         user = new UserDto { UserId = user.UserId, Username = user.Username } 
       });
     }
     catch (Exception ex)
     {
       return Unauthorized(new { message = ex.Message });
     }
   }
   ```

### Security
- Validate JWT tokens.
- Role-based policies (e.g., [Authorize(Roles = "Moderator")]).
- Input validation and sanitization.

### Testing
- Unit tests for services/repositories.
- Integration tests for APIs.
- Use NUnit.

### Deployment
- Publish the ASP.NET Core app: `dotnet publish -c Release`.
- Deploy to Azure App Service or a cloud VM for scalability and integration with Azure Service Bus and MongoDB Atlas.

## Example Workflow

1. User logs in via Privy on frontend.
2. Frontend receives Privy token; calls POST /api/users/auth/exchange.
3. User Service verifies Privy token; creates or retrieves user from MongoDB.
4. Service issues JWT token to frontend.
5. On user update: Publish `UserUpdatedEvent` (from shared library) to Azure Service Bus for downstream services.
8. Moderator reviews claims; calls PUT /api/users/{userId}/role to assign role.
9. Role update: Publish updated `UserUpdatedEvent` to Azure Service Bus.

## Event Publishing

The User Service publishes events from the shared library:

```csharp
// In UserService after updating user profile
var userUpdatedEvent = new UserUpdatedEvent
{
  UserId = user.UserId,
  Username = user.Username,
  Role = user.Role
};

await _userEventPublisher.PublishUserUpdatedEventAsync(userUpdatedEvent);
```

This detailed spec allows an AI agent to generate the ASP.NET Core API code, MongoDB models, Privy integration, JWT exchange logic, and event publishing

This detailed spec allows an AI agent to generate the ASP.NET Core API code, MongoDB models, Privy integration, and JWT exchange logic for the User Service.