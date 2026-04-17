# dokploy-ghcr-deploy

A Claude Code skill that sets up automatic build + deploy of a Dockerized repo to a self-hosted [Dokploy](https://dokploy.com) instance using GitHub Actions, GitHub Container Registry, and the [Dokploy MCP](https://github.com/dokploy/dokploy-mcp).

## Install

```sh
npx skills add Jeffreyyvdb/dokploy-ghcr-deploy
```

Works with any skills-compatible agent harness (Claude Code, etc.) — see [skills.sh](https://skills.sh/docs).

## What it does

When you tell your agent something like *"set up auto-deploy for this repo to my Dokploy"*, this skill walks it through the full pipeline:

1. Verifies prereqs (Dockerfile with `EXPOSE`, `gh` CLI authed, Dokploy MCP connected, GitHub remote).
2. Writes `.github/workflows/docker-publish.yml` that builds on push to `main`, pushes `ghcr.io/<owner>/<repo>:{latest,sha-<short>}` using the built-in `GITHUB_TOKEN`, and triggers a Dokploy redeploy via `/api/application.deploy`.
3. Uses the Dokploy MCP to create a project + app, configure the Docker provider, set env vars, attach a domain with Let's Encrypt, and mint a scoped API key.
4. Sets the three required GitHub Actions secrets via `gh secret set`.
5. Opens a PR. First merge triggers the first real deploy.

## Why this pipeline

- **GHCR over Docker Hub** — Docker Hub's free tier caps at 1 private repo; GHCR gives unlimited private packages and authenticates with the built-in `GITHUB_TOKEN`.
- **API-trigger over registry webhook** — GHCR doesn't fire push webhooks, so the workflow POSTs to Dokploy directly from its final step.

## Requires

- GitHub repo with a working `Dockerfile` that `EXPOSE`s a port
- [Dokploy MCP](https://github.com/dokploy/dokploy-mcp) connected to your agent
- `gh` CLI authenticated
- One manual step the agent cannot automate: a GitHub classic PAT with `read:packages` scope for Dokploy to pull the private image (GitHub has no PAT-creation API)

## Layout

```
.
├── SKILL.md                              # frontmatter + 6-phase recipe + gotchas
└── assets/
    └── docker-publish.yml.template       # the GitHub Actions workflow the skill drops in
```

## License

MIT
