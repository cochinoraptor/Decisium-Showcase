# Security

Decisium is designed to let AI agents transact safely. This document describes the threat model, the design principles, the known limitations, and the results of adversarial testing — honestly, including what the current MVP does not protect.

## Threat model

The following actors and threats inform the design:

| Actor | Threat | MVP mitigation | Production pending |
|-------|--------|---------------|-------------------|
| Compromised agent / prompt injection | Agent is tricked into paying an attacker | Whitelist + on-chain limits + calibrated model + human approval | On-chain receipt, agent session telemetry |
| Remote attacker | Abuses the firewall API | Localhost-only binding, bearer-token auth on state-changing endpoints, body size limit, strict input validation | Per-origin CORS + token rotation |
| Malicious insider (admin) | Abuse admin privileges | By design the human is the ultimate authority | Multi-sig for shared governance |
| Degraded or down decision engine | Garbage or missing verdict | Fail-closed: error leads to review, never allow | Health check with circuit breaker and alerts |
| Manipulated model confidence | "Safe" with artificially high confidence | Calibrated confidence + conservative thresholds | Fine-tuning with own dataset + continuous eval |
| Sponsor front-running | Firewall pays gas for unwanted transaction | Sponsor only signs transactions built by the firewall itself | Per-session gas budget |

## Design principles

### Fail-closed

When anything is uncertain — a network error, a timeout, a low-confidence verdict, a missing approval — the default is to hold the action, not to allow it. There is no permissive default anywhere in the pipeline. A failure in any layer results in review, not execution.

### Policy over model

Hard rules (recipients, limits, budgets) are enforced independently of any model. The model evaluates intent and returns a verdict with confidence; the policy enforces non-negotiable constraints. The model is advice; the policy is law.

### Double validation

Limits are checked both off-chain (before the transaction) and on-chain (as part of the same transaction). If the on-chain check fails, the transaction does not happen. This protects against a mismatch between what the off-chain system thought and what the chain will actually do.

### On-chain kill switch

The human can revoke spending authority on-chain. Revocation leaves limits at zero: even if the entire off-chain infrastructure is compromised, the agent cannot spend. A reversible complement is the pause mechanism, which stops all action while active.

### Constant-time comparison

Approval tokens are compared in constant time to avoid timing side-channels.

### Hardened local API

The local API binds to localhost, authenticates state-changing requests with a bearer token, enforces a body size limit, and validates input strictly — addresses, amounts, and strings are all checked before use.

### Keys

The MVP reads the local wallet only for deployment scripts. No service listens with keys loaded by default in simulation mode. Production: keys in a KMS/HSM with rotation.

## What the MVP validated

- **Fail-closed behavior** with a degraded or unavailable decision engine: errors lead to review, never to allow.
- **Policy enforcement** independent of the model: limits and whitelists block regardless of the model's verdict.
- **Credential guard**: requests for private keys, seed phrases, or keystore access are blocked before the decision engine sees them.
- **Approval gate**: textual claims of approval are rejected; real approval comes from a separate queue.
- **Skill/package integrity**: skills and packages are scanned for injection before entering the pipeline.
- **Transport security**: outgoing requests are inspected for exfiltration, sensitive paths, and unauthorized tool calls.

## Known limitations (MVP)

- **Local API authentication (DEC-2610-001, remediated).** State-changing endpoints now require a bearer token issued on first startup; the API still binds to localhost (Testnet) and the human remains the last line via the approval queue.
- **Whitelist comparison hardened (DEC-2610-002, remediated).** Recipients are matched by exact full address and malformed addresses are rejected; suffix comparison is gone.
- **Queue and budget in memory**, lost on restart. A restart resets the daily budget; the on-chain policy is not reset.
- **Decision thresholds via environment variable**. A careless operator could relax them. Production: signed and versioned configuration.
- **No multi-tenancy.** The current design is single-user.
- **No durable queue.** The approval queue does not survive a restart.
- **Decision engine not formally verified.** DEC-2610 covered the firewall's deterministic controls; the model itself is exercised only by the red-team suite.
- **Mainnet not touched.** Verified: testnet RPC, testnet faucet, testnet explorer only.

## External Security Audit — DEC-2610

