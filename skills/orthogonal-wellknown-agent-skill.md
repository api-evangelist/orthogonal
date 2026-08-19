---
name: Orthogonal
description: Use when searching for and calling paid APIs (lead enrichment, web scraping, email finding, AI search, etc.) without managing individual provider accounts. Agents should reach for this skill when users ask to enrich data, find contact information, scrape websites, search the web, or chain multiple APIs into workflows.
metadata:
    mintlify-proj: orthogonal
    version: "1.0"
---

# Orthogonal Skill

## Product summary

Orthogonal is a unified API gateway that lets agents search and call hundreds of paid APIs (Apollo, Hunter, LinkUp, Olostep, Shofo, etc.) with a single API key and credit account. No per-provider authentication needed. Agents use Orthogonal to enrich leads, find emails, scrape websites, search the web, monitor social media, and chain multiple APIs into complete workflows. 

**Key files and commands:**
- API base: `https://api.orthogonal.com/v1`
- CLI: `orth search`, `orth run`, `orth api`, `orth balance`
- Auth: Bearer token in `Authorization` header
- MCP server: `https://mcp.orthogonal.com` (for Claude, Cursor)
- Primary docs: https://docs.orthogonal.com

## When to use

Reach for this skill when:
- User asks to enrich leads, find emails, or get contact information (Apollo, Hunter, Tomba)
- User wants to scrape a website or extract structured data (Olostep, Riveter)
- User needs to search the web or monitor social media (LinkUp, Shofo, Tavily)
- User wants to research a company or person (brand data, LinkedIn profiles, news)
- User needs to chain multiple APIs into one workflow (research → enrich → find contacts)
- User asks for competitor intelligence, account-based marketing research, or prospecting

Do NOT use this skill for:
- Tasks that don't require external APIs (local data processing, file manipulation)
- Authenticating with individual API providers directly
- Managing API keys for multiple services

## Quick reference

### Core API endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/v1/search` | POST | Find APIs by natural language description |
| `/v1/run` | POST | Execute an API call (main endpoint) |
| `/v1/details` | POST | Get full parameter info for an endpoint |
| `/v1/integrate` | POST | Get code snippets for an endpoint |
| `/v1/list-endpoints` | GET | List all available APIs |
| `/v1/credits/balance` | GET | Check account balance |

### CLI commands

```bash
orth search "enrich lead find email"          # Search for APIs
orth api apollo                                # List endpoints for an API
orth api apollo /v1/people/match              # Get endpoint details
orth run apollo /v1/people/match --body '{...}'  # Execute API call
orth balance                                   # Check credit balance
orth account                                   # View account info
```

### Authentication

All requests require Bearer token:
```bash
Authorization: Bearer orth_live_xxxxxxxxxxxx
```

Set as environment variable:
```bash
export ORTHOGONAL_API_KEY=orth_live_your_key
```

### Common API slugs and endpoints

| API | Slug | Common Endpoint | Use Case |
|-----|------|-----------------|----------|
| Apollo.io | `apollo` | `/v1/people/match` | Lead enrichment by email/name |
| Hunter.io | `hunter` | `/domain-search` | Find emails at a company |
| LinkUp | `linkup` | `/search` | AI-powered web search |
| Olostep | `olostep` | `/v1/scrapes` | Web scraping to markdown |
| Shofo | `shofo` | `/linkedin/company-posts` | Social media monitoring |
| Tomba | `tomba` | `/v1/email-verifier` | Email verification |
| Fiber | `fiber` | `/v1/natural-language-search/profiles` | People search by criteria |
| Brand.dev | `brand-dev` | `/v1/brand/retrieve` | Company brand data |

## Decision guidance

### When to use CLI vs API vs MCP

| Scenario | Use | Why |
|----------|-----|-----|
| Quick testing, one-off calls | CLI (`orth run`) | Fastest to type, no code needed |
| Building an agent integration | MCP server | Automatic tool discovery, no auth handling |
| Programmatic workflows, scripts | API (direct HTTP) | Full control, easy to chain calls |
| Integrating into existing app | API (SDK/library) | Consistent with app architecture |

### When to search vs list vs details

| Goal | Use | Why |
|------|-----|-----|
| Find APIs for a task ("enrich leads") | `/v1/search` | Semantic matching, returns best options |
| Browse all available APIs | `/v1/list-endpoints` | Get complete catalog with pagination |
| Understand endpoint parameters | `/v1/details` | Full schema before making a call |

