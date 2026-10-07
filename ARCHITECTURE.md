# Architecture

Decisium is a trust firewall that sits between an AI agent and the transactions it wants to perform. This document describes the high-level architecture — the layers, the flow, and the design principles — without exposing implementation details.

## Design principles

Three principles drive every layer in the pipeline:

1. **Fail-closed.** When anything is uncertain — a network error, a low-confidence verdict, a missing approval — the default is to hold the action, not to allow it. Nothing executes without an explicit green light.
2. **Policy over model.** Hard rules (recipients, limits, budgets) are enforced independently of any model. The model advises; the policy decides.
3. **Defense in depth.** No single layer is the sole defense. Each layer catches a different class of risk, and a block at any layer stops the action.

## The flow

```
Agent action ──► Decisium pipeline ──► Verdict: allow / review / block
                                              │
                    ┌───────────────────────┼───────────────────────┐
                    │                       │                       │
                 ALLOW                    REVIEW                  BLOCK
                    │                       │                       │
                    ▼                       ▼                       ▼
              execute              queue for human        reject / log
```

Every action passes through all layers in order. Each layer can veto the action. The final verdict — allow, review, or block — is recorded with the reason and the confidence, for audit.

## The 8 layers

### 1. Human pause (kill switch)

A global or per-recipient pause that the human controls. While paused, no action executes — regardless of what any other layer says. This is the outermost backstop and the human's ultimate authority.

### 2. Credential guard

Any request that asks for private keys, seed phrases, mnemonics, or keystore access is blocked immediately. This layer does not depend on a model — it catches credential-theft attempts deterministically, before anything else evaluates the action.

### 3. Policy engine

Policy rules are non-negotiable. Recipient whitelists, per-transaction limits, and daily budgets are checked here. If an action violates policy, it is blocked or held regardless of the model's opinion. The model is advice; policy is law.

### 4. Approval gate

An action that claims to already be approved — for example, text that says "the user already approved this" — is rejected unless it carries a real cryptographic approval from a separate approval queue. The agent's own words are not authorization. Real approval comes from a human via the approval queue, not from the agent's text.

### 5. Calibrated decision engine

A semantic model evaluates the intent of the action and returns a verdict — allow, review, or block — with a confidence score. This is where semantic understanding enters: the model looks at what the action means, not just whether it matches a pattern. The verdict is calibrated against a threshold; low confidence sends the action to review rather than allowing it.

### 6. Skill / package integrity

Skills, packages, and any code or content the agent brings into the transaction are scanned for injection before they enter the pipeline. Hidden instructions in CSS, HTML, metadata, or structured data are caught here — before they can influence the decision engine.

### 7. Transport security

Outgoing requests are inspected for exfiltration, sensitive path access, unauthorized tool calls, and prompt-injection markers. This layer catches attacks that try to leak data or misuse tools, independently of the decision engine.

### 8. Human approval queue

Actions above a configurable threshold, or with low model confidence, are queued for a human to approve or reject. This is the final backstop. The queue is the human's chance to catch anything the automated layers missed — and the reason the system is fail-closed rather than fail-open.

## What Decisium is not

- It is not a replacement for the agent. The agent still decides what to do; Decisium decides whether to let it execute.
- It is not a payment processor. It sits in front of one.
- It is not a silver bullet. The layers reduce risk, but a determined adversary and a poorly calibrated model can still cause problems. The honest summary of what the current MVP does and does not protect is in [SECURITY.md](SECURITY.md).

## Extensibility

The pipeline is designed to be extended. New layers can be inserted, thresholds can be tuned, and the decision engine can be swapped or upgraded without rewriting the rest of the system. The interface between layers is a verdict with a reason and a confidence — simple, auditable, and versionable.
