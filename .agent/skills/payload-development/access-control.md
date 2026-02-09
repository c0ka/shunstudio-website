---
name: Access Control
description: Collection, field, global access patterns
tags: [payload, access-control, security]
---

# Access Control

## Layers

1. **Collection** - Operations (create, read, update, delete, admin)
2. **Field** - Field access (create, read, update) - **boolean only**
3. **Global** - Global docs (read, update)

## Collection Access

```typescript
import type { Access } from 'payload'

access: {
  create: ({ req: { user } }) => Boolean(user),
  read: ({ req: { user } }) => user ? true : { status: { equals: 'published' } },
  update: ({ req: { user } }) => user?.roles?.includes('admin') ? true : { author: { equals: user?.id } },
  delete: async ({ req, id }) => {
    const count = await req.payload.count({ collection: 'comments', where: { post: { equals: id } } })
    return count === 0
  },
}
```

## Common Patterns

```typescript
// Anyone
const anyone: Access = () => true

// Authenticated
const authenticated: Access = ({ req: { user } }) => Boolean(user)

// Admin only
const adminOnly: Access = ({ req: { user } }) => user?.roles?.includes('admin')

// Admin or self
const adminOrSelf: Access = ({ req: { user } }) =>
  user?.roles?.includes('admin') ? true : { id: { equals: user?.id } }

// Org scoped
const orgScoped: Access = ({ req: { user } }) =>
  user?.roles?.includes('admin') ? true : { organization: { equals: user?.organization } }
```

## Field Access (Boolean Only)

```typescript
{
  name: 'salary',
  type: 'number',
  access: {
    read: ({ req: { user }, doc }) => user?.id === doc?.id || user?.roles?.includes('admin'),
    update: ({ req: { user } }) => user?.roles?.includes('admin'),
  },
}
```

## RBAC

```typescript
{
  name: 'roles',
  type: 'select',
  hasMany: true,
  options: ['admin', 'editor', 'user'],
  defaultValue: ['user'],
  saveToJWT: true, // Fast checks
  access: { update: ({ req: { user } }) => user?.roles?.includes('admin') },
}
```

## Multi-Tenant

```typescript
const tenantAccess: Access = ({ req: { user } }) => {
  if (!user) return false
  if (user.roles?.includes('super-admin')) return true
  return { tenant: { equals: user.tenantId } }
}
```

## Advanced Patterns

```typescript
// Recent records (last N days)
const recentRecords =
  (days: number): Access =>
  ({ req: { user } }) => {
    if (user?.roles?.includes('admin')) return true
    const cutoff = new Date()
    cutoff.setDate(cutoff.getDate() - days)
    return { createdAt: { greater_than_equal: cutoff.toISOString() } }
  }

// Scheduled content
const scheduledContent: Access = ({ req: { user } }) => {
  if (user?.roles?.some((r) => ['admin', 'editor'].includes(r))) return true
  const now = new Date().toISOString()
  return {
    and: [
      { publishDate: { less_than_equal: now } },
      { or: [{ unpublishDate: { exists: false } }, { unpublishDate: { greater_than: now } }] },
    ],
  }
}

// Tier-based
const tierAccess = (tier: string): Access => {
  const tiers = ['free', 'basic', 'pro', 'enterprise']
  return async ({ req: { user } }) => {
    if (user?.roles?.includes('admin')) return true
    const sub = await req.payload.findByID({ collection: 'subscriptions', id: user.subscriptionId })
    return sub?.status === 'active' && tiers.indexOf(sub.tier) >= tiers.indexOf(tier)
  }
}
```

## Factory Functions

```typescript
const createRoleAccess =
  (roles: string[]): Access =>
  ({ req: { user } }) =>
    user && roles.some((r) => user.roles?.includes(r))

const createOrgAccess =
  (allowAdmin = true): Access =>
  ({ req: { user } }) => {
    if (!user) return false
    if (allowAdmin && user.roles?.includes('admin')) return true
    return { organizationId: { in: user.organizationIds || [] } }
  }
```

## Performance

```typescript
// ✅ Use query constraints
const efficientAccess: Access = () => ({
  status: { equals: 'active' },
  organizationId: { in: ['org1', 'org2'] },
})

// ✅ Cache expensive checks
const cachedAccess: Access = ({ req: { context } }) => {
  if (!context.orgStatus) {
    context.orgStatus = checkOrgStatus()
  }
  return context.orgStatus
}
```

## Critical Notes

**Local API bypasses access by default**

```typescript
// ❌ WRONG
await payload.find({ collection: 'posts', user: someUser })

// ✅ CORRECT
await payload.find({ collection: 'posts', user: someUser, overrideAccess: false })
```
