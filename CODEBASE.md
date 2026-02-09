# Codebase Overview

Next.js 16 + Payload CMS 3.75 | React 19 | TypeScript 5.9 | TailwindCSS 4 | PostgreSQL

## File Dependencies

### Config Layer

- `payload.config.ts` → imports all collections, globals, plugins
- `plugins/index.ts` → configures redirects, seo, forms, search, s3
- `next.config.js` → Next.js config with Payload integration

### Collections

- `Pages` → access/_, blocks (Archive, CTA, Content, Form, MediaBlock), heros/_
- `Posts` → access/\*, blocks (Banner, Code, MediaBlock), relationships (Categories, Users, Media)
- `Media` → upload handling, S3 in production
- `Categories` → nested docs plugin
- `Users` → authentication

### Blocks (config → component)

- `ArchiveBlock` | `CallToAction` | `Content` | `Form` | `MediaBlock` | `Banner` | `Code`
- All rendered via `RenderBlocks.tsx`

### Frontend Pages

- `app/(frontend)/[slug]/page.tsx` → RenderHero + RenderBlocks
- `app/(frontend)/posts/[slug]/page.tsx` → Post content + RelatedPosts

## Data Flow

### 1. Content Creation (Admin → Database)

```
Admin UI → Collection Config → Hooks (beforeChange) → Database
                                  ↓
                            populatePublishedAt
```

### 2. Content Delivery (Database → Frontend)

```
Next.js Page → getPayload() → payload.find() → Database
     ↓
queryPageBySlug (cached)
     ↓
Page Component → RenderHero + RenderBlocks → Block Components
```

### 3. Revalidation (On Content Change)

```
afterChange hook → revalidatePage/revalidatePost → Next.js revalidatePath/revalidateTag
```

## Key Dependency Chains

| Entry Point             | Dependencies                                           |
| ----------------------- | ------------------------------------------------------ |
| `payload.config.ts`     | collections/_, globals/_, plugins/_, fields/_          |
| `collections/Pages`     | access/_, blocks/_, hooks/_, utilities/_, heros/\*     |
| `collections/Posts`     | access/_, blocks/_, hooks/_, utilities/_               |
| `app/(frontend)/[slug]` | blocks/RenderBlocks, heros/RenderHero, components/\*   |
| `plugins/index.ts`      | hooks/revalidateRedirects, search/\*, utilities/getURL |

## Access Control Flow

```
Request → Collection Access Check → Field Access Check → Response
              ↓                          ↓
         authenticated()           (field-level)
    authenticatedOrPublished()
```

- `authenticated`: Requires logged-in user
- `authenticatedOrPublished`: Public read for published, auth for drafts

## Plugin Dependencies

| Plugin      | Collections  | Hooks                      |
| ----------- | ------------ | -------------------------- |
| redirects   | pages, posts | revalidateRedirects        |
| nestedDocs  | categories   | —                          |
| seo         | pages, posts | generateTitle, generateURL |
| formBuilder | forms        | —                          |
| search      | posts        | beforeSyncWithSearch       |
| s3Storage   | media        | — (prod only)              |

## Block → Component Mapping

| Block Type | Config              | Component              |
| ---------- | ------------------- | ---------------------- |
| archive    | ArchiveBlock/config | ArchiveBlock/Component |
| content    | Content/config      | Content/Component      |
| cta        | CallToAction/config | CallToAction/Component |
| formBlock  | Form/config         | Form/Component         |
| mediaBlock | MediaBlock/config   | MediaBlock/Component   |
| banner     | Banner/config       | Banner/Component       |
| code       | Code/config         | Code/Component         |

## Collection Relationships

```
Posts ──→ categories (hasMany)
      ──→ authors → Users (hasMany)
      ──→ relatedPosts → Posts (self-ref)
      ──→ heroImage → Media
      ──→ meta.image → Media

Pages ──→ layout[] → Blocks → Media
      ──→ hero → Media
      ──→ meta.image → Media
```

## Commands

```bash
pnpm dev              # Dev server
pnpm build            # Production build
pnpm generate:types   # Regenerate payload-types.ts
pnpm generate:importmap # After component changes
```
