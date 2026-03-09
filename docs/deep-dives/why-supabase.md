# Why Supabase?

Knowledge graph storage requires either managed or self-hosted Postgres.

## Managed (Supabase)

- **Free Tier:** Sufficient for personal use. After three months of daily use, my storage sits below 2% of the 8GB free tier limit.
- **Features:** Built-in connection pooling, backups, and observability.
- **Extensions:** `pgvector` available by default.
- **Operations:** Minimal maintenance.

## Self-hosted

- **Control:** Full authority over hardware and region.
- **Independence:** No third-party dependencies.
- **Responsibility:** User manages backups, upgrades, and pooling.

## The Choice

Supabase reduces the operational burden of a personal knowledge graph. If usage exceeds free tier limits, the portable schema allows migration to self-hosted Postgres.

## The Service Role Key

The architecture uses the Supabase Service Role key to bypass Row Level Security (RLS). In single-tenant applications, RLS adds unnecessary overhead. The MCP server runs on private infrastructure and requires no per-user access policies. Use the Service Role key for single-tenant environments.
