---
icon: terminal
metaLinks:
  alternates:
    - https://app.gitbook.com/s/M9ty6FYa3j98VSBHF9LN/
---

# Intro

The Currents API allows programmatic access to the data created during the execution of your tests.

The API is organized around [REST](http://en.wikipedia.org/wiki/Representational_State_Transfer). Our API has resource-oriented URLs, returns [JSON-encoded](http://www.json.org/) responses, and uses standard HTTP response codes, authentication, and verbs.

The base URL is `https://api.currents.dev/v1`. Every request needs a credential, see [Authentication](get-started/authentication.md).

## Terms

* Run: one execution of a test suite, with ID `runId`.
* Spec file execution: one spec file in one run, also called an instance, with ID `instanceId`.
* Test execution: one test in one spec file execution, with ID `testId`.
* Test and spec file: one test or one spec file across runs. Each has a `signature`: a test signature or a spec file signature.

Paths and fields use `instance`, `instanceId` and `specs` for spec file executions.

To get the failed tests of a run or the history of a test or a spec file, see [Failed Tests and Test History](get-started/test-results.md). The sidebar lists every endpoint with its parameters and responses.
