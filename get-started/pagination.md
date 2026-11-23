---
icon: forward-step
---

# Pagination

List endpoints return their items in pages. Lists of projects, runs, pull requests, spec file executions and test executions page by cursor. The lists of tests, spec files and errors of a project, which aggregate results across runs, page by page number.

## Cursor-based pagination

The cursor lists include:

* `GET /v1/projects` and `GET /v1/projects/{projectId}/runs`
* `GET /v1/projects/{projectId}/pull-requests`
* `GET /v1/runs/{runId}/instances` and `GET /v1/runs/{runId}/test-executions`
* `GET /v1/projects/{projectId}/tests/{signature}/executions` and `GET /v1/projects/{projectId}/spec-files/{signature}/executions`

### Parameters

| Parameter        | Description                                                                                                 |
| ---------------- | ----------------------------------------------------------------------------------------------------------- |
| `limit`          | Maximum number of items in the page. The API reference lists the default and the maximum of each endpoint.  |
| `starting_after` | The `cursor` of the last item of a page. Returns the next page.                                             |
| `ending_before`  | The `cursor` of the first item of a page. Returns the previous page.                                        |

Send either `starting_after` or `ending_before`, not both. Send the same filters on every page.

### Response shape

The response has `has_more`, and each item in `data` has a `cursor`.

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

### Sort order

Most cursor lists are newest first: the next page (`starting_after`) has older items, and the previous page (`ending_before`) has newer items. `GET /v1/runs/{runId}/instances` lists the spec file executions of a run in the order they were created. The API reference states the order of each endpoint.

### Read all the pages

1. Send the first request without a cursor.
2. If `has_more` is `false`, stop.
3. Send the next request with `starting_after` set to the `cursor` of the last item in `data`.

```http
GET /v1/projects/bAYZ41/runs?limit=20&starting_after=62baacbf9f4689a83d2b24cb
```

### Pages cut short by a read limit

`GET /v1/runs/{runId}/instances` and `GET /v1/projects/{projectId}/spec-files/{signature}/executions` read at most 1,000 executions per request, before they apply the filters. When a request stops at that limit, the page can have fewer items than `limit`, or none, `has_more` is `true`, and the response has `next_cursor`. Continue with `starting_after=<next_cursor>`, or `ending_before=<next_cursor>` when you page back, instead of the cursor of the last item.

## Offset-based pagination

These lists aggregate results across runs and page by page number:

* `GET /v1/projects/{projectId}/tests`
* `GET /v1/projects/{projectId}/spec-files`
* `GET /v1/projects/{projectId}/errors`

### Parameters

| Parameter | Description                                                         |
| --------- | ------------------------------------------------------------------- |
| `limit`   | Page size. The API reference lists the default and the maximum.     |
| `page`    | Zero-based page index. Default: `0`.                                |

### Response shape

```json
{
  "status": "OK",
  "data": {
    "list": [],
    "total": 77,
    "nextPage": 1
  }
}
```

| Field      | Meaning                                                              |
| ---------- | -------------------------------------------------------------------- |
| `list`     | The items of this page.                                              |
| `total`    | Number of items that match the request, across all pages.            |
| `nextPage` | Page index to request next, or `false` when there are no more pages. |

### Example

```http
GET /v1/projects/{projectId}/spec-files?limit=25&page=0&date_start=2026-11-01&date_end=2026-11-23
```

If `data.nextPage` is `1`, send the same request with `page=1`, until `nextPage` is `false`.
