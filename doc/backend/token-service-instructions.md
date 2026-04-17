# Token Service Detailed Instructions

## Overview

The Token Service is a core microservice in the Meme Token Hub backend, focused on token discovery and metadata management. Based on the project overview and component diagram, it provides the centralized hub for browsing, searching, and exploring meme tokens, supporting both anonymous and authenticated users.

## Purpose and Responsibilities

The Token Service handles:
- Token metadata storage and retrieval (name, description, chain, contract address).
- Feed generation (trending, unclaimed, featured, newly created).
- Search, filtering, and sorting (by popularity, chain, category, claim status).
- Integration with external on-chain APIs for real-time data (e.g., price, holders).
- Social proof and community context (views, likes, comments).
- Analytics and reporting for token performance.

It enables public browsing and supports claim workflows by providing token details.

## Tech Stack

- **Framework**: ASP.NET Core API (Web API project).
- **Database**: MongoDB (document-based for flexible token data).
- **External APIs**: Integrate with blockchain explorers (e.g., Etherscan, Solscan) for on-chain data.
- **Communication**: REST APIs; Azure Service Bus for events (e.g., new token added).
- **Deployment**: Publish to Azure App Service or cloud VM.

## Data Models

### Token Document (MongoDB Collection: Tokens)

```json
{
  "_id": "ObjectId",
  "tokenId": "string", // Unique identifier
  "name": "string",
  "symbol": "string",
  "description": "string",
  "chain": "string", // e.g., "Ethereum", "Solana"
  "contractAddress": "string",
  "category": "string", // e.g., "Meme", "DeFi"
  "creatorId": "string", // Reference to User Service
  "status": "enum", // "Unclaimed", "Claimed", "Featured"
  "metadata": {
    "supply": "number",
    "price": "number", // From external API
    "marketCap": "number",
    "holders": "number"
  },
  "socialProof": {
    "views": "number",
    "likes": "number",
    "comments": ["string"],
    "trendingScore": "number"
  },
  "createdAt": "DateTime",
  "updatedAt": "DateTime",
  "isActive": "boolean"
}
```

### Feed Types
- Trending: Based on views/likes in last 24h.
- Unclaimed: status = "Unclaimed".
- Featured: status = "Featured".
- New: createdAt within last week.

## API Endpoints

Base URL: `/api/tokens`

### Endpoints

1. **GET /api/tokens**
   - Description: Fetch token list with pagination, search, filters, and sorting.
   - Query Params: search (string), chain (string), category (string), status (string), sortBy (popularity|createdAt), limit (int), offset (int).
   - Response: Array of token summaries.
   - Auth: Optional.

2. **GET /api/tokens/{tokenId}**
   - Description: Get detailed token info.
   - Params: tokenId (path).
   - Response: Full token document.
   - Auth: Optional.

3. **POST /api/tokens**
   - Description: Create new token (for creators).
   - Body: Token data.
   - Response: Created token.
   - Auth: Required (Creator role).

4. **PUT /api/tokens/{tokenId}**
   - Description: Update token metadata.
   - Params: tokenId (path).
   - Body: Updated fields.
   - Response: Updated token.
   - Auth: Required (Creator or Admin).

5. **GET /api/tokens/feeds/{type}**
   - Description: Get specific feed (trending, unclaimed, featured, new).
   - Params: type (path), limit (query).
   - Response: Array of tokens.
   - Auth: Optional.

6. **POST /api/tokens/{tokenId}/like**
   - Description: Like a token.
   - Params: tokenId (path).
   - Response: Updated like count.
   - Auth: Required.

7. **GET /api/tokens/analytics/{tokenId}**
   - Description: Get token analytics.
   - Params: tokenId (path).
   - Response: Analytics data.
   - Auth: Optional.

## Database Schema and Operations

- **Collection**: Tokens (MongoDB).
- **Indexes**: tokenId (unique), chain, category, status, createdAt, trendingScore.
- **Operations**: CRUD via MongoDB.Driver; async for performance.

## Integration with Other Services

- **API Gateway**: Routes public requests.
- **Claim Service**: Updates claim status on tokens.
- **User Service**: Links creator profiles.
- **Social Service**: Pulls social proof data.
- **External APIs**: Fetch on-chain data periodically or on-demand.

## Implementation Guidance

### ASP.NET Core Setup
1. Create Web API project: `dotnet new webapi -n TokenService`.
2. Add packages: `MongoDB.Driver`, `HttpClient` for external APIs.
3. Configure MongoDB in `appsettings.json`.
4. Add shared services and shared JWT authentication:
   - `services.AddSharedServices(configuration);`
   - `services.AddJwtAuthentication(configuration);`
   - `services.AddMongoDb(connectionString, "MemeTokenHubDB");`
   - `app.UseAuthentication();`
   - `app.UseAuthorization();`
5. Implement controllers, services, and repositories.

### Key Classes
- `TokenController`: API endpoints.
- `TokenService`: Business logic (feeds, search).
- `TokenRepository`: MongoDB interactions.
- `ExternalApiService`: For blockchain data.

### Security
- Public endpoints for discovery.
- Auth for creation/updates.

### Testing
- Unit tests for logic.
- Integration tests for APIs and external calls.

### Deployment
- Publish: `dotnet publish -c Release`.
- Deploy to Azure App Service.

## Example Workflow

1. Anonymous user browses trending tokens: GET /api/tokens/feeds/trending.
2. Creator adds token: POST /api/tokens.
3. User searches: GET /api/tokens?search=meme.
4. Claim approved: Claim Service updates token status via API.

This spec enables an AI agent to build the Token Service with ASP.NET Core and MongoDB.