---
name: audit-logging
description: "Implement enterprise-grade SaaS audit logging. Covers database schema design, automated trigger-based logging, application-level audit logging, and audit trail APIs."
risk: safe
source: custom
date_added: "2026-05-27"
tags: ['audit-logging', 'saas', 'database', 'postgres-triggers', 'security']
tools: ["claude", "cursor", "gemini", "antigravity"]
---

# SaaS Enterprise Audit Logging

This skill describes the design and implementation of audit logging (audit trails) for tracking user actions and data changes.

## Database Schema Design
An audit log table must be immutable, tenant-scoped, and optimized for queries.
```sql
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE SET NULL,
  action VARCHAR(255) NOT NULL, -- e.g., 'project.created', 'settings.updated'
  resource VARCHAR(255) NOT NULL, -- e.g., 'projects', 'organizations'
  resource_id VARCHAR(255),
  metadata JSONB, -- Contextual data: IP, User-Agent, location
  changes JSONB, -- Before/After state comparison (diff)
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);

CREATE INDEX idx_audit_logs_tenant ON audit_logs(tenant_id, created_at DESC);
```

## Implementation Strategies

### 1. Database-Level Auditing (Triggers)
Automated change capture. Ideal for compliance since it runs regardless of application code bypass.
```sql
CREATE OR REPLACE FUNCTION audit_trigger_func() RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO audit_logs (tenant_id, user_id, action, resource, resource_id, changes)
  VALUES (
    COALESCE(NEW.tenant_id, OLD.tenant_id),
    current_setting('app.current_user_id', true)::uuid,
    TG_OP,
    TG_TABLE_NAME,
    COALESCE(NEW.id, OLD.id)::varchar,
    jsonb_build_object('old', to_jsonb(OLD), 'new', to_jsonb(NEW))
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

### 2. Application-Level Auditing (Middleware/Actions)
Ideal for tracking logical actions (e.g. "User exported data", "User invited member") rather than raw row mutations.
```typescript
import { db } from "@/server/db";
import { auditLogs } from "@/server/db/schema";

export async function logAuditEvent(event: {
  tenantId: string;
  userId: string;
  action: string;
  resource: string;
  resourceId?: string;
  metadata?: Record<string, any>;
  changes?: { old: any; new: any };
}) {
  await db.insert(auditLogs).values({
    tenantId: event.tenantId,
    userId: event.userId,
    action: event.action,
    resource: event.resource,
    resourceId: event.resourceId,
    metadata: event.metadata,
    changes: event.changes,
  });
}
```

## Best Practices
- **Immutability**: Never allow UPDATE or DELETE operations on `audit_logs` (revoke permission from application DB role).
- **IP Anonymization**: Comply with GDPR by hashing or mask-truncating client IP addresses in metadata.
- **Archival Policy**: Move logs older than 90 days to cold storage (e.g., Cloudflare R2 or AWS S3 Glacier).
