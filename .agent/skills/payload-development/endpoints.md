---
name: Custom Endpoints
description: Custom REST endpoints
tags: [payload, endpoints, api]
---

# Custom Endpoints

**Not authenticated by default.** Always check `req.user`.

## Basic

```typescript
import { APIError } from 'payload'

export const protectedEndpoint: Endpoint = {
  path: '/protected',
  method: 'get',
  handler: async (req) => {
    if (!req.user) throw new APIError('Unauthorized', 401)
    const data = await req.payload.find({
      collection: 'posts',
      where: { author: { equals: req.user.id } },
    })
    return Response.json(data)
  },
}
```

## Patterns

```typescript
// Route params
{ path: '/:id/tracking', handler: async (req) => {
  const { id } = req.routeParams
  return Response.json({ id })
}}

// Request body
{ path: '/create', method: 'post', handler: async (req) => {
  const data = await req.json()
  const result = await req.payload.create({ collection: 'posts', data })
  return Response.json(result)
}}

// Query params
{ path: '/search', handler: async (req) => {
  const url = new URL(req.url)
  const query = url.searchParams.get('q')
  const results = await req.payload.find({ collection: 'posts', where: { title: { contains: query } } })
  return Response.json(results)
}}

// CORS
import { headersWithCors } from 'payload'
{ path: '/public', handler: async (req) => {
  return Response.json(data, { headers: headersWithCors({ headers: new Headers(), req }) })
}}
```

## Placement

```typescript
// Collection: /api/{slug}/{path}
export const Orders: CollectionConfig = {
  slug: 'orders',
  endpoints: [{ path: '/:id/tracking', handler: ... }],
}

// Global: /api/globals/{slug}/{path}
export const Settings: GlobalConfig = {
  slug: 'settings',
  endpoints: [{ path: '/clear-cache', handler: ... }],
}

// Root: /api/{path}
export default buildConfig({
  endpoints: [{ path: '/hello', handler: ... }],
})
```

## Best Practices

1. Always check authentication
2. Use `req.payload` for operations
3. Throw `APIError` for errors
4. Return `Response.json()`
5. Validate input
