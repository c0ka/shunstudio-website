---
name: Plugin Development
description: Creating Payload plugins
tags: [payload, plugins]
---

# Plugin Development

## Architecture

```typescript
import type { Config, Plugin } from 'payload'

interface MyPluginConfig {
  enabled?: boolean
  collections?: string[]
}

export const myPlugin =
  (options: MyPluginConfig): Plugin =>
  (config: Config): Config => ({
    ...config,
    // Transform config
  })
```

## Patterns

```typescript
// Add fields
export const seoPlugin =
  (options: { collections?: string[] }): Plugin =>
  (config: Config): Config => {
    const seoFields: Field[] = [
      { name: 'meta', type: 'group', fields: [{ name: 'title', type: 'text' }] },
    ]
    return {
      ...config,
      collections: config.collections?.map((col) =>
        options.collections?.includes(col.slug)
          ? { ...col, fields: [...col.fields, ...seoFields] }
          : col,
      ),
    }
  }

// Add collections
export const redirectsPlugin =
  (options: { overrides?: Partial<CollectionConfig> }): Plugin =>
  (config: Config): Config => ({
    ...config,
    collections: [
      ...config.collections,
      { slug: 'redirects', fields: [{ name: 'from', type: 'text' }], ...options.overrides },
    ],
  })

// Add hooks
export const nestedDocsPlugin =
  (options: { collections: string[] }): Plugin =>
  (config: Config): Config => ({
    ...config,
    collections: config.collections?.map((col) =>
      options.collections.includes(col.slug)
        ? {
            ...col,
            hooks: {
              ...col.hooks,
              afterChange: [resaveChildrenHook, ...(col.hooks?.afterChange || [])],
            },
          }
        : col,
    ),
  })

// Add endpoints
export const apiPlugin =
  (): Plugin =>
  (config: Config): Config => ({
    ...config,
    endpoints: [
      ...(config.endpoints ?? []),
      { path: '/custom', method: 'get', handler: async () => Response.json({ ok: true }) },
    ],
  })

// onInit
export const seedPlugin =
  (): Plugin =>
  (config: Config): Config => {
    const incomingOnInit = config.onInit
    config.onInit = async (payload) => {
      if (incomingOnInit) await incomingOnInit(payload)
      // Seed data
    }
    return config
  }
```

## Best Practices

```typescript
// Preserve existing
collections: [...(config.collections || []), newCollection]

// User overrides last
const collection = { slug: 'redirects', fields: defaultFields, ...options.overrides }

// Hook composition
hooks: { ...col.hooks, afterChange: [myHook, ...(col.hooks?.afterChange || [])] }
```