### Single API vs multi-API workflow

| Scenario | Approach | Example |
|----------|----------|---------|
| One task (find email) | Single API call | `hunter /domain-search` |
| Complete workflow (research + enrich + find contacts) | Chain 2-4 APIs | LinkUp → Hunter → Apollo |
| Complex extraction (news → structured data) | Chain with Riveter | LinkUp → Riveter |

## Workflow

### Typical task: Enrich a lead by email

1. **Search for enrichment APIs**
   ```bash
   orth search "enrich lead by email"
   ```
   Returns: Apollo, Hunter, Tomba, etc. with endpoints and prices.

2. **Get endpoint details** (optional, to understand parameters)
   ```bash
   orth api apollo /v1/people/match
   ```
   Shows required/optional fields, price, example usage.

3. **Make the API call**
   ```bash
   orth run apollo /v1/people/match --body '{"email": "ceo@stripe.com"}'
   ```
   Returns: Enriched person data + cost in cents.

4. **Check the response**
   - Verify `success: true`
   - Review `priceCents` to confirm cost
   - Extract data from `data` field

### Typical task: Chain APIs for lead research

1. **Research the company** (LinkUp AI search)
   ```bash
   orth run linkup /search --body '{"q": "stripe company funding 2026"}'
   ```

2. **Find contacts at the company** (Hunter)
   ```bash
   orth run hunter /domain-search --body '{"domain": "stripe.com"}'
   ```

3. **Enrich a contact** (Apollo)
   ```bash
   orth run apollo /v1/people/match --body '{"email": "founder@stripe.com"}'
   ```

4. **Combine results** into a single report.

### Typical task: Use via MCP in Claude

1. **Add MCP server** to Claude Desktop settings: `https://mcp.orthogonal.com`
2. **Ask Claude**: "Search Orthogonal for APIs to enrich leads by email, then enrich patrick@stripe.com"
3. **Claude automatically**:
   - Calls `search` tool
   - Calls `get_details` to understand parameters
   - Calls `use` to execute the enrichment
   - Returns results

## Common gotchas

- **Missing API key**: Set `ORTHOGONAL_API_KEY` env var or pass `--key` flag. Requests without auth return 401.
- **Insufficient credits**: Check balance with `orth balance` before running expensive calls. Calls fail with 402 if balance too low.
- **Wrong API slug or path**: Search results return the exact `slug` and `path` to use. Copy them exactly; typos return 404.
- **Forgetting to pass body/query params**: Check endpoint details with `/v1/details` to see required fields. Missing required params return 400.
- **Assuming all endpoints are free**: Every endpoint has a price. Check search results or `/v1/details` before calling. Prices range from free to $1+.
- **Not checking `verified` flag**: Unverified endpoints may be outdated. Prefer endpoints with `"verified": true` in search results.
- **Chaining APIs without error handling**: If step 1 fails, step 2 has no data. Always check `success: true` before using response data.
- **Using test keys in production**: Test keys (`orth_test_`) have limited APIs and no charges. Use live keys (`orth_live_`) for real work.
- **Forgetting to extract the right field**: Response structure is `{ success, priceCents, data, requestId }`. The actual result is in `data`, not at the root.
- **Rate limiting on bulk operations**: Orthogonal has rate limits. For 20+ calls, use `batch_use` endpoint instead of looping.

## Verification checklist

Before submitting work:

- [ ] API key is set and valid (test with `orth balance`)
- [ ] Account has sufficient credits for the task
- [ ] Search results show `verified: true` endpoints (preferred)
- [ ] Endpoint path and slug match exactly (copy from search results)
- [ ] Required parameters are included in request body
- [ ] Response includes `success: true`
- [ ] Cost (`priceCents`) is reasonable and expected
- [ ] Data in response `data` field is complete and correct
- [ ] If chaining APIs, each step checks `success` before proceeding
- [ ] No hardcoded API keys in code (use env vars)

## Resources

- **Full page navigation**: https://docs.orthogonal.com/llms.txt
- **API Reference**: https://docs.orthogonal.com/api-reference/introduction
- **CLI Reference**: https://docs.orthogonal.com/cli
- **MCP Setup**: https://docs.orthogonal.com/mcp/setup
- **Use Cases & Examples**: https://docs.orthogonal.com/use-cases

---

> For additional documentation and navigation, see: https://docs.orthogonal.com/llms.txt