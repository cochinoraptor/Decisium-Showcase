# Metrics

Canonical numbers as measured against a calibrated INT4 decision engine on CPU. Updated after the final red-team run.

## Tests

- **Firewall tests:** 124/124 passing
- **WebMCPcss tests:** 652/652 passing
- **Move contract tests:** 17/17 passing
- **Red-team suite:** 30 adversarial attacks executed

## Latency (real Laya, INT4 CPU)

| Metric | Value |
|--------|-------|
| Pure inference (p50) | 143 ms |
| Full pipeline (p50) | 315 ms |
| Pipeline p95 | 434 ms |
| Pipeline max | 1094 ms (long context) |
| Pipeline average | 247 ms |

## Resources

| Metric | Value |
|--------|-------|
| Model size on disk | 262 MB (INT4 ONNX) |
| Process RAM, idle | 510 MB |
| Process RAM, peak | 548 MB |
| Target RAM limit | 4 GB |

## Red team (30 attacks)

| Outcome | Count | Meaning |
|---------|-------|---------|
| Hard block | 9 | The pipeline blocked the action outright |
| Escalated to review | 11 | Held for human review (not auto-approved) |
| Structural block | 10 | Blocked before the decision engine |
| Passed all layers | 0 | No attack succeeded |

**Block rate: 100% (30/30).** No attack passed all layers.

### Confidence distribution (20 evaluated attacks)

| Band | Count |
|------|-------|
| < 0.40 | 2 |
| 0.40 – 0.50 | 13 |
| 0.50 – 0.60 | 3 |
| 0.60 – 0.70 | 2 |
| > 0.70 | 0 |

Confidence values are low (p50 0.44) and uncalibrated for this domain. Only 2 of 20 evaluated attacks cleared the 0.60 threshold.

## Honest notes

- The INT4 checkpoint was fine-tuned primarily on English-language decision flows. On Spanish payloads it discriminates less well.
- Confidence values are uncalibrated. With a 0.6 threshold, many legitimate actions escalate to review.
- The current queue is in-memory and lost on restart.
- These numbers are from a pre-seed MVP on Sui Testnet with test assets. They are not production benchmarks.
