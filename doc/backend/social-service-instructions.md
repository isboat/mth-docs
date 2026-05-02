# Social Service Detailed Instructions

## Overview

The Social Service manages community interactions and social features for Meme Token Hub. Based on the project overview and component diagram, it enables creators and collectors to connect, follow each other, and engage with the platform through activity streams, leaderboards, and reputation systems.

## Purpose and Responsibilities

The Social Service handles:
- Follow/unfollow relationships between users.
- Activity streams and user engagement tracking.
- Leaderboards and reputation badges based on contributor activity.
- Personalized recommendation feeds for token discovery.
- Social proof updates (likes, comments, shares).
- Community interactions on token pages.

It powers the "Creator and Collector Connection" feature of the platform.

## Tech Stack

- **Framework**: ASP.NET Core API.
- **Database**: MongoDB.
- **Communication**: REST APIs; Azure Service Bus for publishing activity events (uses shared event contracts).
- **Deployment**: Azure App Service.

## Data Models

### Follow Document (MongoDB Collection: Follows)

```json
{
  "_id": "ObjectId",
  "followId": "string",
  "followerId": "string", // User following
  "followingId": "string", // User being followed
  "createdAt": "DateTime"
}
```

### Activity Document (MongoDB Collection: Activities)

```json
{
  "_id": "ObjectId",
  "activityId": "string",
  "userId": "string",
  "type": "enum", // "ClaimSubmitted", "TokenPublished", "UserFollowed"
  "description": "string",
  "targetId": "string", // Token or User ID
  "createdAt": "DateTime"
}
```

### Reputation Document (MongoDB Collection: Reputations)

```json
{
  "_id": "ObjectId",
  "userId": "string",
  "score": "number",
  "badges": ["string"], // e.g., "TopCreator", "ActiveCollector"
  "claimsApproved": "number",
  "tokensPublished": "number",
  "followersCount": "number",
  "updatedAt": "DateTime"
}
```

### Engagement Document (MongoDB Collection: Engagements)

```json
{
  "_id": "ObjectId",
  "engagementId": "string",
  "userId": "string",
  "tokenId": "string",
  "type": "enum", // "Like", "Comment", "Share"
  "content": "string", // For comments
  "createdAt": "DateTime"
}
```

## API Endpoints

Base URL: `/api/social`

### Follow Endpoints

1. **POST /api/social/follow**
   - Follow a user.
   - Body: { "followingId": "string" }.
   - Auth: Required.

2. **POST /api/social/unfollow**
   - Unfollow a user.
   - Body: { "followingId": "string" }.
   - Auth: Required.

3. **GET /api/social/followers/{userId}**
   - Get user's followers.
   - Query: limit, offset.
   - Auth: Optional.

4. **GET /api/social/following/{userId}**
   - Get users that a user is following.
   - Query: limit, offset.
   - Auth: Optional.

### Activity and Feed Endpoints

5. **GET /api/social/feed/{userId}**
   - Get personalized activity feed.
   - Query: limit, offset.
   - Auth: Required.

6. **GET /api/social/activities/{userId}**
   - Get user's activity history.
   - Query: limit, offset.
   - Auth: Optional.

7. **POST /api/social/activities**
   - Create activity (internal).
   - Body: Activity data.
   - Auth: Internal.

### Leaderboards and Reputation

8. **GET /api/social/leaderboard/creators**
   - Get top creators by reputation.
   - Query: limit.
   - Auth: Optional.

9. **GET /api/social/leaderboard/collectors**
   - Get top collectors by reputation.
   - Query: limit.
   - Auth: Optional.

10. **GET /api/social/reputation/{userId}**
    - Get user's reputation and badges.
    - Auth: Optional.

### Engagement Endpoints

11. **POST /api/social/tokens/{tokenId}/like**
    - Like a token.
    - Auth: Required.

12. **POST /api/social/tokens/{tokenId}/comment**
    - Comment on a token.
    - Body: { "content": "string" }.
    - Auth: Required.

