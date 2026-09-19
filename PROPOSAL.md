# Blindproof
*Larvatus prodeo — "I advance masked." — Descartes*

## Private & proven AI inference at scale

**Bittensor Global Subnet Hackathon — Subnet Proposal (Checkpoint #1)**

Blindproof is a Bittensor subnet where miners run LLM inference on masked data they mathematically cannot read — and are paid only when a public proof confirms the work ran exactly right. 

* Every value a miner computes is masked noise. 
* Every piece of the computation carries a zero-knowledge (ZK) proof. 
* Every proof is checkable by anyone in milliseconds, and payment follows the proof, not the promise.
* Settlement rests on mathematical certainty that the computation ran correctly — to our knowledge, a first for private inference.

The client masks a question, walks away, and returns to a verified answer — from machines that never saw the plain-text question at all.

The name "Blindproof" is the design: the miners are blind, yet the work is proven. 

The contribution of this subnet is its economic mechanism — to our knowledge a first of its kind: incentives and settlement conditioned on cryptographic proof, with verification cheap enough to run before payment on every job, so that one verified proof moves both money streams at once — the miner's emissions and the client's escrowed fee. 

What makes the mechanism possible is where the latest cutting edge llm technology now stands: inference can run bit-exact across different GPUs, computation on masked data is practical piece by piece, and verifying a proof costs milliseconds while redoing the work costs seconds — a gap wide enough for a market to settle on. 

Our submission is the subnet proposal built on this substrate — specified below, running on testnet by final event close.


## Contents

1. [Problem and use case](#1-problem-and-use-case)
2. [Subnet architecture](#2-subnet-architecture)
3. [Miner responsibilities and tasks](#3-miner-responsibilities-and-tasks)
4. [Validator responsibilities and evaluation](#4-validator-responsibilities-and-evaluation)
5. [Incentive and reward mechanism](#5-incentive-and-reward-mechanism)
6. [Scoring methodology](#6-scoring-methodology)
7. [Expected users and ecosystem value](#7-expected-users-and-ecosystem-value)
8. [Roadmap toward testnet and future deployment](#8-roadmap-toward-testnet-and-future-deployment)

---

## 1. Problem and use case

### The problem

Trustless private inference does not exist. A prompt sent to any of today's AI services is readable by whoever runs the compute on thier machine.

The best current answer — hardware enclaves, formally Trusted Execution Environments (TEEs) — does not remove that exposure. It relocates the trust from the service operator to the chip maker and its supply chain — both with documented histories of compromise. For the workloads that need privacy most, "trust the chip" is exactly the trust that cannot be given.

The exposure carries two costs. For a single party, the cost is the questions that never get asked: an analyst cannot put a market-sensitive question through a shared AI platform, because the operator of that platform becomes an insider to material non-public information the moment the question is typed.

For two parties, it means joint computation on both sides' secrets is not merely risky but unavailable. Consider a company seeking credit against a lender's eligibility bar. The computation is trivial — does the applicant's position clear the lender's threshold — but no inference service can run it, because using one forces at least one party to expose its secret: the applicant its financials, the lender its true criteria, and both of them to whoever operates the service.

So these transactions clear the only way they can — through trusted intermediaries, brokers, bankers, and clean-room vendors, whose fee is compensation for being shown both sides' secrets — and where no trusted middleman exists, the deals die undiscovered, because merely approaching the counterparty leaks interest, price, and position.

### The existing answer, and why it is not enough

The state of the art is the TEE — the Trusted Execution Environment, a hardware "enclave": a sealed region of the chip that promises to run code so that not even the machine's operator can look inside. With one exception addressed below, every private-inference service in production today — on this network and off it — rests on TEEs. That includes the LLM products of the cryptography companies themselves: the leading MPC (secure multi-party computation) vendor runs its private-LLM service inside confidential-computing CPUs and GPUs, and the leading FHE (fully homomorphic encryption) vendor's LLM design runs most of the model unencrypted on the client, reserving encryption for a small fragment.

An enclave does not make inference trustless. It relocates the trust — from the service operator to the chip maker: silicon, firmware, microcode, and the attestation servers that vouch for all of it. The user stops trusting a company and starts trusting a hardware supply chain.

That supply chain keeps breaking. Foreshadow, Plundervolt, and SGAxe broke Intel's first-generation enclaves — SGAxe stole the attestation keys themselves. TDXDown and CounterSEVeillance hit the current Intel and AMD lines. TEE.Fail (2025) broke both current generations with under $1,000 of hardware and forged the attestation certificates — Intel's and NVIDIA's — so ordinary workloads can masquerade as enclave-protected.

The newest result is the most direct. TDXRay (IEEE S&P 2026) recovers users' LLM prompts from unmodified confidential VMs on current hardware: the host reads the prompt back from memory-access patterns during tokenization — reliably, from a single trace — while encryption and attestation stay fully intact.

The enclave can hold and the prompt can leak anyway.

Three non-TEE roads have now been tried. Each proves part of the direction and leaves the gap:

- **Commercial split inference** (an early-access system, CCS 2026 demo track) serves a 119-billion-parameter open model with no TEE by masking the heavy linear math it sends to its servers, while attention, every nonlinear step, and the memory cache stay on the client's own machine. No hardware vendor to trust — but the client stays online for every layer, carries the model's hardest compute itself, and receives no proof the delegated work was done correctly. Privacy, without verification or a market.

- **Client-verified split inference** (an academic prototype, September 2026) adds verification to the same split — but only the client who issued the challenge can check the result. There is no transferable proof a third party could inspect, and therefore nothing a marketplace could settle payment on.

- **Pure FHE** — running the entire model under encryption — posted its best open result in September 2026: a 128-token input through an 8-billion-parameter model in roughly six minutes on an H100. Orders of magnitude slower than plaintext, unverified, unsettled. That cost wall is exactly what our piece architecture is designed to route around.

**Trustless inference — private, proven, and settled — still does not exist.**

To be clear about what we are not claiming: TEE privacy is real security against many adversaries. But it is trust in a manufacturer, not a guarantee from mathematics — and it cannot serve the two-party case at all, because an enclave is by definition one box that sees everything inside it.

### What we build

Private inference as a market commodity: models executed on masked data by machines that see only noise, proven correct by anyone in milliseconds, and paid on proof.

The cryptography for computing on hidden data is forty years old — Yao's millionaires' problem and the field it founded.

What has never existed is the market: a way to price private inference, prove it ran, and settle it, so that anonymous machines are paid for inference they cannot read. This subnet is that market.

Privacy comes from mathematics, with no hardware vendor anywhere in the trust claim; enclaves are required nowhere and welcome everywhere, as an optional second wall on the cryptographic floor.

### Two abilities the world has not had.

First, anyone can use a frontier-scale AI model from a laptop or phone with mathematical certainty that no one — not the machines running it, not the network, not us — can read the question or the answer. The model runs distributed across miners who each see only noise, and the user's device can verify every step of the work itself, because checking proofs costs milliseconds regardless of model size. The frontier model becomes a verifiable public utility.

Second, two parties can transact on their secrets without a middleman. The applicant and the lender agree on a public eligibility model; each submits its inputs masked — the applicant's balance provably sourced from its own bank via a web proof — and both learn exactly one bit: deal or no deal. A "no" costs neither side any information; a "yes" starts a conversation both now know is worth having. The same shape prices reinsurance treaties, screens acquisitions, and matches buyers to sellers — the trusted middleman replaced by a protocol.

### The commodity, precisely

The commodity is compute: a slice of LLM execution. What is new is the guarantees attached to each slice — three of them, and they stack (priced as the SKU stack, section 3):

+ Deterministic — always on. Every slice has exactly one correct output, byte-identical on any GPU. Correctness becomes a fact anyone can check, not an opinion a validator holds.
+ Proven — the slice carries a zero-knowledge proof that the agreed model ran on exactly the committed inputs. Verifiable by anyone, in milliseconds.
+ Private — the slice runs on masked inputs. The miner computing it cannot read the data.

The fully stacked unit is the proven private piece. Pieces chain into complete inference jobs; the flagship is full inference on a pinned open-weight LLM — for example: does this company pass this acquisition screen? The inputs stay private, the answer returns masked to its owner, and every piece of the run carries its proof.

One rule makes this work between strangers: the model is always public and agreed in advance. The inputs are secret; the function never is. A public model is what makes verification meaningful — there is a definite right answer to prove — and it is what lets two adversarial counterparties trust the same computation without trusting each other.

The same rail also runs the smallest possible job: the blind threshold check — a single comparison between two parties' secrets (does A's number clear B's bar?), priced in pennies and settling in near-real time. Not a second product — the smallest job on the same rail, and the market's on-ramp.

### The market today

Hardware-trust private inference already earns eight-figure annual revenue on this network (Targon; SN28's confidential routing), proving the demand.

zkML subnets prove execution but do not hide inputs. The cell where privacy and verification are both cryptographic is, in every deployed offering publicly documented, empty — an earlier ideathon prototype sketched the combination at toy scale and was never deployed, evidence that the concept is not the hard part.

This subnet is built to fill that cell, serving the validated demand one trust level up and adding the buyers — regulated financials, cross-border data, material non-public information — for whom hardware trust is the dealbreaker.

Our proof rail is equally useful to the TEE tier, which we treat as complementary infrastructure, not competition.

### The market today

Hardware-trust private inference already earns eight-figure annual revenue on this network (Targon; SN28's confidential routing), proving the demand.

zkML subnets prove execution but do not hide inputs. The cell where privacy and verification are both cryptographic is, in every deployed offering publicly documented, empty — an earlier ideathon prototype sketched the combination at toy scale and was never deployed, evidence that the concept is not the hard part.

This subnet is built to fill that cell, serving the validated demand one trust level up and adding the buyers — regulated financials, cross-border data, material non-public information — for whom hardware trust is the dealbreaker.

Our proof rail is equally useful to the TEE tier, which we treat as complementary infrastructure, not competition.

### What exists, what the hackathon proves, what remains

- Deterministic inference: Measured (mid-size open-weight model); hackathon: Subnet integration.
- Proof generation: Measured (mid-size open-weight model); hackathon: Miner production path.
- Proof verification: Measured, tens of ms; hackathon: Validator + client integration.
- Masked computation: Piece-scale, demonstrated; hackathon: Integrated private pieces, grade-tagged.
- Two-input masking: Piece-scale, demonstrated; hackathon: Blind threshold check on testnet.
- Gateway mode (interactive): Architecture complete; hackathon: Proven end to end on testnet.
- Capsule mode (non-interactive): Piece-scale, demonstrated; hackathon: Elements demonstrated, grade-tagged.
- Full private LLM: Not claimed; hackathon: Phase-two convergence.
- zkTLS attestation (web proofs binding an input to its HTTPS source): Interface specified; hackathon: Future integration.
- TEE tier: Optional by design; hackathon: Future tier.
Trust removed: miner honesty; validator honesty for deterministic verification; the execution operator seeing plaintext; any hardware vendor.

Assumptions remaining: standard cryptographic hardness per each piece's grade tag; secure client-side masking; blinded material manufacture with manufacturer non-collusion; availability-quorum honesty; Bittensor's validator-stake consensus; implementation correctness. The system does not eliminate trust; it relocates residual trust into explicitly identified, cheaper assumptions.

The hackathon does not ask the judge to take full private frontier inference on faith. It demonstrates the proof substrate, the private pieces, and the economic loop that joins them.

---

## 2. Subnet architecture

Three properties in a dependency chain: **deterministic, therefore provable, therefore payable; masked, therefore private; together, a market.**

### Deterministic

Miners serve a pinned open-weight model (testnet: Qwen2.5-0.5B) through a bit-exact integer execution path producing byte-identical outputs across heterogeneous GPUs (demonstrated across four GPU generations in our existing proof system).

Determinism is load-bearing three times: proofs commit to one canonical answer reproducible anywhere; cross-miner agreement checks are free; and masking arithmetic is exact, so masks strip perfectly at every seam.

### Masked

Input owners mask locally before submission; every value the compute side ever holds, including attention scores and patterns, is masked — in the linear bulk of the model, masks are one-time pads (OTP) — the masked values are exactly uniform noise; inside the nonlinear "islands," masked values are computationally indistinguishable from noise; each piece's grade tag declares which.

Linear layers — the bulk of a transformer — run on additively masked values at near-plaintext cost; nonlinear operations run in confined masked islands.

Execution runs in one of two modes, declared per job. **Gateway mode (interactive):** the client's orchestrator stays in the loop, crossing each seam live — masks are generated on demand from its own pool, so almost no precomputed material is required. This is the simpler mode, available today wherever the client runs or elects to trust an orchestrator: an enterprise gateway on the client's own infrastructure, or a hosted orchestrator the client chooses.

**Capsule mode (non-interactive):** the client masks, submits, and may go offline; every seam crossing consumes precomputed, data-independent material (capsules, triples, delta shares) manufactured in advance and "hydrated" with the job's masks at dispatch. Capsule mode is the harder construction and the endgame: it is what removes the client from the loop entirely — fire-and-forget jobs, batch inference, and ultimately a fully untrusted network-side coordinator — and it is what makes the material market an economy.

In both modes the miner is equally blind, and in both the miner finishes holding the trace pieces that deferred, parallel proving consumes.

### Proven: contract, instance, witness

Each piece *type* (RMSNorm at a width, each linear shape, each island) is governed by one published **compute contract**: an immutable, versioned document containing the piece's circuit — the relation "the value committed in C_out equals the piece function applied to the value committed in C_in under the pinned weight commitment" — together with its interface, material schema, constraint-count weight, grade classes, and conformance vectors.

The contract book is the network's commercial law: what a miner must prove is what the contract says, and settlement follows mechanically, because performing the contract and proving performance are the same artifact — the proof either verifies against the published contract or the work was not delivered. Each job supplies **instances** — commitment pairs, layer index, contract version, manifest and material hashes — and the miner holds the **witness**, the masked trace. Proving is deferred, embarrassingly parallel, and streamable (early pieces proven while later layers execute).

Proofs are self-certifying: a valid proof exists only for the correct computation on the committed values, regardless of who produced it.

### Anchored

The miner never controls either end of the chain. The orchestrator posts the input commitment at dispatch — a valid chain must start there. Piece identities bind to the public manifest — a proof only verifies against the declared circuit and pinned weights.

Every seam requires this piece's input commitment to equal the last piece's output commitment. The chain must end at a commitment matching the bytes actually delivered.

Wrong input fails the start anchor; wrong output fails the delivery match; a tampered middle breaks a seam; a wrong function fails its circuit; an "honest proof of garbage" is impossible by soundness. The miner's proof is a bridge, and the orchestrator owns both banks.

### Parties

The **client** owns plaintext. Its **orchestrator** — a gateway inside the client's trust domain, whether self-hosted on the enterprise's own infrastructure or a hosted orchestrator the client elects to trust — holds masks and bookkeeping only: it slices, hydrates, dispatches, verifies, and unmasks, but never computes, proves, or validates; its work scales with activations, not parameters, which is why it can shrink to a client library, with its non-secret bookkeeping migrating network-side as an untrusted coordinator once full-depth non-interactivity is confirmed. The **miner** holds all compute and no keys. The **validator** reads only commitments and proofs.

In the two-party mode, each party masks its own input slots and contributes additive delta shares computed against the public weights; the output returns under a joint mask, so disclosure is an exchange of mask shares — escrowable by the validator for atomicity.

No plaintext exists anywhere outside its owner, and client privacy depends on the honesty of no one outside the client's own trust domain, including the referee.

### Delivered and stored

The miner posts the proof bundle and masked result to a content-addressed public store and commits the hash on-chain: "delivered" means retrievable at the committed hash and hashing to the chain's terminal commitment.

Publication is safe because nothing secret ever appears — bundles hold Pedersen commitments (perfectly hiding, information-theoretically) and proofs; results are masked noise.

Storage is two-tier by security lifetime: **bundles permanent** (kilobyte-scale, sub-cent, the eternal audit trail), **masked payloads windowed** — retained under bond through the challenge window, deleted by default at finality, the window a declared per-job parameter. Windowing caps the blast radius of any future client-side mask compromise and satisfies data-residency and harvest-averse buyers.

### Graded

Each piece's instance carries machine-readable grade tags — privacy class (information-theoretic / computational), integrity class (malicious / semi-honest), fidelity class (exact / approximate-ε) — so the proof chain certifies not just that the computation ran but the security class each piece ran in.

A run cannot claim a grade it did not execute; disclosure of maturity is a mechanism, not a slide.

### Open core

Everything required to *trust* the system is public: the contract book (every piece type's compute contract), proof-format spec, chain verifier, seam and anchoring rules, all mechanism code, and a complete reference CPU prover for the pinned model — slow, but able to prove every piece end to end. A commercially licensed GPU prover — **Prism** — accelerates the same specification, held to it by a repo conformance test requiring byte-identical bundles.

Anyone can verify for free; anyone can prove slowly for free; proving fast is a product. The accelerator sits only on the proving path, never the verifying path.

---

## 3. Miner responsibilities and tasks

### The SKU stack

The rail sells three properties that stack. **Deterministic** is the floor and is always on — bit-exact, reproducible execution is what makes the other two layers possible, and is a product in itself (reproducible, audit-grade inference).

**+ Proven** adds transferable proof and settlement: verified inference for plaintext workloads where correctness is the commodity — agent-output verification, benchmark integrity, regulated serving — running on the measured proof lane from day one.

**+ Private** adds masked execution for operator-blind workloads; masked-only jobs settle on the cross-miner determinism vote, which works unchanged on masked outputs.

Stacked together — private, proven, and settled — they are the flagship and the empty cell. An attested enclave wraps any SKU as an optional second wall.

Every job declares its SKU — a choice the client makes per query, so one session can run a sensitive question masked and post the next in plaintext, with the routing decided at the client's gateway before anything touches the network. The contract book's grade classes price each layer; miners serve whichever stack they can perform, on one chassis.

### The wire contract, concretely

A job travels as a synapse pair. Request (to the miner):

    { "job_id": "…", "sku": "deterministic+proven+private", "mode": "gateway",
      "model_manifest": "qwen2.5-0.5b@<hash>",
      "contract_versions": ["rmsnorm-896@v1", "linear-896x4864@v1", "…"],
      "input_commitment": "0x…", "masked_input_uri": "store://…",
      "material_hashes": ["0x…"], "response_target_ms": 2000 }

Response (from the miner):

    { "job_id": "…", "result_commitment": "0x…", "masked_result_uri": "store://…",
      "bundle_hash": "0x…", "bundle_uri": "store://…", "pieces_proven": 24 }

Everything the validator needs for its four checks is in public stores addressed by these hashes; nothing in either message is secret.

The default job is single-miner: accept a masked package (masked input slots, hydrated seam material, job parameters), execute the pinned model piece by piece with no callbacks, prove each piece against the public circuits, post bundle and masked result to the store, commit the hash.

Testnet job shapes: the verified-inference run (deterministic + proven, plaintext — the complete product at launch), the blind threshold check (volume product), and the single-party private analyst run (premium product).

Miners bring any hardware and any performance edge — Prism, or their own implementation of the public spec; correctness is enforced by the mechanism, not implementation trust.

### Extended miner roles: material and proving

Two roles extend the market. **Material manufacture:** masking material is data-independent, so idle capacity manufactures capsule/triple inventory in advance — the network's second commodity, and the reason the online path stays light. Batches carry manufacturer commitments and face sacrificial-sampling verification; manufacturing is blinded so no manufacturer holds a complete correlation, with enclave-certified manufacture as an optional hardware wall on the design's most sensitive supply chain. **Delegated proving (roadmap):** the trace is masked, so proof work is privacy-free to outsource — a piece package tells a third-party prover exactly what the executor knew, which is nothing. Proving becomes its own market (assignment with deadlines and stakes, not racing), small hardware earns on proofs, CPUs on material, execution stays latency-optimized.

Execution itself can later slice across miners — an inter-miner hand-off is just another seam — which is how models too large for any one miner get served; v1 keeps one miner per job for answer latency.

---

## 4. Validator responsibilities and evaluation

### Validators are clients

Validators are clients: they submit challenge jobs through the same masked protocol as paying users, with inputs whose plaintext they know. Masked traffic is indistinguishable, so miners cannot behave selectively for the graders.

### Four checks, all public computation

Evaluation is deterministic public computation, never judgment. Per credited job, four checks: the chain starts at the orchestrator's posted anchor; every seam links; every piece verifies against its declared contract and the manifest; the posted result bytes hash to the terminal commitment — with bundle retrievability as a precondition (**publish-to-be-paid**: no bundle, no credit). Full-chain verification runs in tens of milliseconds, so verification precedes payment. A cross-miner **determinism vote** adds a free first tier: honest miners are byte-identical, so hash disagreement flags lazy work at zero cost, with proof audits resolving every disagreement.

Validators also verify attestations at admission (zkTLS web proofs binding input fields to their HTTPS sources; optional TEE quotes), sample material batches, retain and serve every bundle they credit through the challenge window, and escrow mask-share exchange in two-party jobs — settling both markets: compute in TAO on proof, meaning in shares on policy.

### Collusion and the public check

Miner and validator can collude; the design makes it unprofitable and convictable.

On privacy, under the declared masking assumptions, a colluding miner and validator gain no plaintext from their combined protocol view: neither ever holds a mask, so together they know what each knew alone — nothing.

On payment, every crediting decision is re-executable by anyone, because verifier, circuits, and bundles are public: a false credit is a checkable artifact. Anyone presenting a **fraud proof** — a credited job whose published bundle fails the public verifier, or was never published — triggers slashing of the miner's job stake and, more severely, the crediting validator's stake: majority burned, minority paid as bounty (burn-dominant, so sock-puppet self-reporting is unprofitable; bounty-positive, so watching — which costs milliseconds per job — is profitable).

Because verification is nearly free, watchers can check every credited job, driving fraud's expected cost toward its full slash rather than slash-times-detection-probability.

### Availability

Unavailability is the one non-re-executable offense, so it is bonded and observed rather than proven: a retention slice of the miner's stake backs the window, and availability challenges are **scheduled and mandatory** — beacon-timed retrievals per bundle, logged, so "served" is affirmative evidence at named times, adjudicated by validator quorum.

The validator's duties thus split into two accountability classes: deterministic checks any watcher can re-run per job under the bounty layer, and observed duties (availability, latency) that are bonded and quorum-attested.

Proof fraud is caught by one laptop; availability fraud requires a quorum to lie; neither requires trusting validator judgment, because every duty is a computation or a logged observation.

The honest residue: challenge inputs derive from a public randomness beacon plus a committed seed revealed after grading; latency's score weight is capped; what remains is Bittensor's inherited validator-stake consensus — with this subnet's distinction being that validator deviation is provable, not merely statistical.

### Client self-verification

The client need not trust the validator either: its orchestrator fetches the bundle by hash and runs the identical public verifier before unmasking — verify-then-unmask by default, so the client never sees an unverified answer; a fast mode may unmask at answer time and confirm at chain close, voiding payment retroactively on failure.

### The attack surface, compressed

| Attack | Defense | Residual assumption |
|---|---|---|
| Lazy miner | Determinism vote + proof audit | None |
| Tampered intermediate | Seam commitment continuity | None |
| Wrong model or function | Manifest + circuit binding | None |
| Fake result | Terminal commitment vs delivered bytes | None |
| Withheld publication | Publish-to-be-paid | Availability window |
| Corrupt crediting | Public verifier + watcher bounty | Watcher participation |
| Miner–validator collusion | Masking + re-executable adjudication | Masking assumptions |
| Material compromise | Blinded manufacture + sampling, optional enclave | Manufacturer non-collusion |
| Unavailability | Retention bond + scheduled retrievals | Quorum honesty |
| Repeated blind probing | Fresh-material consent + pair rate limits | Policy parameter |


---

## 5. Incentive and reward mechanism

### Settlement flows from the edges inward

Rewards flow **per proven piece**, and settlement streams from the edges inward. A prover's piece fee is final when its proof verifies — honest piece work stays paid even if the assembly later breaks elsewhere; its proof is what convicts the tamperer.

A material fee is final when the consumption record verifies, forfeited retroactively on a bad sampled batch.

The executor's fee clears last, at chain close, because its product is the assembly: anchored start, linked seams, delivered terminus.

With streaming proving, most of a job is proven and cleared by the time the answer lands; the smallest jobs settle end to end in near-real time.

Timing is three-layered: the credit ledger updates per proof immediately; on-chain emission realizes credits at epoch tempo; escrowed client fees release per job on proof.

### Client fees settle on the same event as emissions

At testnet: a client's job fee sits in escrow and releases to the miner's hotkey on the validator's chain-verify — subsidy and revenue settling on one mathematical event, demonstrated live.

At mainnet: enterprise billing follows the model the ecosystem's highest-earning subnet has already proven — customers pay in fiat or stablecoins through ordinary API metering, and platform revenue auto-buys the subnet's alpha, converting product usage into staking demand and emissions share.

Natively, per-job on-chain escrow (taoUSD via the EVM bridge) releases on proof and claws back on fraud proof within the challenge window; in two-party jobs the same escrow doubles as the mask-share exchange.

And this fee rail carries a property no revenue subnet today can claim: external subnet revenue is currently structurally unauditable, because the chain records token movements but not API calls — whereas here every paid job leaves a public proof bundle with an on-chain hash, so fee escrows keyed to bundle hashes make the revenue ledger itself cryptographically checkable by anyone with the public verifier.

The proof is the invoice, so the invoices are provable.

Failed proofs and determinism defection zero the job's reward and apply a multiplicative integrity penalty; repeated failure decays the miner toward zero weight.

Payment requires proof, proof requires the anchored commitments and pinned weights, and every credit is publicly re-checkable under bounty — so there is no reward channel for fabricated work and no durable one for corrupt crediting.

Leakage is priced at the mechanism layer: every job consumes fresh material from all input owners, making participation per-query consent by construction, and per-counterparty-pair rate limits stop repeated blind queries from binary-searching a counterparty's secret.

---

## 6. Scoring methodology

    S_i = ( Σ_j w_j · v_ij / T ) × M_i × L_i

Where: **v_ij** = 1 if piece j from miner i verified against its published contract, else 0; **w_j** = that piece type's declared computational weight, derived from its public circuit's constraint count — objective, and recountable by anyone; **T** = the challenge measurement window; **M_i** = the integrity multiplier; **L_i** = the capped latency factor. In words: constraint-weighted verified pieces per second, times integrity, times latency.

Piece throughput is normalized by each piece type's declared computational weight, derived from its public circuit's constraint count — an objective, recountable difficulty measure — so miners cannot farm trivially cheap pieces at the expense of useful capacity. The integrity multiplier starts at 1, is zeroed for the epoch on any verified proof failure, determinism defection, or availability slash, and recovers over clean epochs.

The latency factor rewards the job class's declared response target (threshold checks latency-scored; analyst runs throughput-scored; proofs may trail answers within a declared window), with its weight capped so the only validator-subjective input cannot dominate the objective ones.

Weights are set from normalized scores; the market is continuous and multi-winner, because the commodity is capacity, not a single best artifact.

### The four incentive experiments

Evidence that incentives behave as intended will be produced on testnet as four named experiments: a **lazy miner** (plausible garbage) caught by the determinism vote with zero audits; a **splicing miner** (substitutes an intermediate) caught at the broken seam, localized by binary search, payment voided while its honest provers stay paid; an **honest miner** paid in proportion to proven throughput across mixed challenge and organic traffic; and a **watcher catch** — a deliberately mis-credited job caught by an independent watcher running the public verifier, bounty paid.

Target scale: three validators, roughly ten miners.

---

## 7. Expected users and ecosystem value

### The buyers

The buyers mirror the two abilities in section 1.

For private use of a model, the buyer is anyone whose question is the secret. Individuals asking medical, legal, or financial questions they would never type into a watched service. Professionals bound by privilege and confidentiality — attorneys, physicians, auditors — for whom sending client or patient matter to a readable inference service is an ethics violation, not a preference. Firms whose prompts are themselves trade secrets: code, strategy, research direction. This is the volume market, and it needs no sales motion beyond existing demand for AI — it is the same demand, minus the watcher.

For transacting on secrets, the first buyers are financial institutions with computations that disclosure currently forbids or degrades: M&A and investment analysis over material non-public information; blind eligibility and know-your-business (KYB) checks where an applicant proves qualification without opening its books and a screener's criteria never leave the building — upgraded from self-report to source-attested by zkTLS-bound input fields; reinsurance submission triage, where a cedent tests a program against a published appetite before exposing anything.

This is the premium market and the beachhead: these are concrete, well-understood workflows in insurance, reinsurance, banking, and corporate finance, and the blind eligibility and submission-triage shapes map directly onto existing underwriting and deal processes, with no workflow invention required.

We are not guessing at this buyer — we are this buyer. The founding team has spent twenty-five years building and governing AI inside regulated insurance and reinsurance, working under exactly the confidentiality constraints section one describes: the questions that never get asked are questions we personally could not ask. Blindproof is the product its builders could not buy.

### Ecosystem value

The subnet serves a demand already validated at eight figures by the hardware-trust tier, at a trust level no incumbent occupies, with the blind threshold check as the instantly-settling volume on-ramp; the contract book, verifier, and proof format are open public goods useful to the TEE tier as well — and the same published contracts a miner performs against are what an enterprise compliance function audits against, one document serving both markets; and the material market gives non-latency-competitive capacity a paying role, broadening who can mine.

Token value accrues against job fees on both commodities.

Longer term, two-party mode makes the network settlement infrastructure for blind deal discovery — the transactions that today die because approaching the counterparty leaks the secret.

### Designed for the ecosystem's hardest question

Independent analysis of subnet economics makes two charges that this proposal was designed to face: external revenue across the network is small against emissions, and — worse — it is structurally unverifiable, because chains record token movements, not API calls; nearly every revenue figure in the ecosystem is a claim its own subject made, and the sole independently verifiable demand number exists only because one subnet happens to route through a third-party platform that publishes throughput.

Blindproof answers the second charge by construction: every paid job leaves a public proof bundle with an on-chain hash, fee escrows key to bundle hashes, and the revenue ledger is therefore recomputable by anyone running the public verifier — verifiable demand as a design property of every job, not a lucky accident of one integration.

We commit to the rule that follows: we will never publish a usage or revenue number that cannot be independently recomputed.

And the first charge — subsidy ratios, where the flagship compute subnets sell commodity inference below centralized prices and depend on emissions for the difference — does not transfer to a differentiated commodity: no centralized provider sells trustless private inference at any price, so Blindproof's pricing rests on capability rather than subsidy, and our own emission-dependency ratio will be publicly computable in real time from the same auditable ledger — converting the ecosystem's most uncomfortable hidden number into a transparency feature.

### Where this points

The abilities in section 1 are this mechanism at scale, and the scaling is fractal: the dispatcher that splits 24 pieces across two testnet miners splits a larger model's pieces across four hundred — same seams, same circuits, same verifier, and verification cost stays flat while serving cost distributes.

The client's burden scales with activations, not parameters, so the laptop endpoint holds at any model size — conditional on a blinded material supply chain whose manufacturer non-collusion is, deliberately, the first item on our own stated attack list.

The declared open frontier is material economics: offline material scales with the model (measured at gigabytes per token at 7B, attention-dominant), which is why the material market is an economy rather than a cache, why batch inference is the first frontier-scale product shape, and why material compression is named future work.

---

## 8. Roadmap toward testnet and future deployment

### At this checkpoint

This proposal; the mechanism specification (scoring formula, anchoring rules, bounty and availability economics, collusion table, storage policy); working piece/seam toy protocols with concrete numbers; a proof system and verifier already measured on a mid-size open-weight model.

### To final (Oct 19) 

- subnet scaffolding on testnet.
- miner implementing deterministic 0.5B serving, per-piece proof bundles, store posting, hash commitment.
- validator implementing beacon-seeded challenge generation, determinism vote, the four anchored checks, publish-to-be-paid crediting, weight-setting.
- a public bundle bulletin as the testnet store.
- public repository with all mechanism code, the contract book, the reference CPU prover, the proof-format spec, the conformance suite, and deployment instructions.
- the four incentive experiments run and documented.
- the mode split proven at its honest grades — gateway mode end to end (masked job in, blind execution, verified answer out, settled), capsule mode as demonstrated elements (non-interactive masked pieces with hydrated seam material, grade-tagged).
- a demo video showing one job end to end — masked input anchored, blind execution, bundle published, chain verified by the validator and independently by the client's orchestrator, payment settled — plus the piece-level private-lane demonstration (masked attention computing on data the operator cannot read, grade-tagged), the splice-and-reject beat, and a two-miner dispatch of one job as the scaling fractal

### Evidence grades, stated plainly

Measured today, on a mid-size open-weight model: the deterministic proof lane — per-layer proofs, tens-of-milliseconds chain verification, single-byte tamper detection, cross-GPU bit-exactness.

Demonstrated at piece scale: the masked private lane, including two-input masked multiplication with independent mask owners.

Designed against measured budgets: the private lane at full model scale, whose dominant cost — offline material — is quantified and motivates the material market.

Declared as interface: zkTLS and TEE attestations.

Phase one proves pieces in the deterministic lane while demonstrating privacy at piece granularity, grade tags keeping every claim exactly as strong as its evidence; the convergence — proof circuits over the masked-domain trace, beginning with the linear pieces, the simplest circuits in the catalog — is the named phase-two work: engineering on a shared piece skeleton, not redesign.

### Post-hackathon

- The fact ledger: proven results are reusable, so the permanent bundle store doubles as a verified-inference cache — a settled answer is retrieved and re-verified in milliseconds instead of recomputed, opening a facts market on the same settlement rail (compute novel facts, resell settled ones, royalties by on-chain priority). Plaintext disclosure is a priced per-query client choice, rebated by resale royalties; private jobs never touch the cache, by construction and as a feature.
- Phase-two convergence.
- Two-party no-plaintext jobs with escrowed disclosure.
- Delegated proving and material markets as distinct miner roles.
- ZkTLS integration.
- The attested-enclave tier (crypto floor, hardware armor).
- Network-side untrusted coordination once full-depth non-interactivity is confirmed.
- Model scale-up along the identical architecture (the 0.5B and 7B+ models share one per-layer op graph, so scaling claims are extrapolations on one circuit).
- Mainnet registration, with regulated-industry enterprise pilots as the first sustained-demand target.
