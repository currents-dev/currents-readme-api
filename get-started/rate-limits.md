---
icon: gauge-high
---

# Rate Limits

The REST API limits how many requests each API key and each organization can send per minute. A request over a limit gets `429 Too Many Requests` and is not processed.

## Limits

| Endpoints | Per API key | Per organization |
| --- | --- | --- |
| `GET /v1/test-results/{signature}` | 3,000 per minute | 5,000 per minute |
| `GET /v1/instances/{instanceId}` | 3,000 per minute | 5,000 per minute |
| All other endpoints | 1,000 per minute | 2,000 per minute |

* Each request counts against one row only. Requests to `GET /v1/test-results/{signature}` don't use up the budget of the other endpoints.
* The organization limit is shared by all API keys and API tokens of the organization.
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
