# Meme Token Hub Architectural Diagram

## Purpose

This document describes the frontend architecture for Meme Token Hub, using the project overview as the functional foundation. It is written to help both AI agents and new developers understand the system structure, component boundaries, and key integrations.

## High-Level Architecture

The architectural diagram is organized into three main layers:

1. **Presentation Layer**
   - React + TypeScript app built with Vite
   - Tailwind CSS for styling
   - Public discovery screens and authenticated dashboards
   - Mobile-first, meme token UI with bold cards, filters, and status indicators

2. **Integration Layer**
   - `privy-io/react-auth` for authentication and session management
   - `heliofi/checkout-react` for payment and checkout flows
   - API service modules for token discovery, profile data, claim workflows, and notifications

3. **Backend / Data Layer**
   - External APIs and backend services that provide token metadata, user profiles, claim status, moderation data, and payment processing
   - Public token feeds accessible anonymously
   - Authenticated endpoints for dashboards, claim submission, saved items, and profile editing

## Core Components

- `App` shell: routes and layout
- `Discover` page: trending tokens, featured drops, search, filters
- `TokenDetails` page: token metadata, social proof, claim status, purchase actions
- `Profile` page: user and creator metadata, reputation badges, saved tokens
- `Dashboard` page: personalized feed, watchlist, claim history
- `ClaimForm` workflow: submission, attachments, review status
- `ModeratorQueue` page: pending claim review and approvals
- `Checkout` flow: Helio checkout component and payment confirmation

## Data Flow

1. Anonymous visitor opens the app.
2. Public discovery and token details pages fetch token feed data from the backend.
3. When the user chooses to login, `privy-io/react-auth` handles the auth flow.
4. Authenticated users fetch personalized data and can access protected routes.
5. Claim submission uses authenticated API endpoints and updates the user dashboard.
6. Payments use `heliofi/checkout-react` to process token purchases and then update purchase history.

## Authentication and Authorization

- Public routes: discovery, token details, creator/collector profiles, FAQ, About Us
- Protected routes: dashboard, claim submission, profile management, moderator queue
- Auth provider wraps the app root and exposes hooks for login, logout, and session state
- The architecture separates anonymous browsing from secure authenticated experiences

## Payment Integration

- The diagram shows a checkout component connected to the payment provider layer
- Payment events are handled by Helio and then persisted to the backend
- After checkout, the frontend updates the token purchase state and dashboard

## Diagram Narrative for AI Agents

For an AI agent, this architectural description should be interpreted as:
- A mapping of UX flows to components and pages
- The boundary between public data and authenticated data
- The role of external integrations (`privy-io/react-auth`, `heliofi/checkout-react`)
- Which pieces can be generated or automated safely, and which require secure backend coordination

An AI agent can use this structure to:
- Recommend page layouts and component composition
- Build feature stories for token discovery, claim workflows, and payments
- Understand when to keep actions client-side versus when to call secured APIs
- Align generated code and documentation with the Meme Token Hub vision

## Guidance for New Developers

New developers should read this as a blueprint for how the frontend fits together:
- Start with the app shell and routing, then build the public discovery and authenticated dashboard flows
- Keep auth logic isolated in an `auth/` module and use provider hooks across protected screens
- Create reusable UI components for token cards, profile cards, notification banners, and checkout modals
- Implement data hooks for token feeds, profile data, claim APIs, and payment status
- Use Tailwind CSS and theme tokens to keep the meme-inspired visual language consistent

## Key Architectural Principles

- **Component-first**: build reusable UI blocks for token discovery, profiles, and actions
- **Layered integration**: authenticate with Privy, pay with Helio, consume backend APIs for content
- **Public / private separation**: allow browsing without login, require auth only for saving, submitting, and purchasing
- **Modern UI**: keep the design sharp, responsive, and centered around meme token culture

## Summary

This architectural diagram description defines the frontend as a Vite-powered React app with Tailwind styling, Privy authentication, and Helio checkout. It is structured so AI agents can understand component roles and data flow, while new developers can implement the core pages, auth boundaries, and integrations in a consistent, meme-native way.