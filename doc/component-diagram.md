# Meme Token Hub Component Diagram: Microservice Architecture

## Overview

This component diagram describes the microservice system design for Meme Token Hub, based on the project overview. It breaks down the backend into independent services that handle specific business domains, enabling scalability, maintainability, and team autonomy. The diagram focuses on service boundaries, data ownership, and inter-service communication.

## Key Principles

- **Domain-Driven Design (DDD)**: Each service owns a bounded context (e.g., tokens, users, claims) aligned with the project's core features.
- **Independence**: Services communicate via APIs (e.g., REST or gRPC) and can be deployed, scaled, and updated separately.
- **Data Ownership**: Each service manages its own MongoDB database to avoid tight coupling.
- **API Gateway**: A single entry point for the frontend to route requests, handling authentication and rate limiting.
- **Event-Driven Communication**: Use Azure Service Bus for asynchronous updates, like notifying users of claim approvals.
- **Security**: Implement JWT-based auth at the gateway level, with service-specific authorization.
- **Deployment**: Deploy to Azure App Service or cloud VMs for portability and scalability.
- **Monitoring**: Include logging, tracing (e.g., Jaeger), and health checks for each service.

## Microservice Components

### 1. User Service
- **Responsibilities**: Manages user profiles, authentication, and roles (anonymous visitor, authenticated user, creator/KOL, collector/dev, moderator/approver). Handles registration, login (integrating with Privy for frontend auth), profile updates, and wallet verification.
- **APIs**: Endpoints for profile CRUD, role assignment, and user search.
- **Data**: User database with profiles, roles, and preferences.
- **Connections**: Called by API Gateway for auth checks; queried by Social Service for follower data.

### 2. Token Service
- **Responsibilities**: Handles token discovery, metadata, and feeds (trending tokens, unclaimed tokens, featured drops). Supports search, filtering, and sorting by popularity, chain, category, and claim status. Integrates with external APIs for on-chain data.
- **APIs**: Endpoints for fetching token lists, details, and analytics.
- **Data**: Token database with metadata, social proof, and community context.
- **Connections**: Publicly accessible via API Gateway for anonymous browsing; integrates with Claim Service for claim status updates.

### 3. Claim Service
- **Responsibilities**: Manages claim submissions, review workflows, approvals, and social validation. Tracks claim status, attachments, and history. Supports guided forms and moderator dashboards.
- **APIs**: Endpoints for submitting claims, reviewing/approving, and fetching claim history.
- **Data**: Claim database with submissions, reviews, and validation records.
- **Connections**: Authenticated via API Gateway; emits events to Notification Service for approvals; queries User Service for role-based access.

### 4. Social Service
- **Responsibilities**: Handles social features like following creators, activity streams, notifications, leaderboards, and reputation badges. Manages community interactions, such as sharing token pages and personalized feeds.
- **APIs**: Endpoints for follows, activity feeds, notifications, and social proof updates.
- **Data**: Social database with relationships, activity logs, and engagement metrics.
- **Connections**: Integrates with User and Token Services for personalized data; triggers notifications via Notification Service.

### 5. Payment Service
- **Responsibilities**: Integrates with Helio (via `heliofi/checkout-react` on the frontend) for token purchases, checkout flows, and receipts. Handles payment processing, transaction history, and updates to user dashboards post-purchase.
- **APIs**: Endpoints for initiating checkouts, confirming payments, and fetching purchase history.
- **Data**: Payment database with transactions and receipts.
- **Connections**: Called by API Gateway for payment flows; updates User Service with purchase history.

### 6. Notification Service
- **Responsibilities**: Sends in-app notifications, push alerts, and email updates (e.g., claim approvals, new token drops, follower activity). Integrates with social and claim services for event-driven alerts.
- **APIs**: Endpoints for sending notifications and managing user preferences.
- **Data**: Notification database with message queues and delivery status.
- **Connections**: Listens to events from Claim and Social Services; queries User Service for preferences.

## Inter-Service Communication and Data Flow

- **Synchronous Communication**: REST APIs for direct requests (e.g., Token Service querying User Service for creator profiles).
- **Asynchronous Communication**: Azure Service Bus for cross-service updates (e.g., Claim Service emits an "approval" event, triggering notifications via the Notification Service and updating the user's dashboard).
- **Frontend Integration**: The API Gateway aggregates responses from services, ensuring the React app (with Privy auth and Helio payments) gets unified data.
  - Example Flow: Anonymous user browses tokens → Gateway routes to Token Service.
  - Authenticated user submits a claim → Gateway validates auth, routes to Claim Service, then triggers events to Social and Notification Services.

## Shared Components

- **API Gateway**: Routes requests, enforces auth, and provides a unified interface.
- **Event Bus**: Handles asynchronous messaging between services.
- **Shared Libraries**: Common schemas, event contract DTOs, messaging helpers, and utilities.
- **Config Service**: Manages environment variables and secrets.

Shared library content can include reusable communication contracts and Azure Service Bus helper registration, while service-specific handlers and business logic remain in each microservice.

## Implementation Steps

1. **Define Service Boundaries**: Start with core services (User, Token, Claim) and expand to Social, Payment, and Notification.
2. **Choose Tech Stack**: ASP.NET Core API for services, providing robust, scalable REST APIs; MongoDB for all service databases, providing flexibility for token metadata, user profiles, and dynamic content.
3. **Database Design**: Each service has its own DB; use migrations for schema changes.
4. **Testing and Deployment**: Unit tests per service, integration tests for API Gateway. Deploy to Azure App Service or cloud VMs for scalability and Azure integration.
5. **Security and Monitoring**: Implement OAuth/JWT, API versioning, and centralized logging.
6. **Scaling Considerations**: Token Service might need caching (Redis) for high-traffic feeds; Claim Service for review queues.

## Diagram Representation

Imagine a visual diagram with:
- Boxes for each service (User, Token, Claim, Social, Payment, Notification).
- Arrows showing API calls and event flows.
- The API Gateway at the top, connected to all services.
- External integrations (e.g., Helio, Privy) shown as external boxes.
- Databases as cylinders attached to each service.

This component diagram keeps the system modular, allowing teams to work on features like token discovery independently while supporting the project's community-focused goals.