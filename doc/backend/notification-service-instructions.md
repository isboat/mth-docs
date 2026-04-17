# Notification Service Detailed Instructions

## Overview

The Notification Service handles event-driven notifications for Meme Token Hub. Based on the project overview and component diagram, it sends alerts for claim approvals, new tokens, and social activity.

## Purpose and Responsibilities

The Notification Service handles:
- Sending in-app, push, and email notifications.
- Managing user preferences for notifications.
- Listening to events from other services.
- Tracking delivery status.

It enhances user engagement with timely alerts.

## Tech Stack

- **Framework**: ASP.NET Core API.
- **Database**: MongoDB.
- **Messaging**: Azure Service Bus for event listening.
- **External**: Email/SMS providers for delivery.
- **Deployment**: Azure App Service.

## Data Models

### Notification Document (MongoDB Collection: Notifications)

```json
{
  "_id": "ObjectId",
  "notificationId": "string",
  "userId": "string",
  "type": "enum", // "ClaimApproval", "NewToken", "Follower"
  "message": "string",
  "channels": ["string"], // "InApp", "Email", "Push"
  "status": "enum", // "Sent", "Delivered", "Failed"
  "sentAt": "DateTime",
  "deliveredAt": "DateTime"
}
```

### UserPreferences Document

```json
{
  "_id": "ObjectId",
  "userId": "string",
  "emailEnabled": "boolean",
  "pushEnabled": "boolean",
  "inAppEnabled": "boolean"
}
```

## API Endpoints

Base URL: `/api/notifications`

1. **POST /api/notifications/send**
   - Send notification.
   - Body: Notification data.
   - Auth: Internal.

2. **GET /api/notifications/{userId}**
   - Get user's notifications.
   - Auth: Required.

3. **PUT /api/notifications/{userId}/preferences**
   - Update preferences.
   - Body: Preferences.
   - Auth: Required.

## Database Schema and Operations

- **Collections**: Notifications, UserPreferences.
- **Indexes**: userId, type, sentAt.

## Integration with Other Services

- **Claim Service**: Listens for approval events.
- **Social Service**: Listens for follower/activity events.
- **User Service**: Queries preferences.

## Implementation Guidance

### ASP.NET Core Setup
1. Create Web API project.
2. Add packages: `MongoDB.Driver`, Azure Service Bus client.
3. Configure MongoDB in `appsettings.json`.
4. Add shared services and shared JWT authentication:
   - `services.AddSharedServices(configuration);`
   - `services.AddJwtAuthentication(configuration);`
   - `services.AddMongoDb(connectionString, "MemeTokenHubDB");`
   - `app.UseAuthentication();`
   - `app.UseAuthorization();`
5. Implement event handlers, controllers, services, repositories.

### Key Classes
- `NotificationController`
- `NotificationService`
- `NotificationRepository : BaseRepository<Notification>`
- `EventHandler` for Azure Service Bus.

### Security
- Use shared JWT validation from the shared library for protected notification endpoints.
- Protect user notification queries and preference updates with `[Authorize]`.

### Testing
- Mock events for tests.

### Deployment
- Publish to Azure App Service.

## Example Workflow

1. Claim approved: Claim Service emits event.
2. Notification Service listens, sends alert based on user preferences.

This spec enables an AI agent to build the Notification Service.