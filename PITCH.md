# PITCH — Decisium

## The problem

AI agents already browse and buy on our behalf, but **nobody gives them a card**:
agentic payment demands either blind trust (unacceptable) or human approval of every
click (unusable). The result: pilots blocked by risk and compliance at the companies
that most need to automate payments and subscriptions.

## The solution

**Decisium** is a firewall for agents' *decisions*:

- **On-chain programmable limits** that the user controls with their wallet
  (daily budget, max per transaction, whitelist, kill switch).
- **Calibrated decision in p50 143 ms pure inference** (Laya, System-1; ~315 ms end-to-end on CPU): every action is evaluated in
  one forward pass with measurable confidence — no generative LLM hallucinating.
- **Human-in-the-loop by exception**: only the doubtful reaches the approval queue;
  the routine flows on its own within limits.
- **Verifiable audit**: every executed action stays in on-chain events linked to the
  agent's identity.
- **Gasless payments** for the user via Sui's native sponsorship.

## Why now

1. Agentic identity standards (ERC-8004) are being born: the trust layer is being
   defined NOW.
2. Sui allows spend policies as on-chain objects + native sponsorship: the agentic
   payment stack exists and nobody has productized it with a calibrated decision
   firewall in front.
3. The market wants control: every news story of an agent that spends badly is a
   customer.

## Initial market

- **Agencies and agent operators** (browser automation) that need to sign enterprise
  customers: the firewall is their compliance.
- **SaaS with support/ops agents** that pay subscriptions or invoices.
- Design partners in conversation: **Houston** (payments), **Callbook AI**
  (voice agents with expenses).

## Stack (no proprietary secrets)

| Layer | Technology |
|---|---|
| Web tools | WebMCPcss (declarative, self-repairing) |
| Decision | Laya 0.3.21 (typed-decisions checkpoint), served locally |
| Chain | Sui Testnet: Move (SpendingPolicy, AgentIdentity, MockUSDC) |
| Service | TypeScript + Express + @mysten/sui SDK |
| UI | React + Vite + @mysten/dapp-kit |

Defensible differentiators: the firewall's own dataset and thresholds (fine-tuning
Laya with real decisions), auditable policy contracts, and the WebMCPcss → action
catalog → decision → payment integration, end to end.

## Validated cases in the MVP

- ✅ Legitimate subscription within limits: `safe` decision with calibrated confidence;
  dropping below threshold goes to human review and is approved 1-click.
- ✅ Fraud attempt ("urgent transfer to new wallet, expires in 10 minutes"):
  **blocked** by Laya (`blocked`, risk 75/100).
- ✅ Limits: exceeding maxPerTx or the daily blocks; daily rollover works on-chain
  (15/15 Move tests).
- ✅ Fail-closed: with Laya down, nothing executes automatically.

## Roadmap to production

1. **Q1**: live deploy with real USDC on Testnet, hot-potato receipts in Move,
   persistent queue, session auth.
2. **Q2**: Laya fine-tuning with partner data (refund, chargeback, providers),
   multi-agent dashboard, external audit.
3. **Q3**: Mainnet with governance multi-sig, SDK for agent operators, wallet
   listing with sponsorship.
4. **Q4**: interoperable spend policy standard (proposal to ERC-8004 as a payment
   extension).

## What we ask for

Design partners with real decision volume to tune the thresholds (data flywheel),
and Sui ecosystem support (grant) to take the security layer to Mainnet with audit.
