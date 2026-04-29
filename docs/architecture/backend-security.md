# Backend Security Architecture

## Authentication
Every request from Chrome extension includes a JWT token.
JWT is issued by Supabase Auth on login.
Middleware verifies JWT on EVERY request before any step runs.
If JWT is invalid or expired -> 401 returned immediately.
No request reaches business logic without valid JWT.

## BYOK Key Encryption
Users can bring their own AI API keys (OpenAI, Gemini, Claude).
Keys are NEVER stored in plain text.
Encryption: AES-256-GCM
Encryption key: stored in Railway environment variables only
Never in code, never in database, never in logs.
Keys are decrypted only at the moment of an API call.
After the call, decrypted value is discarded from memory.

## Razorpay Webhook Verification
Razorpay signs every webhook with HMAC-SHA256.
Your webhook secret is stored in Railway environment variables.
Every incoming webhook is verified before processing.
If signature does not match -> request rejected with 400.
If signature missing -> request rejected with 400.
This prevents fake payment events from crediting coins.

## Rate Limiting
Per-user request limits stored in Upstash Redis.
Free users: 10 requests per minute
Paid users: 60 requests per minute
If limit exceeded -> 429 returned
Limits reset every 60 seconds
Implemented in middleware/rate-limit.ts

## Trial Abuse Prevention
Phone OTP via MSG91 required before trial coins granted.
One phone number = one trial account forever.
Device fingerprinting via extension detects multiple accounts.
Same device creating 3+ accounts in 24 hours -> flagged.
Same IP creating 3+ accounts in 24 hours -> flagged.
Flagged accounts get 0 trial coins instead of 20.

## API Cost Protection
Free users: zero third-party API calls ever
Trial users: maximum Rs 5 API spend total
Each plan has a maximum monthly API spend cap.
If cap reached -> user throttled until next billing cycle.
Prevents a single abusive user from costing you money.

## Environment Variables
All API keys stored as Railway environment variables.
Never hardcoded in source code.
Never committed to GitHub.
.env files are in .gitignore.
.env.example shows variable names only, no real values.

## Database Security
Supabase Row Level Security (RLS) enabled on all tables.
Users can only read their own data.
Users can only write their own data.
Admin operations use service role key (server-side only).
Service role key never sent to browser or extension.

## HTTPS Only
All communication between extension and backend: HTTPS only.
Railway enforces HTTPS automatically.
Vercel enforces HTTPS automatically.
No HTTP allowed anywhere in production.
