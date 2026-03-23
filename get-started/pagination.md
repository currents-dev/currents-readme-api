---
icon: forward-step
---

# Pagination

List endpoints return collections in pages. Currents uses **cursor-based** pagination for most resources and **offset-based** pagination for a few aggregations that do not have stable cursors. This page describes both patterns.

## Cursor-based pagination

Use cursor pagination for list methods such as listing projects or runs for a project.

### Parameters

Optional query parameters:

| Parameter        | Description                                                                                                           |
| ---------------- | --------------------------------------------------------------------------------------------------------------------- |
| `limit`          | Maximum number of items in this page. **1–50**, default **10**.                                                       |
| `starting_after` | Cursor from a previous item. Returns the next page of items **older** than that item (see [Sort order](#sort-order)). |
| `ending_before`  | Cursor from a previous item. Returns the next page of items **newer** than that item (see [Sort order](#sort-order)). |

{% hint style="warning" %}
Only one of `starting_after` or `ending_before` may be sent in a single request. They are mutually exclusive.
{% endhint %}

### Response shape

Every list response includes `has_more`. Each item in `data` includes a `cursor` string you pass back as `starting_after` or `ending_before` on the next request.

Example: `GET https://api.currents.dev/v1/projects/bAYZ41/runs`

```json
{
  "status": "OK",
  "has_more": true,
  "data": [
    {
      "runId": "9ee42e4b85c02c634fe30b26d728624e",
      "cursor": "62c538efcbd7fab8a5edb371"
    },
    {
      "runId": "9af7a261f148d1b45d013eec6a22902e",
      "cursor": "62baacbf9f4689a83d2b24cb"
    }
  ]
}
```

When `has_more` is `true`, request the next page using the appropriate cursor parameter (see below).

### Sort order

Picture the full timeline as **newest at the top**, **oldest at the bottom** (like a “latest first” feed):

```text
… newest …
cursor_D
cursor_C
cursor_B   ← reference cursor
cursor_A
… oldest …
```

| Request                                      | What you get                                                                                                                                                 |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Neither `starting_after` nor `ending_before` | First page, in that endpoint’s default order.                                                                                                                |
| `starting_after=cursor_B`                    | Items **older than B** (e.g. `cursor_A`, …)—continuing **down** toward the oldest. The API describes this page as **chronological** order.                   |
| `ending_before=cursor_B`                     | Items **newer than B** (e.g. `cursor_C`, `cursor_D`, …)—continuing **up** toward the newest. The API describes this page as **reverse chronological** order. |

### Example: next page (older items)

After the first request, take the **last** item’s `cursor` in `data` and call again with `starting_after`:

```http
GET /v1/projects/bAYZ41/runs?limit=20&starting_after=62baacbf9f4689a83d2b24cb
```

Stop when `has_more` is `false` or `data` is empty.

### Example: page toward newer items

Use `ending_before` with a cursor from an item **newer** than the slice you want to extend (often the **first** item of the current page when walking back toward the present):

```http
GET /v1/projects/bAYZ41/runs?limit=20&ending_before=62c538efcbd7fab8a5edb371
```

## Offset-based pagination

Some list operations (for example, aggregated lists such as **spec files** for a project) do not map to a stable row cursor. Those endpoints document offset pagination explicitly.

### Response shape

```typescript
{
  "status": "OK",
  "data": {
    // ... resource-specific fields
    "nextPage": number | false,
    "total": number
  }
}
```

| Field      | Meaning                                                              |
| ---------- | -------------------------------------------------------------------- |
| `total`    | Total number of items matching the query (across all pages).         |
| `nextPage` | Page index to request next, or `false` when there are no more pages. |

### Parameters

| Parameter | Description            |
| --------- | ---------------------- |
| `limit`   | Page size. **1–50**.   |
| `page`    | Zero-based page index. |

The slice applied server-side is:

```text
items.slice(page * limit, page * limit + limit)
```

(Equivalent to `items.splice(page * limit, limit)` for a copy of the list.)

### Example

First page:

```http
GET /v1/projects/{projectId}/specs?limit=25&page=0
```

If `data.nextPage` is `1`, fetch the next page with `page=1`, same `limit`, until `nextPage` is `false`.
