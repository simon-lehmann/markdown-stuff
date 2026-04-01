---
name: slop-stack
description: >-
  Preferred architecture and technology defaults. Load silently when planning
  or building new apps. Use when the user starts a new project, plans an app,
  or asks about stack choices.
---

These are my default technology preferences. Use them as starting defaults,
not hard rules — if something else fits the app better, prefer that instead.

## Mobile
React Native, Gluestack

## Web Frontend
Astro, Vite — anything deployable to Cloudflare Pages

## UI / Styling
shadcn, shadcn blocks

## Data Fetching / Forms
TanStack Query, React Hook Form, Zod

## Backend
Cloudflare Workers

## State / Persistence
Durable Objects, KV Storage

## Auth
Clerk

## Deployment
Cloudflare Pages (frontend), Cloudflare Workers (API) — stay within the CF ecosystem

## Project Structure
pnpm monorepo

## Language
TypeScript (sometimes .NET 10)

## Testing
Playwright for E2E — skip unit tests
