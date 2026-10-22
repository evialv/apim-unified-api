# apim-unified

The Unified AI API for Azure API Management: one wildcard endpoint (`/unified-ai/*`) that accepts several model-request formats and routes each request to the right model backend.

## How a request flows

```
client ──► https://<apim>/unified-ai/<format-prefix>/...
              │
              ├─ metadata-config            format table + model settings (generated)
              ├─ central-cache-manager      caches that configuration
              ├─ request-processor          path → format; body / path / header → model
              ├─ security-handler           API key + optional JWT
              ├─ responses-id-security      Responses API: callers only reach their own IDs
              ├─ validate-model-access      model allowed for this product?
              ├─ set-backend-pools          routing table (generated)
              ├─ strip-backend-headers      drop browser / forwarding headers
              ├─ set-target-backend-pool    model + allowed backend types → pool / backend
              ├─ resolve-model-alias        alias → real model name in the body
              ├─ set-backend-authorization  managed identity / provider key; set backend
              ├─ path-builder               rewrite URL for that format + backend type
              ├─ set-llm-usage              token metrics
              └─ backend: retry 429/5xx (alias fallback); outbound: response headers, alerts
```

## Files

| Read | File | What it does |
|---|---|---|
| 1 | [policies/unified-ai-api-policy.xml](policies/unified-ai-api-policy.xml) | The whole pipeline, step by step (start here) |
| 2 | [policies/frag-request-processor.xml](policies/frag-request-processor.xml) | Format detection by path prefix; finding the model |
| 3 | [policies/frag-metadata-config.xml](policies/frag-metadata-config.xml) | Format table (`api-types`), backend path templates |
| 4 | [policies/frag-set-target-backend-pool.xml](policies/frag-set-target-backend-pool.xml) | Choosing the backend for a model + format; aliases |
| 5 | [policies/frag-path-builder.xml](policies/frag-path-builder.xml) | URL rewrite per format and backend type |
| 6 | [policies/frag-set-backend-authorization.xml](policies/frag-set-backend-authorization.xml) | Backend authentication (managed identity, API keys, AWS SigV4) |
| — | [policies/frag-*.xml](policies/) | The remaining steps of the pipeline |
| — | [policies/unified-ai-api-deployments-policy.xml](policies/unified-ai-api-deployments-policy.xml), [policies/unified-ai-api-deployment-by-name-policy.xml](policies/unified-ai-api-deployment-by-name-policy.xml) | `GET /deployments`: model discovery |
| 7 | [unified-ai-api.bicep](unified-ai-api.bicep), [unified-ai-api/UnifiedAIWildcard.json](unified-ai-api/UnifiedAIWildcard.json) | The wildcard API and its operations |
| 8 | [llm-backends.bicep](llm-backends.bicep) | One APIM backend per model endpoint, with credentials |
| 9 | [llm-backend-pools.bicep](llm-backend-pools.bicep) | Load-balanced pools per (model, backend type) |
| 10 | [llm-policy-fragments.bicep](llm-policy-fragments.bicep) | Generates the routing configuration into the fragments |
| 11 | [policy-fragments.bicep](policy-fragments.bicep) | Deploys the fixed fragments |
| 12 | [apim.bicep](apim.bicep) | Wires the modules together |
| — | [README-llm-backends.md](README-llm-backends.md) | Backend configuration examples |

## Note

These files are for reading and reuse. They aren't a standalone deployment: the Bicep modules reference files that aren't included here, such as other APIs, networking modules and `release.json`.

## License

MIT; see [LICENSE](LICENSE).
