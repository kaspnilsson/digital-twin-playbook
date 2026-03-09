# Economics

## The SaaS Stack

A typical AI power user pays (check current pricing -- these tiers shift):

- ChatGPT Plus: $20/month
- Claude Pro: $20/month
- Cursor Pro: $20/month

**Total: $60/month** for three siloed ecosystems.

## The Digital Twin Stack

- **VPS:** $6/month (DigitalOcean or similar).
- **Supabase:** $0 (Free tier).
- **TypingMind:** ~$3/month amortized (one-time ~$40 license).
- **API Compute:** Variable. Last month I spent $36 on OpenRouter for ~50M tokens. Claude Opus is *very* expensive; cheaper models bring this down.

**Monthly cost: ~$45.**

## The Tradeoff

Yes, $36/month in API costs can exceed a $20 subscription for heavy users. That is the point.

The $20 price is subsidized. OpenAI and Anthropic burn billions to acquire users. They can raise prices, degrade the product, or monetize your data in ways you have not consented to. You have no leverage because your context lives inside their walls.

When you pay for metered compute, you see the real cost. You can optimize it -- use cheaper models for simple tasks, cache aggressively, batch requests. You can switch providers overnight without losing your history.

## The Sovereignty Premium

You pay a premium to own the database. Whether that premium is worth it depends on how much you value:

- Portability between AI providers.
- Control over your personal context.
- The ability to leave a vendor without starting over.

For me, that premium is small compared to the cost of vendor lock-in.
