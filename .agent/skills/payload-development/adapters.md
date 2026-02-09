---
name: Adapters
description: Database, storage, email adapters
tags: [payload, database, transactions]
---

# Adapters

## Database

```typescript
// MongoDB
import { mongooseAdapter } from '@payloadcms/db-mongodb'
db: mongooseAdapter({ url: process.env.DATABASE_URL })

// Postgres
import { postgresAdapter } from '@payloadcms/db-postgres'
db: postgresAdapter({
  pool: { connectionString: process.env.DATABASE_URL },
  migrationDir: './migrations',
})

// SQLite
import { sqliteAdapter } from '@payloadcms/db-sqlite'
db: sqliteAdapter({ client: { url: 'file:./payload.db' }, transactionOptions: {} }) // Enable transactions
```

## Transactions

**CRITICAL**: Pass `req` to nested operations for atomicity.

```typescript
// ✅ CORRECT
afterChange: [
  async ({ doc, req }) => {
    const children = await req.payload.find({
      collection: 'children',
      where: { parent: { equals: doc.id } },
      req,
    })
    for (const child of children.docs) {
      await req.payload.update({
        id: child.id,
        collection: 'children',
        data: { updated: true },
        req,
      })
    }
  },
]

// ❌ WRONG: Missing req breaks transaction
afterChange: [
  async ({ doc, req }) => {
    const children = await req.payload.find({
      collection: 'children',
      where: { parent: { equals: doc.id } },
    })
    for (const child of children.docs) {
      await req.payload.update({ id: child.id, collection: 'children', data: { updated: true } })
    }
  },
]
```

## Storage

```typescript
// S3
import { s3Storage } from '@payloadcms/storage-s3'
plugins: [
  s3Storage({
    collections: { media: true },
    bucket: process.env.S3_BUCKET,
    config: {
      credentials: {
        accessKeyId: process.env.S3_ACCESS_KEY_ID,
        secretAccessKey: process.env.S3_SECRET_ACCESS_KEY,
      },
      region: process.env.S3_REGION,
    },
  }),
]
```

## Email

```typescript
// Nodemailer
import { nodemailerAdapter } from '@payloadcms/email-nodemailer'
email: nodemailerAdapter({
  defaultFromAddress: 'noreply@example.com',
  transportOptions: {
    host: process.env.SMTP_HOST,
    port: 587,
    auth: { user: process.env.SMTP_USER, pass: process.env.SMTP_PASS },
  },
})

// Resend
import { resendAdapter } from '@payloadcms/email-resend'
email: resendAdapter({
  defaultFromAddress: 'noreply@example.com',
  apiKey: process.env.RESEND_API_KEY,
})
```

## Notes

- MongoDB transactions require replica set
- SQLite transactions disabled by default
- Always pass `req` for transaction safety
- Point fields not supported in SQLite
