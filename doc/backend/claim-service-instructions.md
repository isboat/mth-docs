# Claim Service Detailed Instructions

## Overview

The Claim Service manages claim submissions, reviews, and approvals in the Meme Token Hub. Based on the project overview and component diagram, it supports social validation workflows, enabling users to prove activity and participate in token campaigns.

## Purpose and Responsibilities

The Claim Service handles:
- Claim submission with attachments and proof fields.
- Review and approval workflows for moderators.
- Status tracking and history.
- Social validation integration.
- Guided forms for users and dashboards for reviewers.

It ensures secure, role-based claim processing.

## Tech Stack

- **Framework**: ASP.NET Core API.
- **Database**: MongoDB.
- **Communication**: REST APIs; Azure Service Bus for events.
- **Deployment**: Azure App Service.

## Data Models

### Claim Document (MongoDB Collection: Claims)

```json
{
  "_id": "ObjectId",
  "claimId": "string",
  "userId": "string", // From User Service
  "tokenId": "string", // From Token Service
  "type": "string", // e.g., "Ownership", "Activity"
  "description": "string",
  "attachments": ["string"], // URLs or file refs
  "proofFields": {
    "socialLinks": ["string"],
    "walletTx": "string"
  },
  "status": "enum", // "Pending", "Approved", "Rejected"
  "reviewerId": "string", // Moderator
  "reviewNotes": "string",
  "submittedAt": "DateTime",
  "reviewedAt": "DateTime"
}
```

## API Endpoints

Base URL: `/api/claims`

1. **POST /api/claims**
   - Submit claim.
   - Body: Claim data.
   - Auth: Required.

2. **GET /api/claims/{userId}**
   - Get user's claims.
   - Auth: Required.

3. **GET /api/claims/pending**
   - Get pending claims for review.
   - Auth: Moderator.

4. **PUT /api/claims/{claimId}/review**
   - Approve/reject claim.
   - Body: { "status": "Approved", "notes": "string" }.
   - Auth: Moderator.

## Database Schema and Operations

- **Collection**: Claims.
- **Indexes**: userId, tokenId, status, submittedAt.

## Integration with Other Services

- **User Service**: Role checks.
- **Token Service**: Update claim status.
- **Notification Service**: Events on approvals.

## Implementation Guidance

### ASP.NET Core Setup
1. Create Web API project.
2. Add packages: `MongoDB.Driver`.
3. Configure MongoDB in `appsettings.json`.
4. Add shared services and shared JWT authentication:
   - `services.AddSharedServices(configuration);`
   - `services.AddJwtAuthentication(configuration);`
   - `services.AddMongoDb(connectionString, "MemeTokenHubDB");`
   - `app.UseAuthentication();`
   - `app.UseAuthorization();`
5. Implement controllers, services, repositories.

### Key Classes
- `ClaimController`
- `ClaimService`
- `ClaimRepository : BaseRepository<Claim>`

### Security
- Use shared JWT validation from the shared library via `services.AddJwtAuthentication(configuration);`.
- Protect endpoints with `[Authorize]` and moderator-only review routes with `[Authorize(Roles = "Moderator")]`.

### Testing
- Unit and integration tests.

### Deployment
- Publish to Azure App Service.

## Example Workflow

1. User submits claim: POST /api/claims.
2. Moderator reviews: GET /api/claims/pending, PUT /api/claims/{id}/review.
3. On approval: Emit event to Notification Service.

This spec enables an AI agent to build the Claim Service.