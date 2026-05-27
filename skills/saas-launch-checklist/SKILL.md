---
name: saas-launch-checklist
description: "Comprehensive checklist for launching a production-ready SaaS. Covers multi-tenant RLS, auth flows, billing integration (Stripe), rate limiting, Sentry, PostHog, Resend, and performance optimization."
risk: safe
source: custom
date_added: "2026-05-27"
tags: ['saas', 'launch', 'checklist', 'security', 'production', 'monitoring']
tools: ["claude", "cursor", "gemini", "antigravity"]
---

# SaaS Launch Checklist

A rigorous production checklist to ensure a secure, scalable, and high-performance SaaS launch.

## Pre-Launch Checklist

### 1. Multi-Tenant Security
- [ ] Row-Level Security (RLS) enabled on all tenant-specific tables.
- [ ] Policies verified to prevent cross-tenant data leaks.
- [ ] Middleware sets `app.current_tenant_id` reliably on every database session/transaction.
- [ ] Admin route bypasses verified and auditing logs configured.

### 2. Authentication & Authorization
- [ ] Sign-up, Sign-in, Password Reset, and Verification flows tested.
- [ ] Social logins configured with production client IDs and secrets.
- [ ] Role-Based Access Control (RBAC) rules enforced on both frontend and APIs.
- [ ] Session validation hooks set to automatically sign out inactive users.

### 3. Billing & Subscriptions
- [ ] Stripe customer portal enabled.
- [ ] Webhook validation configured securely (using HMAC signing secrets).
- [ ] Database updates from webhooks (checkout completed, subscription updated/deleted) verified.
- [ ] Stripe checkout handles errors gracefully (payment failure, expired cards).

### 4. API Resilience & Defense
- [ ] Upstash Rate Limiting configured on all public endpoints.
- [ ] Edge middleware handles CORS correctly.
- [ ] Payload size limits set to prevent DDoS.
- [ ] Input schemas strictly validated via Zod.

### 5. Observability & Analytics
- [ ] Sentry error monitoring configured for Next.js (client, server, and edge).
- [ ] Source maps uploaded to Sentry in the production build.
- [ ] PostHog setup for feature flags and session recording.
- [ ] Webhook alerts for critical errors channeled to Slack or Discord.
- [ ] Resend transactional emails configured and domains authenticated (SPF, DKIM, DMARC).

### 6. Performance Optimization
- [ ] Lighthouse score: Performance > 90, Accessibility > 90, SEO > 95.
- [ ] Core Web Vitals (LCP < 2.5s, CLS < 0.1, INP < 200ms) within green thresholds.
- [ ] Database indexes present on all foreign keys and RLS filter columns.
- [ ] Static assets cached aggressively at the edge via CDN/Vercel.
