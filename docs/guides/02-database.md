# 02. Database Provisioning

The memory system relies on a graph structure mapped to relational tables. 
- **Entities:** Nodes in your knowledge graph (e.g., "React", "Kasper's Desktop", "My Dog").
- **Observations:** Discrete facts attached to those entities (e.g., "Is a framework", "Uses a 4k monitor", "Eats kibble").
- **Relations:** Directed edges connecting entities.

To support semantic search across this graph, we need a Postgres database with the `pgvector` extension. This guide uses Supabase.

## 1. Create the Project

1. Log into [Supabase](https://supabase.com/) and create a new project.
2. The free tier is perfectly adequate for a personal knowledge graph. You will likely never hit the limits.
3. Save your database password securely. You will not need it for the MCP server, but you may need it for direct database access later.

## 2. Execute the Schema

The `mcp-memory-supabase` package requires a specific schema to function. It creates the tables, enables `pgvector`, sets up the HNSW index for fast similarity search, and defines the `match_entities` RPC function that the MCP server calls.

1. In the Supabase dashboard, navigate to the **SQL Editor**.
2. Click **New Query**.
3. Copy the raw SQL from [this schema file](https://raw.githubusercontent.com/kaspnilsson/mcp-memory-supabase/main/supabase/schema.sql).
4. Paste it into the editor and click **Run**.

You should see a success message indicating the tables and functions were created.

*Note on Security:* The schema includes RLS (Row Level Security) policies that permit all access. This is intentional. The MCP server connects to Supabase using the Service Role key, which bypasses RLS entirely. Because this is a single-tenant backend service, we do not need per-user access control. See the [Why Supabase?](../deep-dives/why-supabase.md) deep dive for more context.

## 3. Retrieve Your Credentials

You need two pieces of information from Supabase to configure the MCP server in the next step.

1. Navigate to **Project Settings -> API**.
2. Copy the **Project URL** (e.g., `https://xxxxxx.supabase.co`).
3. Scroll down to the **Project API keys** section.
4. Copy the **`service_role`** key. *Do not use the `anon` public key.* The Service Role key allows the server to bypass RLS and perform administrative writes.

Keep these secure. You will add them to your VPS environment in Step 3.
