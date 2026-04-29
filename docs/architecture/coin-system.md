# Coin System Architecture

## What Are Coins
Coins are the internal currency of LeadLens.
Users buy coin packs or get monthly allocations via subscription.
Every action costs coins. Different actions cost different amounts.

## Why Coins (Not Direct Lookups)
- Flexible: different features cost different amounts
- Psychological: users hate losing coins more than money
- Profitable: margin baked into coin pricing
- Transparent: users see exactly what they spend

## Coin Costs Per Action
| Action | Coins | Real API Cost | Margin |
|--------|-------|---------------|--------|
| Email lookup | 10 | Rs 0.80 | 88% |
| Company enrichment | 15 | Rs 1.20 | 75% |
| AI lead score | 5 | Rs 0.15 | 67% |
| AI insights | 8 | Rs 0.25 | 68% |
| Cold email generation | 10 | Rs 0.30 | 70% |
| LinkedIn DM | 6 | Rs 0.18 | 67% |
| Outreach sequence | 20 | Rs 0.60 | 67% |
| CRM push | 2 | Rs 0.02 | 90% |
| CSV export | 5 | Rs 0.05 | 89% |

## Coin Packs
| Pack | Coins | Price |
|------|-------|-------|
| Starter | 100 | Rs 99 |
| Growth | 300 | Rs 249 |
| Pro | 700 | Rs 499 |
| Power | 1500 | Rs 899 |

## Monthly Subscription Allocations
| Plan | Price | Coins | Rollover |
|------|-------|-------|----------|
| Free | Rs 0 | 0 | None |
| Starter | Rs 199 | 250 | 100 max |
| Growth | Rs 499 | 700 | 300 max |
| Pro | Rs 999 | 2000 | 500 max |
| Team | Rs 2499 | 6000 shared | 1000 max |

## Deduction Flow
1. User triggers action (e.g. email lookup)
2. Backend checks coin balance in Redis
3. Not enough coins -> return error, show upgrade prompt
4. Enough coins -> continue
5. Check Redis cache for this profile
6. Cache hit -> return data, deduct coins (no API called)
7. Cache miss -> call external API
8. API returns result -> deduct coins -> cache result
9. API returns empty -> refund coins automatically

## Critical Rules
- Coins only deduct AFTER successful API response
- Failed lookups always refund coins automatically
- Free users never trigger paid API calls
- Trial coins expire after 14 days
- Purchased packs never expire
- Subscription coins expire monthly (with rollover)

## Trial System
- New signup gets 20 trial coins after phone OTP
- Phone OTP prevents abuse (1 phone = 1 trial)
- Device fingerprinting detects multiple accounts
- Trial coins cover approximately 2 email lookups

## Admin Controls
- Change coin costs without code deployment
- Grant bonus coins to specific users
- Run promotional campaigns (double coins)
- Monitor coin velocity and API cost per coin
