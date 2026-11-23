---
icon: list-check
description: >-
  Get the failed tests of a CI run, the history of one test and the history of
  one spec file with the Currents REST API, and get the test and spec file
  signatures these requests need
---

# Failed Tests and Test History

Use these requests to get the failed tests of a CI run, the history of one test across runs, and the history of one spec file. Each section names the endpoint to call; the API reference in the sidebar lists all its parameters and response fields.

The examples use `curl` with an API key in `CURRENTS_API_KEY` (see [Authentication](authentication.md)). Pass `-g` so that `curl` sends the `[]` in parameter names such as `status[]` as written. For the terms run, spec file execution, test execution, test and spec file, see [Terms](../README.md#terms).

## Get the failed tests of a CI run

1. Find the run by its CI build ID:

   ```bash
   curl -g "https://api.currents.dev/v1/runs/find?projectId=$PROJECT_ID&ciBuildId=$CI_BUILD_ID" \
     -H "Authorization: Bearer $CURRENTS_API_KEY"
   ```

   The run ID is `data.runId`. Without `ciBuildId`, pass `branch` to get the last completed run of a branch.
2. List the failed test executions of the run:

   ```bash
   curl -g "https://api.currents.dev/v1/runs/$RUN_ID/test-executions?status[]=failed&limit=100" \
     -H "Authorization: Bearer $CURRENTS_API_KEY"
   ```

   Each row has the test's `title`, `spec`, `displayError` and `attempts`, the `instanceId` and `testId` of the execution, and the `signature` and `specSignature` for the history requests below. Add `include_flaky=true` to also get the flaky tests, or `flaky=true` to get only the flaky tests.
3. To get the screenshots, videos, traces and attachments of one failed test, call `GET /v1/instances/{instanceId}/test-executions/{testId}` with the `instanceId` and `testId` of its row.

## Get a test signature

A test signature identifies one test across runs. Get it in one of these ways:

* Copy `signature` from a row of `GET /v1/runs/{runId}/test-executions`.
* Copy `signature` from a row of `GET /v1/projects/{projectId}/tests`, the list of tests of a project with their metrics.
* Compute it from the project, the spec file path and the test title with `POST /v1/signature/test`:

  ```bash
  curl -X POST "https://api.currents.dev/v1/signature/test" \
    -H "Authorization: Bearer $CURRENTS_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{"projectId": "'$PROJECT_ID'", "specFilePath": "tests/auth/login.spec.ts", "testTitle": ["Login", "signs in with a password"]}'
  ```

  `testTitle` is the title path: the `describe` blocks, then the test name. Copy `specFilePath` and `testTitle` from the `spec` and `title` of a test execution, so that they match what the reporter recorded.

## Get the history of one test

List the executions of one test, newest first:

```bash
curl -g "https://api.currents.dev/v1/projects/$PROJECT_ID/tests/$TEST_SIGNATURE/executions?date_start=2026-11-01&date_end=2026-11-23&branches[]=main" \
  -H "Authorization: Bearer $CURRENTS_API_KEY"
```

`date_start` and `date_end` are required. `date_end` is exclusive, and a date without a time runs up to the next midnight UTC, so `date_end=2026-11-23` includes all of November 23. A value without a time zone is UTC.

Filter the rows with `status[]`, `flaky`, `branches[]`, `tags[]`, `authors[]`, `groups[]` and `environments[]`.

## Get a spec file signature

A spec file signature identifies one spec file across runs. Get it in one of these ways:

* Copy `specSignature` from a row of `GET /v1/runs/{runId}/test-executions`.
* Copy `signature` from a spec file execution: an item of `specs` in `GET /v1/runs/{runId}`, or of `GET /v1/runs/{runId}/instances`.
* Copy `signature` from a row of `GET /v1/projects/{projectId}/spec-files`, the list of spec files of a project with their metrics.
* Compute it from the project and the spec file path with `POST /v1/signature/spec`:

  ```bash
  curl -X POST "https://api.currents.dev/v1/signature/spec" \
    -H "Authorization: Bearer $CURRENTS_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{"projectId": "'$PROJECT_ID'", "specFilePath": "tests/auth/login.spec.ts"}'
  ```

  Send the path as the reporter recorded it, relative to the test root: copy it from the `spec` field of a run or a spec file. The endpoint removes a leading `./`. It answers `404` when the project has no spec file execution recorded under that path, for example when the path starts at the repository root instead of the test root. An OAuth token needs the `results:read` or the `analytics:read` scope.

## Get the history of one spec file

List the executions of one spec file across runs, newest first:

```bash
curl -g "https://api.currents.dev/v1/projects/$PROJECT_ID/spec-files/$SPEC_SIGNATURE/executions?date_start=2026-11-01&date_end=2026-11-23&branches[]=main" \
  -H "Authorization: Bearer $CURRENTS_API_KEY"
```

`date_start` and `date_end` are optional here and follow the same rules as for one test. Filter the rows with `status[]`, `branches[]` and `groups[]`.

One request reads at most 1,000 executions before it applies the filters. When a request stops at that limit, the page can have fewer rows than `limit`, or none, and the response has `next_cursor`. Pass `next_cursor` as `starting_after` to continue. See [Pagination](pagination.md).

## Move from the deprecated routes

The routes on the left still work and are marked deprecated in the API reference. Each one returns the same data as the route on the right.

| Deprecated route                     | Use                                                 |
| ------------------------------------ | --------------------------------------------------- |
| `GET /v1/test-results/{signature}`   | `GET /v1/projects/{projectId}/tests/{signature}/executions` |
| `GET /v1/tests/{projectId}`          | `GET /v1/projects/{projectId}/tests`                |
| `GET /v1/spec-files/{projectId}`     | `GET /v1/projects/{projectId}/spec-files`           |
| `GET /v1/errors/{projectId}`         | `GET /v1/projects/{projectId}/errors`               |
