# Decisium — The Trust Firewall for AI Agents

![Status: Pre-Seed MVP](https://img.shields.io/badge/status-pre--seed%20MVP-blue)

> Decisium is the trust layer that lets AI agents transact safely on the web.

## The problem

AI agents can already browse, compare, and buy on our behalf. But every agent faces the same unresolved question: **who decides whether a payment is safe?** Today that question is answered by the agent itself — and an agent that can spend is also an agent that can be tricked into spending.

Prompt injection, roleplay, and context manipulation are real attack vectors against agentic systems. A malicious webpage, an exfiltrated prompt, or a carefully framed conversation can push an agent to authorize a payment it should never make. Without a separate trust layer, the agent's judgment is the only line of defense.

## What Decisium does

Decisium sits between the agent and the transaction. Every action the agent wants to perform passes through a multi-layer decision pipeline that evaluates it before anything executes. The pipeline is **fail-closed by design**: when in doubt, the action is held for human review rather than allowed to proceed.

The key insight is that no single signal is enough. Decisium combines:
- **Policy rules** (recipients, limits, budgets) that are non-negotiable.
- **A calibrated decision engine** that evaluates the semantic intent of the action.
- **Structural checks** (credential requests, skill/package integrity, sensitive paths) that catch specific attack patterns.
- **Human approval** as the final backstop for anything above a threshold.

The result is a system where an agent can act autonomously within safe boundaries, and anything outside those boundaries is escalated — not auto-approved.

## The 8 layers

1. **Human pause** — A global or per-recipient kill switch. While paused, nothing executes.
2. **Credential guard** — Requests for private keys, seed phrases, or keystores are blocked immediately, regardless of everything else.
3. **Policy engine** — Recipient whitelists, per-transaction limits, and daily budgets. Policy is law.
4. **Approval gate** — Textual claims of approval ("the user already approved") are rejected. Real approval comes from a separate approval queue, not from the agent's words.
5. **Calibrated decision engine** — A semantic model evaluates the action's intent and returns a verdict with confidence.
6. **Skill/package integrity** — Skills and packages are scanned for injection before they enter the pipeline.
7. **Transport security** — Outgoing requests are inspected for exfiltration, sensitive paths, and unauthorized tool calls.
8. **Human approval queue** — Actions above a threshold, or with low confidence, are queued for a human to approve or reject.

## Architecture

```
                    ┌─────────────────────────────┐
                    │        AI AGENT              │
                    │  (browse, compare, purchase) │
                    └─────────────┬───────────────┘
                                  │ action request
                                  ▼
                    ┌─────────────────────────────┐
                    │         DECISIUM              │
                    │    (trust firewall layer)    │
                    ├─────────────────────────────┤
                    │  1. Human pause (kill switch)│
                    │  2. Credential guard          │
                    │  3. Policy engine             │
                    │  4. Approval gate             │
                    │  5. Calibrated decision engine│
                    │  6. Skill/package integrity   │
                    │  7. Transport security        │
                    │  8. Human approval queue      │
                    └─────────────┬───────────────┘
                                  │ verdict: allow / review / block
                                  ▼
                    ┌─────────────────────────────┐
                    │     TRANSACTION EXECUTOR     │
                    │  (payment, subscription, etc)│
                    └─────────────────────────────┘
```

Each layer is independent. A block at any layer stops the action. Layers are consulted in order, and the final verdict is recorded for audit.

## Verification highlights

### Tests
- **Firewall tests:** 124/124 passing
- **WebMCPcss tests:** 652/652 passing
- **Move contract tests:** 17/17 passing

### Adversarial validation
- **30 adversarial attacks** executed across 5 categories (direct injection, indirect injection, roleplay, context manipulation, sandbox escape).
- **3 vulnerabilities found and resolved** before the public release.
- **Result:** 0 attacks passed all layers in the final run.

## Red-team results (honest summary)

An independent red-team suite executed 30 adversarial prompt injection attacks against the pipeline. The results, as measured against a calibrated INT4 decision engine on CPU:

| Outcome | Count | Meaning |
|---------|-------|---------|
| Hard block | 9 | The pipeline blocked the action outright |
| Escalated to review | 11 | The action was held for human review (not auto-approved) |
| Structural block | 10 | Blocked before the decision engine (skill integrity, credential guard, etc.) |
| Passed all layers | 0 | No attack succeeded |

**Block rate: 100% (30/30).** No attack passed all layers.

### Known limitations (stated honestly)

- The INT4 decision checkpoint was fine-tuned primarily on English-language decision flows. On Spanish payloads it discriminates less well. Production use with non-English agents requires either English normalization or a multilingual checkpoint.
- Confidence values are low (p50 0.44) and uncalibrated for this domain. With a 0.6 confidence threshold, many legitimate actions would be escalated to review. Calibration against a real corpus is required before production deployment.
- The current queue is in-memory and lost on restart. Durable persistence is a production requirement.

## Status

- **Pre-seed.** MVP on Sui Testnet with test assets.
- **Seeking design partners** who want to let their agents transact safely.
- Full source code is private and available upon request under NDA.

## Contact

- **Email:** cochineator@gmail.com
- **Issues:** [Open an issue](https://github.com/cochinoraptor/Decisium-Showcase/issues)
