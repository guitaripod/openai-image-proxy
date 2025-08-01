# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

```bash
# Development
npx wrangler dev            # Run locally with hot reload
npx wrangler deploy         # Deploy to production
npx wrangler tail           # Watch logs

# Database
npx wrangler d1 migrations apply DB --local                         # Apply migrations locally
npx wrangler d1 execute openai-image-proxy --file=migrations/001_schema.sql --remote  # Apply to production

# Secrets (MUST use wrangler secret, not config files)
npx wrangler secret put OPENAI_API_KEY

# CLI development
cd cli && cargo run -- [args]
The CLI app is a standalone product, but we also use it to validate the backend functions correctly.
```

## Important Notes

- **No automated tests** - Test with `npx wrangler dev` and the CLI tool.
- **Testing**: Use the CLI (`cd cli && cargo run -- [args]`) to test both CLI and backend functionality.
- **Cost optimization**: When testing image generation, always use `--quality low` (4-5 credits) instead of high (50-80 credits).
- **Rate limiting**: One concurrent request per user via `user_locks` table. Locks can get stuck.
- **API compatibility**: `/v1/images/generations` must match OpenAI's format exactly.
- **Build failures**: Run `cargo install worker-build` first.
- **Database migrations**: Tables have foreign keys - order matters.
- **No warnings**: There should be no compiler warnings.
- **No code comments**: Don't add code comments until specifically asked for, such as interface documentation.
- **CLI is the source of truth**: All client code must consider the CLI as the source of truth.
