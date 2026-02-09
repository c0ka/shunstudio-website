---
name: Hooks
description: Lifecycle hooks and context
tags: [payload, hooks]
---

# Hooks

## Collection Hooks

```typescript
hooks: {
  beforeValidate: [({ data, operation }) => {
    if (operation === 'create') data.slug = slugify(data.title)
    return data
  }],

  beforeChange: [({ data, operation }) => {
    if (operation === 'update' && data.status === 'published') data.publishedAt = new Date()
    return data
  }],

  afterChange: [async ({ doc, req, context }) => {
    if (context.skipNotification) return
    await sendNotification(doc)
    return doc
  }],

  afterRead: [async ({ doc }) => {
    doc.viewCount = await getViewCount(doc.id)
    return doc
  }],

  beforeDelete: [async ({ req, id }) => {
    await req.payload.delete({ collection: 'comments', where: { post: { equals: id } }, req })
  }],
}
```

## Field Hooks

```typescript
const beforeValidate: FieldHook = ({ value }) => value.trim().toLowerCase()
const afterRead: FieldHook = ({ value, req }) =>
  req.user?.roles?.includes('admin') ? value : value.replace(/(.{2})(.*)(@.*)/, '$1***$3')

{ name: 'email', type: 'email', hooks: { beforeValidate: [beforeValidate], afterRead: [afterRead] } }
```

## Context

```typescript
hooks: {
  beforeChange: [async ({ context }) => {
    context.expensiveData = await fetchExpensiveData()
  }],
  afterChange: [async ({ context, doc }) => {
    await processData(doc, context.expensiveData) // Reuse
  }],
}
```

## Next.js Revalidation

```typescript
import { revalidatePath } from 'next/cache'

export const revalidatePage: CollectionAfterChangeHook = ({
  doc,
  previousDoc,
  req: { payload, context },
}) => {
  if (!context.disableRevalidate) {
    if (doc._status === 'published') {
      revalidatePath(doc.slug === 'home' ? '/' : `/${doc.slug}`)
    }
    if (previousDoc?._status === 'published' && doc._status !== 'published') {
      revalidatePath(previousDoc.slug === 'home' ? '/' : `/${previousDoc.slug}`)
    }
  }
  return doc
}
```

## Auto-Set Date

```typescript
{
  name: 'publishedOn',
  type: 'date',
  hooks: {
    beforeChange: [({ siblingData, value }) =>
      siblingData._status === 'published' && !value ? new Date() : value
    ],
  },
}
```

## Best Practices

- `beforeValidate` for formatting
- `beforeChange` for business logic
- `afterChange` for side effects
- `afterRead` for computed fields
- Cache in `context`
- **Pass `req` for transactions**
- **Use context flags to prevent loops**
