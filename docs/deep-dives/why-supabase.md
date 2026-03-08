# Why Supabase?

You need a database to store the knowledge graph. The two options: managed Postgres or self-hosted.

## Managed (Supabase)

- Free tier handles personal use easily.
- Built-in connection pooling, backups, and observability.
- `pgvector` extension available out of the box.
- Dashboard for manual queries and debugging.

## Self-hosted

- Full control over hardware and region.
- No dependency on a third-party provider.
- You handle backups, upgrades, and connection pooling.

## Why I chose Supabase

For a personal tool, the free tier removes all operational burden. I have not hit the limits. If I did, I could migrate the data to a self-hosted Postgres instance in an afternoon -- the schema is portable.

## On the Service Role Key

This architecture uses the Supabase Service Role key, which bypasses Row Level Security (RLS). This is deliberate.

RLS exists to enforce per-user access policies in multi-tenant applications. Here, there is one user: you. The MCP server runs on your infrastructure and talks directly to the database. Enforcing RLS would add overhead with no security benefit.

If you expose the database to other clients or users, you should implement RLS policies. For the single-tenant case, the Service Role key is the correct choice.
