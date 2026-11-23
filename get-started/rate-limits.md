---
icon: gauge-high
---

# Rate Limits

The REST API limits how many requests each API key and each organization can send per minute. A request over a limit gets `429 Too Many Requests` and is not processed.

## Limits

| Requests | Per API key | Per organization |
| --- | --- | --- |
| The executions of one test: `GET /v1/projects/{projectId}/tests/{signature}/executions`, `GET /v1/test-results/{signature}` and `POST /v1/instances/{instanceId}/trace-link` | 1,000 per minute | 2,000 per minute |
| One spec file: `GET /v1/instances/{instanceId}` and `GET /v1/projects/{projectId}/spec-files/{signature}/executions` | 1,000 per minute | 2,000 per minute |
| Test, spec file and error lists and metrics: `GET /v1/projects/{projectId}/tests`, `/spec-files` and `/errors`, their `/metrics` and `/{signature}` routes, and `GET` on `/v1/tests/{projectId}`, `/v1/errors/{projectId}` and `/v1/spec-files/{projectId}` | 200 per minute | 400 per minute |
| Signatures: `POST /v1/signature/test` and `POST /v1/signature/spec` | 3,000 per minute | 6,000 per minute |
| Other requests for a project: `GET /v1/projects/...` and `POST /v1/projects/{projectId}/test-issues` | 600 per minute | 1,200 per minute |
| Other `GET` requests, including `/v1/runs/{runId}/instances`, `/v1/runs/{runId}/test-executions` and `/v1/instances/{instanceId}/test-executions/{testId}` | 600 per minute | 1,200 per minute |
| Other `POST` requests, and `PUT`, `PATCH` and `DELETE` requests | 300 per minute | 600 per minute |

* Each request counts against one row only. Reading the history of many tests one by one doesn't use up the budget of the other rows.
* Personal access tokens and OAuth tokens count per person: all of one person's tokens share the per-key limit.
* The organization limit is shared by all API keys and tokens of the organization.
* Each limit counts requests in fixed one-minute windows.

## When you hit a limit

The response is `429` with a `Retry-After` header, the number of seconds until the window resets:

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 12

{
  "status": "FAILED",
  "error": "Too many requests. Retry in 12 seconds.",
  "code": "rate_limited"
}
```

Wait for `Retry-After` seconds before you send the next request.

## Staying under the limits

* Don't poll a run while it's in progress more often than you need. The [HTTP webhooks](https://docs.currents.dev/resources/integrations/http-webhooks) and [GitHub](https://docs.currents.dev/resources/integrations/github) integrations notify you when a run finishes.
* Fetch lists with the list endpoints and pagination instead of one request per item, where a list endpoint has the data you need.
* Spread bulk jobs, such as a backfill over many tests, across several minutes.
* Each list and metrics request computes its result over the whole date range. Request the date range and filters you need, and page through the list, instead of repeating the same request.