13. **GET /api/social/tokens/{tokenId}/engagement**
    - Get token engagement (likes, comments).
    - Auth: Optional.

## Database Schema and Operations

- **Collections**: Follows, Activities, Reputations, Engagements.
- **Indexes**: followerId, followingId, userId, tokenId, createdAt, score.
- **Operations**: CRUD via MongoDB.Driver; async for performance.

## Integration with Other Services

- **User Service**: Link follower/following data; update reputation scores; listens for `UserUpdatedEvent` from shared library.
- **Token Service**: Track token engagement metrics.
- **Notification Service**: Publishes custom activity events for follows, comments, badges based on shared messaging patterns.
- **Claim Service**: Track claimed tokens for reputation.

## Implementation Guidance

### ASP.NET Core Setup
1. Create Web API project: `dotnet new webapi -n SocialService`.
2. Add packages: `MongoDB.Driver`, `Azure.Messaging.ServiceBus`.
3. Configure MongoDB in `appsettings.json`.
4. Add shared services, JWT authentication, and messaging:
   - `services.AddSharedServices(configuration);`
   - `services.AddJwtAuthentication(configuration);`
   - `services.AddMongoDb(connectionString, "MemeTokenHubDB");`
   - `services.AddServiceBusMessaging(configuration);`
   - `app.UseAuthentication();`
   - `app.UseAuthorization();`
5. Implement controllers, services, repositories, and activity event publishers.

### Key Classes
- `SocialController`: API endpoints.
- `SocialService`: Business logic (follows, feeds, leaderboards).
- `ReputationService`: Calculate blish activity events using shared messaging patterns.
- `SocialRepository : BaseRepository<SocialActivity>`: MongoDB interactions.
- `ActivityEventPublisher`: Publishes custom activity events to Azure Service Bus following `IEventMessage` pattern from shared library
- `SocialRepository : BaseRepository<SocialActivity>`: MongoDB interactions.

### Security
- Use shared JWT validation from the shared library for protected endpoints.
- Add `services.AddSharedServices(configuration);` and `services.AddJwtAuthentication(configuration);` in `Program.cs` / `Startup.cs`.
- Role-based access for reputation updates.
- Protect write and user-specific routes with `[Authorize]`.

#### JWT Validation
- Use shared `AuthExtensions.AddJwtAuthentication(...)` to configure `TokenValidationParameters`.
- Example settings:
  - `ValidateIssuerSigningKey = true`
  - `IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(configuration["Jwt:SecretKey"]))`
  - `ValidateIssuer = true`
  - `ValidIssuer = configuration["Jwt:Issuer"]`
  - `ValidateAudience = true`
  - `ValidAudience = configuration["Jwt:Audience"]`
  - `ValidateLifetime = true`

#### Implementation
- Add JWT bearer authentication with the shared library helper in `Program.cs` / `Startup.cs`.
- Protect write and user-specific routes with `[Authorize]`.
- Use claims from the validated token to identify the user and role:
  - `User.FindFirst(ClaimTypes.NameIdentifier)` for user ID
  - `User.IsInRole("Creator")` for role checks
- Return `401 Unauthorized` for invalid or missing tokens and `403 Forbidden` for invalid role access.

### Testing
- Unit tests for service logic.
- Integration tests for APIs and MongoDB.

### Deployment
- Publish: `dotnet publish -c Release`.
- Deploy to Azure App Service.

## Example Workflow

1. User follows creator: POST /api/social/follow.
2. Creator publishes token: Create activity, emit event to Notification Service.
3. User views feed: GET /api/social/feed/{userId} returns personalized activities.
4. Leaderboard updated: Reputation Service calculates scores.
5. User likes token: POST /api/social/tokens/{tokenId}/like, update engagement metrics.

## Reputation Calculation

- Points for publishing tokens: +50.
- Points for approved claims: +10.
- Points per follower: +1.
- Badges awarded at thresholds (e.g., TopCreator at 500 points).

This spec enables an AI agent to build the Social Service.