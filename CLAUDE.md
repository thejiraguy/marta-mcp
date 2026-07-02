# CLAUDE.md

Guidance for AI coding assistants (and humans) working in this repo.

## What this is

An MCP server exposing Atlanta MARTA real-time transit data (rail arrivals,
bus positions/trip updates). Python 3.10+, built with FastMCP, packaged with
hatchling, managed with [uv](https://docs.astral.sh/uv/).

## Layout

- `src/marta_mcp/server.py` — MCP tool definitions and entry point (`marta-mcp`)
- `src/marta_mcp/config.py` — API key resolution (`MARTA_API_KEY` env var, then
  `~/.marta/config.json`) and the `marta-mcp-config` helper CLI
- `src/marta_mcp/stations.py` — canonical rail station names and fuzzy matching
- `manifest.json` — Claude Desktop extension (MCPB/DXT) manifest; the extension
  is a thin bundle that launches the server via `uvx --from git+<this repo>`
- `scripts/build_dxt.sh` / `build_dxt.ps1` — pack the extension into
  `dist/marta-mcp.mcpb` and `.dxt` (requires Node.js for `npx`)

## Commands

```bash
uv sync            # install deps (including dev group)
uv run pytest      # LIVE smoke tests against MARTA's real APIs — needs network;
                   # rail tests auto-skip without an API key, bus tests need none
bash scripts/build_dxt.sh   # build the extension bundles locally
```

There is no linter or formatter configured; match the existing code style.

## Versioning and releases

The version lives in **two places** and must stay in sync:

- `pyproject.toml` → `[project] version`
- `manifest.json` → `"version"`

To cut a release: bump both, merge to `main`, then tag and push
(`git tag vX.Y.Z && git push origin vX.Y.Z`). The `Release` GitHub Actions
workflow (`.github/workflows/release.yml`) verifies the tag matches both files,
builds the `.mcpb`/`.dxt` bundles, and publishes a GitHub release with the
bundles attached and auto-generated notes. Don't create releases by hand.

Note: installed servers track `main` (the extension and README configs use
`uvx --from git+<repo>`), so users get code changes on merge to `main` —
releases exist to version the extension bundle and mark milestones.

## Constraints

- Public repo: never commit API keys, tokens, or personal config. The MARTA
  key is read from the environment or `~/.marta/config.json` at runtime and
  must never appear in code, tests, or fixtures.
- Rail data needs a free MARTA API key; GTFS-realtime bus feeds do not.
- Bus `route_id` values are MARTA's internal GTFS IDs, not public route
  numbers — don't "fix" them to match.
- This project is not affiliated with MARTA; keep wording in docs consistent
  with that (see README notes).
