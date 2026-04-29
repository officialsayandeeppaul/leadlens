# LeadLens Architecture Overview

## What This Is
LeadLens is a monorepo containing 3 apps and 2 shared packages.

## The 3 Apps

### 1. Chrome Extension (apps/extension)
- Built with React + Vite + TypeScript + Manifest V3
- Runs on LinkedIn pages
- Reads profile data from DOM
- Shows enriched data in side panel
- Talks to backend via HTTPS

### 2. Backend (apps/backend)
- Built with Motia (event-driven pipeline engine)
- Node.js 20 + Python 3.13
- Handles all enrichment, AI, coins, payments
- Deploys to Railway

### 3. Dashboard (apps/dashboard)
- Built with Next.js 14 App Router
- Tailwind CSS + shadcn/ui (Nova preset)
- Lead management, billing, ICP settings
- Deploys to Vercel

## The 2 Shared Packages

### packages/types
- Single source of truth for all TypeScript types
- User, Lead, Coin, Plan, ICP, Score types
- Imported by all 3 apps

### packages/utils
- Shared utility functions
- formatINR(), formatDate(), COIN_COSTS, PLAN_LIMITS
- Imported by all 3 apps

## How They Connect

Chrome Extension reads LinkedIn DOM
    -> sends profile data to Backend API
    -> Backend checks Redis cache
    -> Cache miss: calls Hunter.io + PDL
    -> Python step scores lead with Gemini
    -> Returns enriched data to Extension
    -> Extension shows panel to user
    -> User saves lead -> stored in Supabase
    -> Dashboard shows all saved leads

## Data Flow
User opens LinkedIn profile
    -> content script detects profile page
    -> background service worker pre-fetches
    -> side panel opens with enriched data
    -> user clicks save
    -> lead stored in Supabase
    -> available in dashboard immediately

## External Services
- Supabase: PostgreSQL database + Auth
- Upstash Redis: Caching layer
- Hunter.io: Email finding
- PDL: Company enrichment
- Gemini Flash: Lead scoring (free tier)
- GPT-4o Mini: Outreach generation
- Razorpay: Payments (INR, UPI)
- Resend: Transactional email
- MSG91: Phone OTP verification
- Railway: Backend hosting
- Vercel: Dashboard hosting
