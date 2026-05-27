---
name: webhook-handling
description: "Secure and robust webhook processing for SaaS (Stripe, Clerk, Supabase, GitHub). Covers signature verification (HMAC), idempotency/deduplication, queueing, and retry policies."
risk: safe
source: custom
date_added: "2026-05-27"
tags: ['webhooks', 'stripe', 'clerk', 'security', 'hmac', 'nextjs']
tools: ["claude", "cursor", "gemini", "antigravity"]
---

# Webhook Handling Patterns

Secure, robust, and asynchronous webhook ingestion handlers.

## Signature Verification (HMAC)
Never process webhooks without verifying their signature.
```typescript
// Next.js App Router API Route (app/api/webhooks/stripe/route.ts)
import { NextResponse } from "next/server";
import Stripe from "stripe";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

export async function POST(req: Request) {
  const body = await req.text();
  const signature = req.headers.get("stripe-signature");

  if (!signature) {
    return NextResponse.json({ error: "Missing signature" }, { status: 400 });
  }

  let event: Stripe.Event;
  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err: any) {
    return NextResponse.json({ error: `Signature verification failed: ${err.message}` }, { status: 400 });
  }

  // Handle event
  return NextResponse.json({ received: true });
}
```

## Webhook Idempotency (Deduplication)
Webhook providers may deliver the same event multiple times. Keep a log of event IDs.
```typescript
import { db } from "@/server/db";
import { webhookEvents } from "@/server/db/schema";
import { eq } from "drizzle-orm";

async function isEventProcessed(eventId: string): Promise<boolean> {
  const existing = await db.query.webhookEvents.findFirst({
    where: eq(webhookEvents.id, eventId),
  });
  return !!existing;
}

async function markEventProcessed(eventId: string, provider: string) {
  await db.insert(webhookEvents).values({ id: eventId, provider });
}
```

## Asynchronous Processing (Queues)
Never do heavy processing (email sending, PDF generation) inside the webhook HTTP request. Respond with `200 OK` as fast as possible to avoid timeouts, and offload processing to Inngest or Trigger.dev.
