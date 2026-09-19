# Blindproof

**Private & proven AI inference at scale — a Bittensor subnet.**

*Larvatus prodeo — "I advance masked." — Descartes*

Blindproof is a Bittensor subnet where miners run LLM inference on data they mathematically cannot read: they are paid only when a public proof confirms the work ran exactly right. The client masks a question, walks away, and returns to a verified answer from machines that never saw the question — or the answer they just computed.

## The problem

Trustless private inference does not exist. Every AI service today can read your prompts. The state of the art — TEEs — relocates trust to the chip vendor, and 2026 research (TDXRay, IEEE S&P) recovers user prompts from confidential VMs with encryption and attestation fully intact. For finance, healthcare, and law, "trust the chip" is exactly the trust that cannot be given.

## What the subnet sells

Three properties that stack:

- **Deterministic** — byte-identical execution on any GPU, making cross-miner verification free.
- **Proven** — a ZK proof per piece of computation, chain verification in ~20 milliseconds.
- **Private** — masked execution: every value a miner holds is noise.

Together: private, proven, settled — a cell no deployed system occupies.

## How it works (one job)

1. **Client masks & submits.** The input commitment is anchored on-chain; the miner receives only noise.
2. **Miner executes blind & proves.** Every piece of the computation carries a ZK proof; the bundle is posted to a public store and its hash committed on-chain.
3. **Validator verifies in ~20 ms — and money moves.** Four public checks pass: the miner's weight rises (emissions) and the client's escrowed fee releases (revenue). Any watcher can re-run the same checks; a false credit is slashable.

## Settlement is the new primitive

Verification is cheap enough (~20 ms) to run *before* payment on every job — not sampled, not probabilistic. One verified proof moves both money streams at once: the miner's validator-set weight rises (emissions), and the client's escrowed fee releases (revenue). Every crediting decision is publicly re-checkable, so a false credit is a slashable offense any watcher can prove for a bounty. And because every paid job leaves a proof bundle hashed on-chain, the subnet's revenue ledger is auditable by anyone.

Payment follows the proof, not the promise.

## Read the proposal

**[PROPOSAL.md](PROPOSAL.md)** — the full Checkpoint 1 subnet proposal: problem and use case, architecture, miner and validator design, incentive and reward mechanism, scoring methodology, users and ecosystem value, and the roadmap to testnet.

## Status

- **Checkpoint 1 (Sep 20):** proposal complete — this repository.
- **Week of Sep 21:** testnet netuid registration; mechanism code, contract specs, and reference verifier land here.
- **By Oct 2:** miner/validator loop running on Bittensor testnet.
- **Oct 9–16:** incentive experiments (lazy miner, splicing miner, honest miner, watcher catch), fee escrow released on verification, demo video.
- **Oct 19:** final submission — testnet live, repository complete, demo filmed.

## Team

We come from the regulated finance this subnet serves — we are building the product we were never allowed to buy.
