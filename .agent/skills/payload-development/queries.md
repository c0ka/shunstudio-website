---
name: Queries
description: Local API query patterns
tags: [payload, queries]
---

# Queries

## Operators

| Operator          | Example                                     |
| ----------------- | ------------------------------------------- |
| `equals`          | `{ color: { equals: 'blue' } }`             |
| `not_equals`      | `{ status: { not_equals: 'draft' } }`       |
| `greater_than`    | `{ price: { greater_than: 100 } }`          |
| `less_than_equal` | `{ age: { less_than_equal: 65 } }`          |
| `contains`        | `{ title: { contains: 'payload' } }`        |
| `like`            | `{ description: { like: 'cms headless' } }` |
| `in`              | `{ category: { in: ['tech', 'news'] } }`    |
| `exists`          | `{ image: { exists: true } }`               |
| `near`            | `{ location: { near: [10, 20, 5000] } }`    |

## Logic

```typescript
{ or: [{ color: { equals: 'mint' } }, { and: [{ color: { equals: 'white' } }, { featured: { equals: false } }] }] }
{ 'author.role': { equals: 'editor' } } // Nested
```

## Local API

```typescript
// Find
const posts = await payload.find({
  collection: 'posts',
  where: { status: { equals: 'published' } },
  depth: 2,
  limit: 10,
  sort: '-createdAt',
  select: { title: true, author: true },
})

// By ID
const post = await payload.findByID({ collection: 'posts', id: '123' })

// Create/Update/Delete
await payload.create({ collection: 'posts', data: { title: 'New' } })
await payload.update({ collection: 'posts', id: '123', data: { status: 'published' } })
await payload.delete({ collection: 'posts', id: '123' })

// Count
const count = await payload.count({
  collection: 'posts',
  where: { status: { equals: 'published' } },
})
```

## Access Control

**CRITICAL**: Local API bypasses access by default.

```typescript
// ❌ WRONG
await payload.find({ collection: 'posts', user: currentUser })

// ✅ CORRECT
await payload.find({ collection: 'posts', user: currentUser, overrideAccess: false })
```

## Performance

- Set `maxDepth` on relationships
- Use `select` to limit fields
- Index queried fields
- Cache in `req.context`
