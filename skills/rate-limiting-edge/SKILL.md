---
name: rate-limiting-edge
description: "Implement API rate limiting at the Edge. Covers Upstash Rate Limit (Redis), Next.js Middleware integration, client IP extraction, and custom headers."
risk: safe
source: custom
date_added: "2026-05-27"
tags: ['rate-limiting', 'upstash', 'redis', 'nextjs', 'middleware', 'edge']
tools: ["claude", "cursor", "gemini", "antigravity"]
---

# Edge Rate Limiting

This skill guides the implementation of low-latency API rate limiting at the Edge (Next.js middleware or Cloudflare Workers) using Upstash Redis.

## Upstash Rate Limit Integration

### 1. Installation & Setup
```typescript
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL!,
  token: process.env.UPSTASH_REDIS_REST_TOKEN!,
});

// Create rate limiter: 10 requests per 10 seconds
export const ratelimit = new Ratelimit({
  redis: redis,
  limiter: Ratelimit.slidingWindow(10, "10 s"),
  analytics: true,
});
```

### 2. Next.js Edge Middleware
Apply rate limiting in `middleware.ts` before the request reaches the server.
```typescript
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";
import { ratelimit } from "@/lib/ratelimit";

export async function middleware(request: NextRequest) {
  const ip = request.ip ?? "127.0.0.1";
  const { success, limit, reset, remaining } = await ratelimit.limit(
    `rate_limit_ip_${ip}`
  );

  if (!success) {
    return new NextResponse("Too Many Requests", {
      status: 429,
      headers: {
        "X-RateLimit-Limit": limit.toString(),
        "X-RateLimit-Remaining": remaining.toString(),
        "X-RateLimit-Reset": reset.toString(),
      },
    });
  }

  return NextResponse.next();
}

export const config = {
  matcher: "/api/:path*",
};
```

### 3. Limiting by Tenant or User
For authenticated routes, rate limit based on the `userId` or `tenantId` instead of IP:
```typescript
const key = userId ? `rate_limit_user_${userId}` : `rate_limit_ip_${ip}`;
```
