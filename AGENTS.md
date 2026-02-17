# AGENTS.md

This file is a quick bootstrap guide for coding agents working in this repository.

## 1) What this repo is

- Repo: `mech-predict` (Autonolas Mech service for prediction-related tools).
- Primary logic lives under `packages/`:
  - `packages/*/customs/*`: custom tool implementations.
  - `packages/valory/agents/mech/aea-config.yaml`: agent package definition (customs list, dependencies).
  - `packages/valory/services/mech/service.yaml`: service definition and runtime env wiring.
  - `packages/packages.json`: local package hash registry.

## 2) Environment setup

- Python: `3.10` (recommended; CI and tox are built around 3.10).
- Install deps:
  - `poetry install && poetry shell`
- Sync autonomy packages:
  - `autonomy packages sync --update-packages`

## 3) Common commands

- Format:
  - `make format`
- Lint/check:
  - `make code-checks`
  - `make security`
- Package + metadata checks:
  - `make common-checks-1`
  - `make common-checks-2`
- Full local sweep:
  - `make all-checks`

Useful tox targets:
- `tox -e check-tools` (tool tests in `tests/`)
- `tox -e check-hash`
- `tox -e check-packages`
- `tox -e check-doc-hashes`

## 4) Running service/agent locally

Service mode:
- `cp .example.env .1env`
- fill values in `.1env`
- `source .1env`
- `autonomy generate-key ethereum -n 1` (creates `keys.json`)
- `bash run_service.sh`

Standalone agent mode:
- `cp .example_agent.env .agentenv`
- `autonomy generate-key ethereum` (creates `ethereum_private_key.txt`)
- `bash run_agent.sh`

Notes:
- `run_agent.sh` starts Tendermint internally.
- `run_service.sh` is destructive for existing `mech/` build dir (`rm -r mech`).

## 5) Where to change model/tool behavior

- Tool names and model aliases are usually in each custom's Python module (`ALLOWED_TOOLS`, `TOOL_TO_ENGINE`, `LLM_SETTINGS`).
- Component defaults are in each `component.yaml` (`params.default_model` where present).
- Service-level runtime maps are injected via env vars (`TOOLS_TO_PACKAGE_HASH`, `TOOLS_TO_PRICING`, `API_KEYS`) in `service.yaml`.
- If you add/rename tool aliases, also update any scripts/tests that hardcode names:
  - `scripts/test_tools.py`
  - `scripts/test_tool.py`
  - `README.md` tool docs

## 6) Consistency checks after edits

When editing tools/components, verify all of these:

1. Python module constants updated (`ALLOWED_TOOLS`, model map).
2. Corresponding `component.yaml` defaults updated (if applicable).
3. `packages/valory/agents/mech/aea-config.yaml` still references intended customs.
4. `packages/packages.json` remains in sync (run package lock/update flow as needed).
5. Docs/scripts that mention tool names/models are updated.

Quick syntax sanity:
- `python -m compileall packages scripts`

## 7) Repository-specific pitfalls

- Some tool modules are large (1k+ LOC). Prefer small, targeted edits.
- Error handling is often broad (`except Exception`) in customs; avoid introducing silent failures.
- There are multiple similarly named prediction tools; double-check you are editing the correct vendor path.
- Tests do not cover every custom end-to-end; rely on targeted checks and static validation in addition to `tox -e check-tools`.
- Do not assume README examples are authoritative for internals; verify against `packages/valory/agents/mech/aea-config.yaml` and `service.yaml`.

## 8) PR guidance

- Create focused branches with `codex/` prefix.
- Keep commits scoped (tool behavior vs docs vs packaging metadata).
- In PR description, explicitly list:
  - tool aliases changed,
  - model defaults changed,
  - env vars or config maps impacted,
  - validation commands run.
