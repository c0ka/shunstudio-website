---
name: Field Type Guards
description: Runtime field type checking
tags: [payload, typescript, type-guards]
---

# Field Type Guards

## Common Guards

```typescript
import { fieldAffectsData, fieldHasSubFields, fieldIsArrayType } from 'payload'

// Most common - checks if field stores data
if (fieldAffectsData(field)) {
  schema[field.name] = getFieldType(field) // Safe to access field.name
}

// Has nested fields
if (fieldHasSubFields(field)) {
  traverseFields(field.fields) // Safe to access field.fields
}

// Is array
if (fieldIsArrayType(field)) {
  console.log(field.minRows, field.maxRows)
}
```

## Patterns

```typescript
// Recursive traversal
function traverseFields(fields: Field[], callback: (field: Field) => void) {
  fields.forEach((field) => {
    if (fieldAffectsData(field)) callback(field)
    if (fieldHasSubFields(field)) traverseFields(field.fields, callback)
  })
}

// Filter data fields
const dataFields = fields.filter(fieldAffectsData)

// Type switching
if (fieldIsArrayType(field)) {
  // Array
} else if (fieldIsBlockType(field)) {
  // Blocks
} else if (fieldHasSubFields(field)) {
  // Group/row/collapsible
}
```

## All Guards

| Guard                       | Checks                     |
| --------------------------- | -------------------------- |
| `fieldAffectsData`          | Stores data (has name)     |
| `fieldHasSubFields`         | Contains nested fields     |
| `fieldIsArrayType`          | Is array                   |
| `fieldIsBlockType`          | Is blocks                  |
| `fieldIsGroupType`          | Is group                   |
| `fieldSupportsMany`         | Can have multiple values   |
| `fieldHasMaxDepth`          | Supports depth control     |
| `fieldIsPresentationalOnly` | UI-only                    |
| `fieldIsVirtual`            | Virtual field              |
| `tabHasName`                | Named tab                  |
| `optionIsObject`            | Option is `{label, value}` |
