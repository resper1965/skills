---
name: supabase-realtime
description: "SaaS real-time synchronization using Supabase Realtime (WebSockets). Covers channels, Presence, Broadcast, Postgres changes listeners, and tenant-scoped channel authorization."
risk: safe
source: custom
date_added: "2026-05-27"
tags: ['realtime', 'websockets', 'supabase', 'presence', 'broadcast']
tools: ["claude", "cursor", "gemini", "antigravity"]
---

# Supabase Realtime Channels

How to implement low-latency WebSockets synchronization, Presence, and Postgres event tracking.

## Client Setup
```typescript
import { createClient } from "@supabase/supabase-js";

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);
```

## 1. Broadcast (Ephemeral Messaging)
Send and receive messages with latency < 100ms. Perfect for cursors, typing indicators.
```typescript
// Subscribe to cursor movements channel
const channel = supabase.channel("room-1");

channel
  .on("broadcast", { event: "cursor" }, (payload) => {
    console.log("Cursor position:", payload);
  })
  .subscribe(async (status) => {
    if (status === "SUBSCRIBED") {
      // Send cursor coordinates
      await channel.send({
        type: "broadcast",
        event: "cursor",
        payload: { x: 100, y: 200 },
      });
    }
  });
```

## 2. Presence (Online User Status)
Track which users are currently online in a workspace.
```typescript
const presenceChannel = supabase.channel("online-users");

presenceChannel
  .on("presence", { event: "sync" }, () => {
    const state = presenceChannel.presenceState();
    console.log("Online users:", state);
  })
  .subscribe(async (status) => {
    if (status === "SUBSCRIBED") {
      await presenceChannel.track({
        user_id: "user-42",
        online_at: new Date().toISOString(),
      });
    }
  });
```

## 3. Postgres Changes
Listen to real-time updates directly from your Neon database tables.
```typescript
supabase
  .channel("db-changes")
  .on(
    "postgres_changes",
    { event: "INSERT", schema: "public", table: "messages" },
    (payload) => {
      console.log("New message:", payload.new);
    }
  )
  .subscribe();
```
*Note: Make sure Realtime is enabled on the target table in Supabase dashboard (or run `alter publication supabase_realtime add table messages;` in SQL).*

## 4. Multi-Tenant Scoping
Always prefix your channel names with the `tenant_id` to prevent users from subscribing to other tenants' channels.
```typescript
const tenantChannel = supabase.channel(`tenant_${tenantId}_chat`);
```
