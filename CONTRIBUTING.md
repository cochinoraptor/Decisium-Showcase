# Contributing to Decisium

Thanks for your interest in Decisium. This document covers how to report issues, submit changes, and run the project locally.

## Code of conduct

Be respectful and constructive. This project is a security-sensitive agentic payment firewall; keep discussions technical and evidence-based.

## Reporting issues

- Use the GitHub issue tracker.
- Include: what you expected, what happened, steps to reproduce, and environment (OS, Node version, Sui CLI version, laya version if relevant).
- For security issues, do **not** open a public issue. Follow the project's security disclosure process if one exists; otherwise contact the maintainers privately.

## Submitting changes

1. Fork the repo and create a feature branch.
2. Make your change. Keep it focused: one concern per PR.
3. Add or update tests if the change affects behavior.
4. Run the checks below and make sure they pass.
5. Open a pull request with a clear title and description.

## Development setup

See [SETUP.md](SETUP.md) for the full environment.

Quick start:

```bash
# WebMCPcss (third-party, vendor/)
cd vendor/WebMCPcss && npm install && npm run build && npm test

# Decision engine (Laya)
pip install --user "laya[serve]"
laya-serve --host 127.0.0.1 --port 8000 &

# Firewall
cd firewall && npm install && npm test && npm run build

# Trust Console
cd trust-console && npm install && npm run dev
```

## Checks

From `firewall/`:

- `npm test` — vitest suite (must stay green; current status is 192/192).
- `npm run build` — strict `tsc` compile (must exit 0).
- `npx tsc --noEmit` — typecheck without emitting.

## Code style

- TypeScript, strict mode.
- Keep logic and strings separate: user-facing messages and comments should be in English.
- Do not rename variables, functions, types, or config values to translate text.
- Do not change test expectations unless the underlying behavior changed.
- Prefer small, revertable commits. Squash if the project prefers it.

## Documentation

- Docs are Markdown under the repo root and `docs/`.
- Update docs when behavior changes.
- Keep README, SECURITY_NOTES, DECISIONS, and test reports consistent with the current state.

## Testing requirements

- New features should include tests.
- Security-relevant changes should include negative tests (fail-closed paths, malformed input, blocked attacks).
- Keep the red team suite reproducible: do not translate or modify attack payloads.

## License

By contributing, you agree that your contributions are licensed under the project's license (see [LICENSE](LICENSE)).
