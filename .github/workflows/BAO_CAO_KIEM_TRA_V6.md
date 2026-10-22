# BÁO CÁO KIỂM TRA TĨNH V6

- JSON parse/serialize: PASS
- Node count: 139
- PostgreSQL nodes: 31
- Schedule Trigger: 10
- GitHub dispatch nodes: 10
- Manual Trigger: 1
- Connection targets and unique node IDs: PASS
- Code-node JavaScript syntax (`node --check`): PASS
- PostgreSQL-backed history read for all 10 channels: configured
- Atomic PostgreSQL reservation before GitHub dispatch: configured
- Guarded dispatch status updates and callback/orphan audit: configured
- Static-data registry reads/writes: NONE
- Five previously miswired Pexels branches now include a preparation Code node: PASS
- Workflow active: False

## Mandatory setup before run
1. Apply `registry_postgres_schema_v6.sql`.
2. Assign the same PostgreSQL credential to all PostgreSQL nodes.
3. Confirm OpenAI, Pexels, GitHub, and webhook credentials.
4. Keep inactive and test one channel at a time using a non-publishing GitHub workflow.

## Limitations
- No live database/API/GitHub/FB request was executed in this environment.
- High-risk factual channels fail closed unless sources metadata is marked VERIFIED, has at least 2 URLs, and includes an allowlisted domain. The workflow does not automatically fetch/read and independently validate source pages; human/dedicated fact-check approval is still required.
- Pexels candidate relevance is based on search keywords and metadata; no visual model has independently judged the video frames.
- Retry/recovery should use `content_jobs_reconciliation_10ch` and the actual GitHub run status before re-dispatch.
