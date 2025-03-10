---
navigation_title: Node.js
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/timeout-best-practices.html
---

# Troubleshoot {{es}} Node.js client [timeout-best-practices]

Starting in 9.0.0, this client is configured to not time out any HTTP request by default. {{es}} will always eventually respond to any request, even if it takes several minutes. Reissuing a request that it has not responded to yet can cause performance side effects. See the [official {{es}} recommendations for HTTP clients](asciidocalypse://docs/elasticsearch/docs/reference/elasticsearch/configuration-reference/networking-settings.md#_http_client_configuration) for more information.

Prior to 9.0, this client was configured by default to operate like many HTTP client libraries do, by using a relatively short (30 second) timeout on all requests sent to {{es}}, raising a `TimeoutError` when that time period elapsed without receiving a response.

If your circumstances require you to set timeouts on Elasticsearch requests, setting the `requestTimeout` value to a millisecond value will cause this client to operate as it did prior to 9.0.

