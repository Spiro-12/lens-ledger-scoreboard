# Lens & Ledger — public proof log

Lens & Ledger is building a human + AI trading firm for prediction markets. Right now everything is **simulated with fake money**. Real results would likely be worse.

This repo is our public record. It holds fingerprints (SHA-256 hashes) of our pre-registered test rules and, soon, of our daily logs. Each fingerprint is stamped with [OpenTimestamps](https://opentimestamps.org), which anchors it in Bitcoin, so anyone can check that nothing was changed after the fact.

- `proofs/prereg-v1/`: the test rules, approved and frozen on 17 Nov 2026, before any simulated trading. The full rules text will be published here with the scoreboard; its hash must match.
- `proofs/YYYY-MM-DD/` (coming): daily log fingerprints.
- A plain-English scoreboard (coming): fake profit and loss after fees, every loss included.

See `VERIFY.md` to check a proof yourself.
