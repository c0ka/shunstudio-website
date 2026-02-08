---
name: payload-development
description: You are an expert Payload CMS developer. When working with Payload projects, this skill teaches the PRINCIPLES of Payload development and directs you to the right sub-skill based on context.
---

# Sub-Skills

- [Access Control](access-control.md)
- [Access Control Advanced](access-control-advanced.md)
- [Adapters](adapters.md)
- [Collections](collections.md)
- [Configuration](configuration.md)
- [Components](components.md)
- [Endpoints](endpoints.md)
- [Field Type Guards](field-type-guards.md)
- [Fields](fields.md)
- [Hooks](hooks.md)
- [Plugin Development](plugin-development.md)
- [Queries](queries.md)
- [Security Critical](security-critical.md)

# Payload CMS Development Rules

## Core Principles

1. **TypeScript-First**: Always use TypeScript with proper types from Payload
2. **Security-Critical**: Follow all security patterns, especially access control
3. **Type Generation**: Run `generate:types` script after schema changes
4. **Transaction Safety**: Always pass `req` to nested operations in hooks
5. **Access Control**: Understand Local API bypasses access control by default
6. **Access Control**: Ensure roles exist when modifiyng collection or globals with access controls

### Code Validation

- To validate typescript correctness after modifying code run `tsc --noEmit`
- Generate import maps after creating or modifying components.

## Best Practices

### Security

1. Always set `overrideAccess: false` when passing `user` to Local API
2. Field-level access only returns boolean (no query constraints)
3. Default to restrictive access, gradually add permissions
4. Never trust client-provided data
5. Use `saveToJWT: true` for roles to avoid database lookups

### Performance

1. Index frequently queried fields
2. Use `select` to limit returned fields
3. Set `maxDepth` on relationships to prevent over-fetching
4. Use query constraints over async operations in access control
5. Cache expensive operations in `req.context`

### Data Integrity

1. Always pass `req` to nested operations in hooks
2. Use context flags to prevent infinite hook loops
3. Enable transactions for MongoDB (requires replica set) and Postgres
4. Use `beforeValidate` for data formatting
5. Use `beforeChange` for business logic

### Type Safety

1. Run `generate:types` after schema changes
2. Import types from generated `payload-types.ts`
3. Type your user object: `import type { User } from '@/payload-types'`
4. Use `as const` for field options
5. Use field type guards for runtime type checking

### Organization

1. Keep collections in separate files
2. Extract access control to `access/` directory
3. Extract hooks to `hooks/` directory
4. Use reusable field factories for common patterns
5. Document complex access control with comments

## Common Gotchas

1. **Local API Default**: Access control bypassed unless `overrideAccess: false`
2. **Transaction Safety**: Missing `req` in nested operations breaks atomicity
3. **Hook Loops**: Operations in hooks can trigger the same hooks
4. **Field Access**: Cannot use query constraints, only boolean
5. **Relationship Depth**: Default depth is 2, set to 0 for IDs only
6. **Draft Status**: `_status` field auto-injected when drafts enabled
7. **Type Generation**: Types not updated until `generate:types` runs
8. **MongoDB Transactions**: Require replica set configuration
9. **SQLite Transactions**: Disabled by default, enable with `transactionOptions: {}`
10. **Point Fields**: Not supported in SQLite

## Quick Reference

| Task                  | Solution                           |
| --------------------- | ---------------------------------- |
| Auto-generate slugs   | `slugField()`                      |
| Restrict by user      | Access control with query          |
| Local API user ops    | `user` + `overrideAccess: false`   |
| Draft/publish         | `versions: { drafts: true }`       |
| Computed fields       | `virtual: true` with afterRead     |
| Conditional fields    | `admin.condition`                  |
| Custom validation     | `validate` function                |
| Filter relationships  | `filterOptions` on field           |
| Select fields         | `select` parameter                 |
| Auto-set dates        | beforeChange hook                  |
| Prevent loops         | `req.context` check                |
| Cascading deletes     | beforeDelete hook                  |
| Geospatial queries    | `point` field with `near`/`within` |
| Reverse relationships | `join` field type                  |
| Query relationships   | Nested property syntax             |
| Complex queries       | AND/OR logic                       |
| Transactions          | Pass `req` to operations           |
| Background jobs       | Jobs queue with tasks              |
| Custom routes         | Collection custom endpoints        |
| Cloud storage         | Storage adapter plugins            |
| Multi-language        | `localization` + `localized: true` |

## Resources

- Docs: https://payloadcms.com/docs
- LLM Context: https://payloadcms.com/llms-full.txt
- GitHub: https://github.com/payloadcms/payload
- Examples: https://github.com/payloadcms/payload/tree/main/examples
- Templates: https://github.com/payloadcms/payload/tree/main/templates
