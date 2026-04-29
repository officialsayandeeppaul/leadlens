# GitHub Flow and Branch Strategy

## Branch Structure
main      <- production. Railway + Vercel deploy from here. Never commit directly.
staging   <- pre-production. Final test with real APIs before going to main.
dev       <- development. All feature branches merge here.
feature/* <- one branch per feature, created from dev
fix/*     <- bug fixes during development, created from dev
hotfix/*  <- emergency production fixes, created from main
release/* <- release preparation, created from dev

## The Golden Rules
1. Never commit directly to main
2. Never commit directly to staging
3. Every change goes through a PR
4. PRs to dev and staging require typecheck to pass
5. Hotfixes are the ONLY exception - they go main -> hotfix/* -> main

## Daily Development Flow

### Starting a new feature
git checkout dev
git pull origin dev
git checkout -b feature/your-feature-name

### Working on the feature
git add .
git commit -m 'feat: description of what you built'
git push origin feature/your-feature-name

### When feature is ready
Go to GitHub -> New Pull Request
Base: dev
Compare: feature/your-feature-name
Create PR -> GitHub Actions runs typecheck
If checks pass -> Merge PR
Delete the feature branch

### Releasing to production
git checkout staging
git pull origin staging
git merge dev
git push origin staging
# Test on staging with real API keys
# If everything works:
git checkout main
git pull origin main
git merge staging
git push origin main
# Railway and Vercel auto-deploy

## Emergency Hotfix Flow
git checkout main
git pull origin main
git checkout -b hotfix/description-of-fix
# Fix the issue
git add .
git commit -m 'hotfix: what you fixed'
git push origin hotfix/description-of-fix
# PR -> main (emergency merge)
# After merging to main:
git checkout dev
git merge main
git push origin dev

## Commit Message Format
feat:     new feature added
fix:      bug fixed
hotfix:   emergency production fix
refactor: code restructured, no feature change
docs:     documentation only
style:    formatting, no logic change
test:     adding or fixing tests
chore:    config, dependencies, build changes
release:  version bump, changelog update

## Examples
feat: add Hunter.io email lookup step
fix: coin refund not triggering on empty email result
hotfix: razorpay webhook signature verification failing
chore: upgrade tailwindcss from 3.4.19 to 3.4.20
docs: update enrichment pipeline documentation
test: add unit tests for coin deduction logic
release: v1.0.0 MVP launch

## Branch Naming
feature/coin-system
feature/linkedin-dom-reader
feature/hunter-integration
feature/ai-lead-scoring
feature/razorpay-subscriptions
fix/cache-miss-not-refunding
fix/extension-postcss-config
hotfix/payment-webhook-500
hotfix/redis-connection-timeout
release/v1.0.0
release/v1.1.0

## What GitHub Actions Check on Every PR
1. TypeScript compilation - dashboard must compile
2. TypeScript compilation - extension must compile
3. TypeScript compilation - backend must compile
If any check fails - PR cannot be merged

## Environment per Branch
main    -> Railway production + Vercel production
staging -> Railway staging + Vercel preview
dev     -> Local only (your machine)
