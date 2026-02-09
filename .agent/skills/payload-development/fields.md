---
name: Fields
description: Field types and patterns
tags: [payload, fields]
---

# Fields

## Common Patterns

```typescript
import { slugField } from 'payload'

// Auto-slug
slugField({ fieldToUse: 'title' })

// Filtered relationship
{ name: 'category', type: 'relationship', relationTo: 'categories', filterOptions: { active: { equals: true } } }

// Conditional
{ name: 'featuredImage', type: 'upload', relationTo: 'media', admin: { condition: (data) => data.featured } }

// Virtual
{ name: 'fullName', type: 'text', virtual: true, hooks: { afterRead: [({ siblingData }) => `${siblingData.firstName} ${siblingData.lastName}`] } }
```

## Types

```typescript
// Text
{ name: 'title', type: 'text', required: true, unique: true, minLength: 5, maxLength: 100, index: true }

// Rich Text
import { lexicalEditor, HeadingFeature } from '@payloadcms/richtext-lexical'
{ name: 'content', type: 'richText', editor: lexicalEditor({ features: ({ defaultFeatures }) => [...defaultFeatures, HeadingFeature()] }) }

// Relationship
{ name: 'author', type: 'relationship', relationTo: 'users', maxDepth: 2 }
{ name: 'categories', type: 'relationship', relationTo: 'categories', hasMany: true }
{ name: 'related', type: 'relationship', relationTo: ['posts', 'pages'], hasMany: true } // Polymorphic

// Array
{ name: 'slides', type: 'array', minRows: 2, maxRows: 10, fields: [{ name: 'title', type: 'text' }] }

// Blocks
const HeroBlock: Block = { slug: 'hero', fields: [{ name: 'heading', type: 'text' }] }
{ name: 'layout', type: 'blocks', blocks: [HeroBlock] }

// Select
{ name: 'status', type: 'select', options: ['draft', 'published'], defaultValue: 'draft' }
{ name: 'tags', type: 'select', hasMany: true, options: ['tech', 'news'] }

// Point
{ name: 'location', type: 'point' }
// Query: where: { location: { near: [10, 20], maxDistance: 5000 } }

// Join (reverse relationship)
{ name: 'orders', type: 'join', collection: 'orders', on: 'customer' }

// Tabs
{ type: 'tabs', tabs: [{ label: 'Content', fields: [...] }, { label: 'SEO', fields: [...] }] }

// Group
{ name: 'meta', type: 'group', fields: [{ name: 'title', type: 'text' }] }
```

## Validation

```typescript
{
  name: 'email',
  type: 'email',
  validate: (value, { operation }) => {
    if (operation === 'create' && !value) return 'Required'
    if (value && !value.includes('@')) return 'Invalid'
    return true
  },
}
```
