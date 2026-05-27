---
name: resend-email
description: "SaaS transactional email delivery using Resend and React Email. Covers setup, React templates, queueing emails, webhook tracking, and domain authorization."
risk: safe
source: custom
date_added: "2026-05-27"
tags: ['email', 'resend', 'react-email', 'transactional', 'saas']
tools: ["claude", "cursor", "gemini", "antigravity"]
---

# Transactional Email with Resend

Standard patterns for setting up transactional emails with Resend and React Email.

## SDK Setup
```typescript
import { Resend } from "resend";

export const resend = new Resend(process.env.RESEND_API_KEY);
```

## Email Templates (React Email)
Build templates with Tailwind styling and standard layout blocks.
```tsx
// emails/welcome.tsx
import { Html, Button, Text, Container, Heading } from "@react-email/components";

export default function WelcomeEmail({ name }: { name: string }) {
  return (
    <Html>
      <Container style={{ padding: "20px" }}>
        <Heading>Welcome, {name}!</Heading>
        <Text>Thank you for signing up for our SaaS.</Text>
        <Button href="https://example.com/dashboard" style={{ background: "#000", color: "#fff", padding: "12px 20px", borderRadius: "5px" }}>
          Go to Dashboard
        </Button>
      </Container>
    </Html>
  );
}
```

## Sending Emails Non-Blocking
Do not await `resend.emails.send` inside user response cycles. Offload to a background job.
```typescript
import WelcomeEmail from "@/emails/welcome";
import { resend } from "@/lib/resend";

export async function sendWelcomeEmailJob(userId: string, email: string, name: string) {
  await resend.emails.send({
    from: "Acme <onboarding@yourdomain.com>",
    to: email,
    subject: "Welcome to Acme!",
    react: WelcomeEmail({ name }),
  });
}
```

## Webhooks (Deliverability Monitoring)
Configure a webhook endpoint in Resend pointing to `/api/webhooks/email` to capture events:
- `email.bounced` (deactivate sending to this email in database)
- `email.complained` (flag account for review)
- `email.delivered`