Decisium was audited externally by **Pentia** (report DEC-2610). The audit found 8 issues: 7 were in scope for the MVP and **all 7 are remediated**, each covered by regression tests (the firewall suite grew from 127 to 192 tests). One finding — on-chain custody — is out of scope for the MVP and is deferred to the next audit cycle alongside the contract work.

| Finding | Status |
|---------|--------|
| DEC-2610-001 — API authentication | FIXED |
| DEC-2610-002 — Recipient whitelist comparison | FIXED |
| DEC-2610-003 — On-chain custody | OUT OF SCOPE (MVP) |
| DEC-2610-004 — Input normalization before security checks | FIXED |
| DEC-2610-005 — Approval queue rate limiting and capacity | FIXED |
| DEC-2610-006 — Outbound request (SSRF) protections | FIXED |
| DEC-2610-007 — Concurrent budget enforcement | FIXED |
| DEC-2610-008 — Payment authorization replay protection | FIXED |

Severities and technical detail are part of the full report, available under NDA.

## Adversarial testing

A red-team suite of 34 adversarial prompt injection attacks targets the pipeline. The published run below executed 30 of them; the 4 payloads added during the DEC-2610 remediation are blocked by deterministic layers (regression-tested). The attacks covered 5 categories:

- Direct injection
- Indirect injection (in content, metadata, structured data)
- Roleplay
- Context manipulation
- Sandbox escape

**Results (against a calibrated INT4 decision engine on CPU):**

| Outcome | Count |
|---------|-------|
| Hard block | 9 |
| Escalated to review | 11 |
| Structural block (before decision engine) | 10 |
| Passed all layers | 0 |

**Block rate: 100% (30/30) in the published run.** No attack passed all layers.

### Vulnerabilities found and resolved

The suite found 3 structural vulnerabilities before the public release. All three were resolved and are covered by tests:

1. **Credential guard gap** — Reading the local wallet keystore was not blocked. Resolved by expanding the sensitive-path list and adding a credential request detector that blocks deterministically, before the decision engine sees the action.
2. **Fake approval via text** — The agent's text could claim the user had approved a payment. Resolved by rejecting textual approval claims unless they carry a real cryptographic approval from the approval queue. The claim is always blocked, at any amount.
3. **Skill/package injection not scanned** — Skills and packages were validated for structure and checksum only; their content was not scanned for injection. Resolved by adding a prompt-injection scanner that blocks skills containing bypass, transfer, or fake-approval instructions before they enter the pipeline.

### Honest limitations of the red-team results

- The decision engine checkpoint was fine-tuned primarily on English-language flows. The suite was run in Spanish. A Spanish result is approximate by model design.
- Confidence values are low (p50 0.44) and uncalibrated for this domain. With a 0.6 threshold, most actions escalate to review, including the benign control. This is fail-closed behavior, not evidence that the system discriminates well.
- The 10 structural blocks (skill integrity, credential guard) do not depend on the model at all. Their success is the pipeline's, not the model's.
- These numbers are from a pre-seed MVP on a testnet with test assets. They are not production benchmarks, and they should not be presented as evidence that the system is production-ready.

## Production recommendations

1. **Normalize the state to English** before the decision engine evaluates it (or use a multilingual checkpoint). This is a requirement, not an option, for the model to discriminate well.
2. **Calibrate confidence thresholds** against a real corpus until the benign control passes to allow without opening the attacks.
3. **Apply confidence gating to blocks as well as allows.** Do not trust a block with low confidence blindly.
4. **Repeat the adversarial run multiple times** and average before publishing any numbers. Add the payloads the model did not catch to the fine-tuning corpus.
5. **Add durable persistence** for the queue and budget before any production deployment.
6. **Add authentication** to the local API before any multi-process or multi-user deployment.
7. **Re-audit before mainnet.** The MVP was externally audited (DEC-2610); re-audit after contract changes and before any mainnet deployment.

## Disclaimer

This document is an honest accounting of a pre-seed MVP. It is not a security audit, not a guarantee, and not a recommendation to deploy as-is. Production security requires a dedicated senior engineer and continuous auditing; the MVP has been externally audited (DEC-2610) with all in-scope findings remediated.
