# 02. Database Provisioning

The memory system uses a graph structure mapped to relational tables:
- **Entities:** Nodes (e.g., "React", "Desktop", "Dog").
- **Observations:** Facts (e.g., "Is a framework", "Uses 4k monitor").
- **Relations:** Edges connecting entities.

Semantic search requires a Postgres database with the `pgvector` extension. This guide uses Supabase.

## 1. Create the Project

1. Create a new project on [Supabase](https://supabase.com/). The free tier is sufficient for personal use.
2. Save your database password securely for future access.

## 2. Execute the Schema

The `mcp-memory-supabase` package requires a specific schema to enable `pgvector`, create tables, and define search functions.

1. Navigate to the **SQL Editor** in the Supabase dashboard.
2. Open a **New Query**.
3. Copy the raw SQL from [this schema file](https://raw.githubusercontent.com/kaspnilsson/mcp-memory-supabase/main/supabase/schema.sql).
4. Paste it into the editor and click **Run**.

### Security Note
The schema includes Row Level Security (RLS) policies that permit all access. This is intentional. The MCP server uses the Service Role key to bypass RLS. As a single-tenant backend service, the system requires no per-user access control.

## 3. Retrieve Credentials

Collect two values to configure the MCP server:

1. **Project URL:** Found in **Project Settings -> API**.
2. **Service Role Key:** Found in **Project API keys**. Copy the `service_role` key, not the `anon` key. The Service Role key allows the server to perform administrative writes.

Store these credentials securely for use in the VPS setup.
