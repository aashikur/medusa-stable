# Project Architecture & Troubleshooting Guide

Welcome to the Medusa + Next.js Starter Monorepo. This document breaks down the root causes behind common setup failures, explains how they were resolved, provides a future-proofing guide for new projects, and details the exact tech stack required to develop and maintain this codebase.

---

## 1. What Went Wrong & How We Fixed It

When starting up a full-stack E-commerce monorepo containing both a Medusa backend and a Next.js storefront, two main categories of issues frequently halt execution: **React dependency conflicts** and **frontend API configuration mismatches**.

### Issue 1: Backend React Version & Bundler Conflict
- **The Problem:** The root `package.json` had locked `react` and `react-dom` dependencies to `18.3.1` globally across all workspace apps. However, Next.js 15 (used in `apps/storefront`) requires React 19, whereas the Medusa Admin Dashboard (used in `apps/backend`) uses React 18. Forcing a single React version at the monorepo root caused package resolution errors and bundler crashes during `npm run backend:dev`.
- **The Fix:** Removed the top-level `"dependencies"` block containing `react` and `react-dom` from the root `package.json`. This allowed each sub-application (`apps/backend` and `apps/storefront`) to install and use its specific compatible React version independently.

### Issue 2: Storefront Data Fetching Failures
- **The Problem:** When running the storefront, server components were failing to render pages or fetch products/regions from the Medusa backend. This happened due to two misconfigurations:
  1. **Missing / Invalid Publishable API Key:** Medusa v2 requires every storefront HTTP request to pass a valid `NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY`.
  2. **Default Region Mismatch:** The storefront environment file (`apps/storefront/.env.local`) defaulted `NEXT_PUBLIC_DEFAULT_REGION` to `dk` (Denmark), but the backend database was seeded with `us` (United States), causing API lookup queries to return `null` and crash server pages.
- **The Fix:** Updated `NEXT_PUBLIC_DEFAULT_REGION=us` inside `apps/storefront/.env.local` and documented the exact steps to generate and bind a Publishable API Key from the Medusa Admin Dashboard.

---

## 2. Future Troubleshooting Guide (How to Fix This in a New Project)

If you create a new Medusa + Next.js project or encounter similar startup issues in the future, follow these verification steps:

### Step 1: Check Monorepo Dependency Isolation
- Open the root `package.json`.
- Ensure heavy framework dependencies like `react`, `react-dom`, or `next` are **NOT** listed in the root `dependencies`. They should only exist inside the specific `apps/<app-name>/package.json` files.

### Step 2: Verify Database & Migrations
- Ensure PostgreSQL is running locally and the connection string in `apps/backend/.env` is correct.
- Always run migrations and initial seed data before launching the storefront:
  ```bash
  cd apps/backend
  npx medusa db:migrate
  npm run backend:seed
  ```

### Step 3: Configure Publishable API Key & Regions
- Start the backend (`npm run backend:dev`) and access `http://localhost:9000/app`.
- Go to **Settings > Publishable API Keys**, create a key, and link it to your **Default Sales Channel**.
- Copy the generated key into `apps/storefront/.env.local`:
  ```env
  NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=pk_your_generated_key
  NEXT_PUBLIC_MEDUSA_BACKEND_URL=http://localhost:9000
  NEXT_PUBLIC_DEFAULT_REGION=us
  ```

---

## 3. Tech Stack Requirements

To comfortably maintain and build features in this codebase, developers should be familiar with the following core tools and frameworks:

### Core Frameworks
1. **Medusa.js (v2)** — Open-source headless E-commerce engine powering the backend (modules, workflows, data models, and API routes).
2. **Next.js (v15 - App Router)** — React framework powering the storefront using Server Components, Server Actions, and dynamic routing.
3. **Turborepo & Workspaces** — Monorepo build system orchestrating task pipelines (`dev`, `build`, `lint`) across backend and storefront packages.

### Database & ORM
4. **PostgreSQL** — Relational database storing store items, orders, customers, and regions.
5. **Drizzle / MikrORM** — Used internally by Medusa v2 for data modeling and migrations.

### Language & Styling
6. **TypeScript** — Strictly typed codebase for both API integration models and UI components.
7. **Tailwind CSS** — Utility-first CSS framework powering the storefront user interface styling.
8. **Medusa UI / Radix UI** — Component libraries used across the Medusa Admin dashboard and storefront interface.
