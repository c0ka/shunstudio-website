---
name: Components
description: Custom React components for Admin Panel
tags: [payload, components, react]
---

# Custom Components

## Component Paths

```typescript
export default buildConfig({
  admin: {
    components: {
      logout: { Button: '/src/components/Logout#MyComponent' }, // Named
      Nav: '/src/components/Nav', // Default
    },
  },
})
```

**Rules**: Relative to `baseDir`, named exports use `#ExportName`, default exports no suffix

## Server vs Client

```tsx
// Server (default) - async, Local API access
async function MyServer({ payload }: { payload: Payload }) {
  const page = await payload.findByID({ collection: 'pages', id: '123' })
  return <p>{page.title}</p>
}

// Client - state, effects, handlers
;('use client')
export function MyClient() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

## Root Components

| Component | Path                             |
| --------- | -------------------------------- |
| Nav       | `admin.components.Nav`           |
| Logo      | `admin.components.graphics.Logo` |
| Icon      | `admin.components.graphics.Icon` |
| Logout    | `admin.components.logout.Button` |
| Actions   | `admin.components.actions`       |
| Providers | `admin.components.providers`     |

## Field Components

```tsx
// Edit View
'use client'
import { useField } from '@payloadcms/ui'

export const StatusField = ({ path, field }) => {
  const { value, setValue } = useField({ path })
  return (
    <select value={value} onChange={(e) => setValue(e.target.value)}>
      {field.options.map((opt) => <option key={opt.value} value={opt.value}>{opt.label}</option>)}
    </select>
  )
}

// List View (Cell)
export const StatusCell = ({ cellData }) => (
  <span style={{ color: cellData === 'published' ? 'green' : 'orange' }}>{cellData}</span>
)

// UI Field (presentational)
{ name: 'refundButton', type: 'ui', admin: { components: { Field: '/components/RefundButton' } } }
```

## Hooks

```tsx
'use client'
import {
  useAuth, // Current user
  useConfig, // Config (client-safe)
  useDocumentInfo, // Doc (id, slug)
  useField, // Field value/setValue
  useFormFields, // Multiple fields (optimized)
  useLocale, // Current locale
  useTranslation, // i18n
  usePayload, // Local API
} from '@payloadcms/ui'
```

## Patterns

```tsx
// Conditional field
'use client'
import { useFormFields } from '@payloadcms/ui'

export const ConditionalField = ({ path }) => {
  const show = useFormFields(([fields]) => fields.enableFeature?.value)
  if (!show) return null
  return <input type="text" />
}

// Server Component with Local API
async function RelatedPosts({ payload, id }) {
  const post = await payload.findByID({ collection: 'posts', id, depth: 0 })
  const related = await payload.find({
    collection: 'posts',
    where: { category: { equals: post.category }, id: { not_equals: id } },
    limit: 5,
  })
  return (
    <ul>
      {related.docs.map((doc) => (
        <li key={doc.id}>{doc.title}</li>
      ))}
    </ul>
  )
}
```

## Performance

```tsx
// ❌ Re-renders on every form change
const { fields } = useForm()

// ✅ Only re-renders when specific field changes
const value = useFormFields(([fields]) => fields[path])
```

**Use Server Components when possible** - no JS to client. Client only for state/effects/handlers/browser APIs.

## Import Map

```bash
payload generate:importmap  # Regenerate after component changes
```
