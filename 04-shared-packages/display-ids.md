# `packages/display-ids`

## Paths

| Item | Path |
|------|------|
| Format | `packages/display-ids/src/format.ts` — `formatDisplayId` |
| Prefix map | `packages/display-ids/src/prefixes.ts` — `TABLE_DISPLAY_ID_PREFIXES` |

## Behavior synopsis

**`formatDisplayId`** turns internal numeric/string keys into **human-readable IDs** (e.g. prefixed codes) per table type **`TablesWithDisplayIds`**. Used in UI tables, PDFs, and exports for support readability.

**Features controlled:** Consistent **customer-facing identifiers** across admin and dashboards.
