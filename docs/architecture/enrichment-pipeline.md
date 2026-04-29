# Enrichment Pipeline

## Overview
Every LinkedIn profile lookup goes through this pipeline.
Each step is a Motia event step - independent, observable, retryable.

## Pipeline Steps

### Step 1: API Endpoint (steps/api/enrich-profile.step.ts)
- Receives POST request from Chrome extension
- Validates request body with Zod schema
- Checks JWT authentication
- Checks rate limits
- Emits: profile.received event

### Step 2: Cache Check (steps/enrichment/cache-check.step.ts)
- Listens: profile.received
- Checks Upstash Redis for cached profile
- Cache key: profile:{linkedin_url_hash}
- Cache TTL: 30 days
- Hit: emits enrichment.complete (zero API cost)
- Miss: emits cache.miss

### Step 3: Coin Check (steps/coins/check-balance.step.ts)
- Listens: cache.miss
- Verifies user has enough coins for requested action
- Not enough: emits coins.insufficient (returns error to extension)
- Enough: emits coins.verified

### Step 4: Hunter Email Lookup (steps/enrichment/hunter-email.step.ts)
- Listens: coins.verified
- Calls Hunter.io API with name + company domain
- Returns: email address + confidence score
- Emits: email.found or email.not_found

### Step 5: PDL Company Data (steps/enrichment/pdl-company.step.ts)
- Listens: email.found or email.not_found
- Calls People Data Labs API
- Returns: company size, industry, revenue, funding
- Emits: company.enriched

### Step 6: Merge Results (steps/enrichment/merge-results.step.ts)
- Listens: company.enriched
- Combines: profile + email + company data into one object
- Saves to Redis cache (30 day TTL)
- Saves to Supabase enriched_profiles table
- Emits: data.ready

### Step 7: AI Scoring (steps/ai/lead-scoring.step.py)
- Listens: data.ready
- Calls Gemini Flash with profile + user ICP config
- Returns: score 0-100 with 5-dimension breakdown + reason
- Emits: scoring.complete

### Step 8: AI Insights (steps/ai/generate-insights.step.py)
- Listens: scoring.complete
- Calls Gemini Flash for why-contact-now analysis
- Returns: timing reason + pain points + pitch angle
- Emits: insights.ready

### Step 9: Coin Deduction (steps/coins/deduct-coins.step.ts)
- Listens: insights.ready
- Deducts coins from user balance
- Logs to coin_transactions table
- Emits: coins.deducted

### Step 10: Return to Extension (steps/api/enrich-profile.step.ts)
- Listens: coins.deducted
- Returns complete enriched profile to Chrome extension
- Extension renders the side panel

## Error Handling
- Hunter fails -> emit email.not_found -> continue pipeline
- PDL fails -> emit company.partial -> continue with partial data
- Gemini fails -> fallback to OpenAI -> if both fail, return data without score
- Any step fails -> Motia retries automatically (3 attempts)
- All retries exhausted -> refund coins -> return error to extension

## Cache Strategy
| Data Type | Cache Key | TTL |
|-----------|-----------|-----|
| Full profile | profile:{url_hash} | 30 days |
| Email address | email:{name}:{domain} | 60 days |
| Company data | company:{name} | 30 days |
| AI score | score:{url_hash}:{icp_hash} | 7 days |
| Company news | news:{company} | 24 hours |

## Cost Impact of Caching
- 50 users: ~25% cache hit rate
- 500 users: ~55% cache hit rate
- 2000 users: ~70% cache hit rate
- At 70% hit rate: API costs reduced by 70%
