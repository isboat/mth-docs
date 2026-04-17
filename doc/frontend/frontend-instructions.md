# Meme Token Hub Frontend Instructions

## Overview

Build the frontend as a modern, meme-token oriented experience with a polished, community-first interface. The app should feel bold, playful, and easy to navigate while supporting serious token discovery, creator/collector connection, claim workflows, and authenticated profile features.

Use:
- TypeScript with React for a type-safe and scalable SPA
- `privy-io/react-auth` for authentication and session management
- Tailwind CSS for responsive, modern UI styling
- `heliofi/checkout-react` for payments and token checkout flows

## Tech Stack

### Core
- React 18+ with TypeScript
- Vite for a fast developer experience and pure SPA routing
- `react-router-dom` for client-side navigation

### Authentication
- `@privy-io/react-auth` to handle login, registration, session persistence, and user guard behavior
- Support login flows for collectors, creators, and moderators
- Ensure anonymous browsing is available until the user authenticates
- Implement a Privy token exchange: frontend receives Privy token, sends it to backend exchange endpoint, and stores the returned JWT for protected requests

### Styling
- Tailwind CSS latest version
- Use utility-first styling for rapid layout design and a consistent theme
- Build a meme-inspired design system with:
  - neon accent colors
  - gradient backgrounds
  - large token cards
  - micro-animations and hover interactions
  - glassmorphism / card-style UI for token and profile panels

### Payments
- `heliofi/checkout-react` for checkout integration and token purchases
- Support one-click purchase or claim flows for premium meme tokens or featured drops
- Keep payment flows frictionless and mobile-friendly

## User Experience

### Public browsing
- Anonymous visitors can:
  - view trending tokens, featured drops, and public token feeds
  - search and filter tokens by popularity, category, chain, and claim status
  - explore creator/collector profiles and public token pages
  - view FAQs and About pages without signing in

### Authenticated experience
- Authenticated users should have access to:
  - a dashboard with saved tokens, watchlists, and claim status
  - token claim submission and status tracking
  - creator/collector profile editing and social verification
  - in-app notifications and personalized recommendations

### Persona-driven flows
- Creator / KOL profile owners: publish token launches, update pages, and view follower activity
- Collectors / Devs: discover new meme tokens, save drops, submit claims, and track approvals
- Moderator / approver: review pending claims and approve social validation workflows

## Recommended Pages and Components

### Pages
- Home / Discover
  - Trending meme token feed
  - Featured drops and curated collections
  - Search, sort, and filter controls
- Token details
  - Token metadata, on-chain info, social proof, and community chatter
  - Claim status, purchase actions, and token analytics
- Profile
  - User profile with bio, wallets, reputation badges, and saved tokens
  - Creator profile with published tokens and audience stats
- Dashboard
  - Personalized feed, saved items, watchlist, and claim history
- Claim submission
  - Guided claim form with attachments, proof fields, and status updates
- Moderator queue
  - Pending claims, review actions, and approval history
- Checkout / purchase
  - Payment UI powered by `heliofi/checkout-react`
  - Clear success, failure, and confirmation states

### Components
- Token card / list item
- Creator / collector profile card
- Search/filter toolbar
- Auth guard wrapper
- Notification toast / alert system
- Onboarding progress panel
- Payment checkout modal or embedded widget

## Authentication Implementation

1. Install `@privy-io/react-auth` and configure it in the app root.
2. Wrap the app with the Privy auth provider and expose hooks for login/logout and session state.
3. Build public and protected routes:
   - public token discovery, token details, and creator exploration
   - protected dashboard, claim submission, and profile edit pages
4. After Privy login, exchange the Privy token with backend endpoint `POST /api/users/auth/exchange`.
   - Send the Privy token from the frontend to your backend.
   - Receive a backend-issued JWT.
   - Store the JWT in memory or secure storage and attach it to authenticated API requests.
5. Use a fast onboarding flow that supports email or wallet-linked registration if available.
6. Persist user preferences, saved tokens, and claim drafts in authenticated state.

## Payment Integration

1. Install `@heliofi/checkout-react` and configure merchant checkout options.
2. Use checkout components for:
   - purchasing featured meme tokens
   - adding token drops to a cart-like experience
   - supporting single-item and bundle checkout flows
3. Keep payment flow visually consistent with the rest of the app using Tailwind styling.
4. Show clear token purchase receipts and update the user dashboard after a successful checkout.

## Styling and UX Tone

- Keep the interface modern and meme-native:
  - large bold typography
  - vibrant gradients and subtle texture overlays
  - neon buttons, pill-shaped tags, and card shadows
  - status chips for trending, unclaimed, new, and verified tokens
- Use Tailwind CSS to build a responsive layout that works well on desktop and mobile.
- Add playful motion for hover, transitions, and page reveals.
- Make token discovery feel like a community hub, not a generic marketplace.

## Architecture Guidance

- Use a component-first structure: `components/`, `pages/`, `hooks/`, `services/`, `ui/`.
- Create reusable data hooks for tokens, profiles, claims, and payments.
- Keep auth logic isolated in an `auth/` module using `privy-io/react-auth` hooks.
- Store style tokens and theme values in Tailwind config for brand consistency.
- Use TypeScript interfaces for token metadata, user profiles, claim records, and checkout items.

## Development Setup

1. Create the app with TypeScript support using Vite:
   - `npm create vite@latest . -- --template react-ts`
2. Install dependencies:
   - `npm install react react-dom` (if needed)
   - `npm install @privy-io/react-auth @heliofi/checkout-react tailwindcss postcss autoprefixer`
   - `npm install react-router-dom` (if using Vite)
3. Configure Tailwind:
   - `npx tailwindcss init -p`
   - add content paths and theme extensions
4. Set up Privy auth provider and checkout provider in
   `src/main.tsx` / `src/pages/_app.tsx`.
5. Build the UI with responsive layouts and token-first interactions.

## Design Principles

- Make every screen feel like a meme token clubhouse.
- Prioritize discovery, social context, and token identity.
- Keep public browsing simple; make auth optional until users want to save, submit, or purchase.
- Ensure the app is polished, mobile-friendly, and strongly branded around meme token culture.

## Summary

This frontend should be a modern React + TypeScript app with Tailwind styling, Privy authentication, and Helio checkout for purchase flows. The experience should reinforce the Meme Token Hub brand by combining playful design with powerful token discovery, creator/collector workflows, and secure authenticated features.
