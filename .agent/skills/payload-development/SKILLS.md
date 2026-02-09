---
name: payload-development
description: Expert Payload CMS development. Core principles and sub-skill navigation.
---

# Payload CMS Development

## Sub-Skills

- [Security Critical](security-critical.md) - **READ FIRST** - Critical security patterns
- [Access Control](access-control.md) - Collection, field, global access
- [Collections](collections.md) - Collection configs, auth, uploads, drafts
- [Fields](fields.md) - Field types and patterns
- [Hooks](hooks.md) - Lifecycle hooks and context
- [Queries](queries.md) - Local API query patterns
- [Components](components.md) - Custom React components
- [Endpoints](endpoints.md) - Custom REST endpoints
- [Adapters](adapters.md) - Database, storage, email
- [Configuration](configuration.md) - Project setup
- [Plugin Development](plugin-development.md) - Creating plugins
- [Field Type Guards](field-type-guards.md) - Runtime type checking

## Critical Security

### 1. Local API Access Control

```typescript
// ❌ WRONG: Bypasses access control
await payload.find({ collection: 'posts', user: someUser })

// ✅ CORRECT: Enforces permissions
await payload.find({ collection: 'posts', user: someUser, overrideAccess: false })
```

### 2. Transaction Safety

```typescript
// ❌ WRONG: Breaks atomicity
afterChange: [
  async ({ doc, req }) => {
    await req.payload.create({ collection: 'audit', data: { docId: doc.id } })
  },
]

// ✅ CORRECT: Pass req
afterChange: [
  async ({ doc, req }) => {
    await req.payload.create({ collection: 'audit', data: { docId: doc.id }, req })
  },
]
```

### 3. Prevent Hook Loops

```typescript
// Use context flags
afterChange: [
  async ({ doc, req, context }) => {
    if (context.skipHooks) return
    await req.payload.update({
      collection: 'posts',
      id: doc.id,
      data: { views: doc.views + 1 },
      context: { skipHooks: true },
      req,
    })
  },
]
```

## Best Practices

**Security**: Default restrictive access, use `saveToJWT: true` for roles, never trust client data  
**Performance**: Index queried fields, use `select`, set `maxDepth`, cache in `req.context`  
**Data Integrity**: Use `beforeValidate` for formatting, `beforeChange` for logic, enable transactions  
**Type Safety**: Import from `payload-types.ts`, use `as const`, use type guards  
**Organization**: Separate files for collections/access/hooks, document complex logic

## Common Gotchas

| Issue                         | Solution                                |
| ----------------------------- | --------------------------------------- |
| Local API bypasses access     | Set `overrideAccess: false` with `user` |
| Nested ops break transactions | Always pass `req`                       |
| Hook loops                    | Use `req.context` flags                 |
| Field access constraints      | Only boolean, no queries                |
| Types outdated                | Run `generate:types`                    |
| MongoDB transactions          | Require replica set                     |
| SQLite transactions           | Enable with `transactionOptions: {}`    |

## Validation

```bash
tsc --noEmit                    # TypeScript check
payload generate:importmap      # After component changes
npm run generate:types          # After schema changes
```
