# Payment Service Detailed Instructions

## Overview

The Payment Service handles payment processing for Meme Token Hub, integrating with Helio for checkout flows. Based on the project overview and component diagram, it supports token purchases and transaction management.

## Purpose and Responsibilities

The Payment Service handles:
- Initiating checkouts via Helio integration.
- Confirming payments and generating receipts.
- Storing transaction history.
- Updating user dashboards post-purchase.

It ensures secure payment processing.

## Tech Stack

- **Framework**: ASP.NET Core API.
- **Database**: MongoDB.
- **External Integration**: Helio API for payments.
- **Communication**: REST APIs.
- **Deployment**: Azure App Service.

## Data Models

### Payment Document (MongoDB Collection: Payments)

```json
{
  "_id": "ObjectId",
  "paymentId": "string",
  "userId": "string",
  "tokenId": "string",
  "amount": "number",
  "currency": "string",
  "status": "enum", // "Pending", "Completed", "Failed"
  "helioTransactionId": "string",
  "receiptUrl": "string",
  "createdAt": "DateTime",
  "completedAt": "DateTime"
}
```

## API Endpoints

Base URL: `/api/payments`

1. **POST /api/payments/checkout**
   - Initiate checkout.
   - Body: { "userId": "string", "tokenId": "string", "amount": "number" }.
   - Response: Checkout URL or session.

2. **POST /api/payments/confirm**
   - Confirm payment (webhook from Helio).
   - Body: Helio payload.
   - Response: Success.

3. **GET /api/payments/{userId}/history**
   - Get user's payment history.
   - Auth: Required.

## Database Schema and Operations

- **Collection**: Payments.
- **Indexes**: userId, tokenId, status, createdAt.

## Integration with Other Services

- **User Service**: Update purchase history.
- **Token Service**: Link to purchased tokens.

## Implementation Guidance

### ASP.NET Core Setup
1. Create Web API project.
2. Add packages: `MongoDB.Driver`, `Helio` SDK or HTTP client.
3. Configure MongoDB in `appsettings.json`.
4. Add shared services and shared JWT authentication:
   - `services.AddSharedServices(configuration);`
   - `services.AddJwtAuthentication(configuration);`
   - `services.AddMongoDb(connectionString, "MemeTokenHubDB");`
   - `app.UseAuthentication();`
   - `app.UseAuthorization();`
5. Implement controllers, services, repositories.

### Key Classes
- `PaymentController`
- `PaymentService`
- `PaymentRepository : BaseRepository<Payment>`

### Security
- Use shared JWT validation from the shared library for user-facing endpoints.
- Protect purchase history and checkout endpoints with `[Authorize]`.
- Secure webhooks by validating Helio signatures.

### Testing
- Mock Helio for tests.

### Deployment
- Publish to Azure App Service.

## Example Workflow

1. User initiates purchase: Frontend calls Helio, then POST /api/payments/checkout.
2. Helio processes: Webhook to POST /api/payments/confirm.
3. On success: Update User Service.

This spec enables an AI agent to build the Payment Service.