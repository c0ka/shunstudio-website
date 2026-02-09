---
name: Collections
description: Collection configurations
tags: [payload, collections]
---

# Collections

## Basic

```typescript
export const Posts: CollectionConfig = {
  slug: 'posts',
  admin: { useAsTitle: 'title', defaultColumns: ['title', 'author', 'status', 'createdAt'] },
  fields: [
    { name: 'title', type: 'text', required: true },
    { name: 'slug', type: 'text', unique: true, index: true },
    { name: 'content', type: 'richText' },
    { name: 'author', type: 'relationship', relationTo: 'users' },
  ],
  timestamps: true,
}
```

## Auth with RBAC

```typescript
export const Users: CollectionConfig = {
  slug: 'users',
  auth: true,
  fields: [
    {
      name: 'roles',
      type: 'select',
      hasMany: true,
      options: ['admin', 'editor', 'user'],
      defaultValue: ['user'],
      saveToJWT: true,
      access: { update: ({ req: { user } }) => user?.roles?.includes('admin') },
    },
  ],
}
```

## Upload

```typescript
export const Media: CollectionConfig = {
  slug: 'media',
  upload: {
    staticDir: 'media',
    mimeTypes: ['image/*'],
    imageSizes: [
      { name: 'thumbnail', width: 400, height: 300, position: 'centre' },
      { name: 'card', width: 768, height: 1024 },
    ],
    adminThumbnail: 'thumbnail',
  },
  fields: [{ name: 'alt', type: 'text', required: true }],
}
```

## Drafts

```typescript
export const Pages: CollectionConfig = {
  slug: 'pages',
  versions: {
    drafts: { autosave: true, schedulePublish: true, validate: false },
    maxPerDoc: 100,
  },
  access: {
    read: ({ req: { user } }) => (user ? true : { _status: { equals: 'published' } }),
  },
}

// API
await payload.create({ collection: 'posts', data: { title: 'Draft' }, draft: true })
await payload.findByID({ collection: 'pages', id: '123', draft: true })
```

## Globals

```typescript
export const Header: GlobalConfig = {
  slug: 'header',
  fields: [
    { name: 'logo', type: 'upload', relationTo: 'media' },
    {
      name: 'nav',
      type: 'array',
      fields: [
        { name: 'link', type: 'relationship', relationTo: 'pages' },
        { name: 'label', type: 'text' },
      ],
    },
  ],
}
```
