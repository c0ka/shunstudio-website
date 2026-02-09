---
name: coding-standards
description: TypeScript/JavaScript coding standards and best practices for clean, maintainable code
---

# Coding Standards

## Immutability (CRITICAL)

**NEVER mutate, ALWAYS create new:**

```typescript
// ❌ user.name = name; items.push(item)
// ✅ {...user, name}; [...items, item]

// Arrays: filter/map/reduce instead of mutating
items.filter((i) => i.id !== id) // remove
items.map((i) => (i.id === id ? { ...i, ...u } : i)) // update
```

## File Organization

- **200-400 lines typical, 800 max**
- Organize by feature, not type
- High cohesion, low coupling

## Error Handling

```typescript
try {
  const user = await api.getUser(id)
  if (!user) throw new Error(`User not found: ${id}`)
  return user
} catch (e) {
  throw new Error(`Failed to fetch user ${id}: ${e instanceof Error ? e.message : e}`)
}
```

## Input Validation (Zod)

```typescript
const Schema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150),
})
const data = Schema.parse(input) // throws on invalid
// or: Schema.safeParse(input) for {success, data/error}
```

## Function Design

- **< 50 lines**, single responsibility
- Max 3-4 params (use objects for more)
- Pure when possible

## Naming

| Type                | Convention    | Example                     |
| ------------------- | ------------- | --------------------------- |
| Variables/Functions | camelCase     | `userName`, `getUserById`   |
| Types/Interfaces    | PascalCase    | `User`, `UserRepository`    |
| Constants           | UPPER_SNAKE   | `MAX_RETRIES`               |
| Booleans            | is/has/should | `isActive`, `hasPermission` |
| Handlers            | handle/on     | `handleClick`, `onSubmit`   |
| Hooks               | use           | `useAuth`                   |

## Patterns

```typescript
// Early returns (avoid nesting)
if (!user) return null
if (!user.isActive) return null

// Optional chaining + nullish coalescing
const city = user?.address?.city ?? 'Unknown'

// Array methods > loops
users.filter((u) => u.isActive).map((u) => u.name)
```

## Checklist

- [ ] Readable, descriptive names
- [ ] Functions < 50 lines, files < 800 lines
- [ ] No nesting > 4 levels
- [ ] Errors handled with context
- [ ] No console.log, no hardcoded values
- [ ] Immutable patterns, no `any`
- [ ] User input validated with Zod
