# Adversarial review of the Sain threat model

Date: 2026-07-31

Reviewer: Codex, performing the project-requested review in this workspace.

Independence disclosure: this review was performed separately from the original implementation work, but by the same active engineering session and without an outside organization. It is an internal adversarial review, not the independent third-party review required by full-vision audit item V-01.

## Verdict

The threat model correctly refuses to equate hashes with signatures, replay with truth, or local checkpoints with disaster recovery. Its principal weakness is that it has become stale while controls were added. Several new controls are real and tested, but some exist as libraries rather than mandatory gates in the authoritative organism path. The largest residual risks remain compromised-host resealing, incomplete integration of governance and physical I/O protocols, project-controlled evaluation, supply-chain ambiguity, and denial of service.

No current evidence supports autonomous high-consequence physical deployment or a claim of broadly superior intelligence.

## Review method

The review compared each stated asset, boundary, adversary, control, and acceptance test with the Rust and Python implementations. It included targeted mutation tests, inspection of independent verifier boundaries, inspection of state/capsule serialization, and review of the full-vision audit. It did not include penetration testing of a deployed service, hardware attacks, external institutional review, or source review of every transitive dependency.

## Findings

### RV-001 — Controls implemented as optional libraries are not authoritative gates (critical)

The signed GovernanceCharter, actuation transition protocol, complete intelligence-capsule wrapper, resource-efficiency selector, grounded world model, and neuro-symbolic protocol are testable modules. Several are not yet mandatory in every relevant `NativeOrganism` transition. An attacker or ordinary legacy path may bypass a control without violating the older state schema.

Required remediation: define one authoritative transition dispatcher; require the applicable governance, epistemic, resource, simulation, and actuation receipt before state persistence; reject legacy bypass for newly created authoritative states.

Acceptance test: enumerate every mutating command and prove that no command reaches persistence without the required typed receipts.

### RV-002 — Writable-host attackers can reseal most hash-only objects (critical)

Domain-separated digests detect accidental or uncoordinated mutation. They do not authenticate origin when the attacker can run the sealing code. Event signatures improve this only when a separately protected key and pinned trust root are configured. The current host, environment, binary, and many local files remain one trust domain.

Required remediation: mandatory signed transitions after a declared activation generation, hardware or threshold key protection, externally anchored heads, and recovery that checks the external anchor.

Acceptance test: a process with write access to all project files but without threshold key shares cannot create a state accepted by an offline verifier.

### RV-003 — Canonical serialization remains a cross-language hazard (high)

The merge work exposed a concrete defect: an embedded Rust struct was committed in declaration order, but its keys were reordered after conversion through a JSON value, causing the independent Python verifier to reconstruct different bytes. That defect is fixed for promotion manifests, but the pattern can recur wherever commitments depend on incidental map or struct order.

Required remediation: publish a single canonical encoding specification, use it for all new commitments, and add differential vectors for every committed schema.

Acceptance test: Rust, Python, and one additional implementation produce identical bytes and digests after parse/serialize cycles and reject duplicate keys, unknown fields, noncanonical numbers, and reordered nested objects where order is not semantic.

### RV-004 — Approximate/exact certification is narrow (high)

The numerical boundary soundly covers a bounded binary64/quantized-linear profile with exact integer policy decisions. It does not cover transformer attention, normalization, nonlinear activations, arbitrary accelerator kernels, training, or analog devices.

Required remediation: certify a quantized transformer block end to end, specify kernel and rounding profiles, and compare at least two hardware implementations against the same exact acceptance relation.

Acceptance test: cross-device results remain within a committed bound and authoritative decisions replay identically under an independent verifier.

### RV-005 — Physical sensing and actuation are protocol fixtures, not trusted reality (critical before deployment)

The new world and actuation schemas bind the right categories and reject cross-receipt substitution. They do not prove calibration, placement, device health, command execution, or outcome in a physical environment. Digest-shaped identities and receipts can be fabricated by the same host.

Required remediation: hardware-rooted device identity, signed calibration lineage, raw measurement retention, independent corroboration, actuator acknowledgements, and discrepancy monitoring on an isolated physical testbed.

Acceptance test: swapped sensors, stale calibration, replayed commands, forged acknowledgements, and outcome substitution fail at the offline verifier.

### RV-006 — Project-controlled evaluation cannot establish open-world capability (high)

The challenge, meta-evaluation, Observatory, and governed-promotion protocols are meaningful anti-tamper controls. Their authored challenges and trust roots remain project controlled. Repeated development on visible benchmarks permits evaluator overfitting.

Required remediation: independently administered private challenges, precommitted criteria, one-shot frozen candidates, external baselines, null-result retention, and cross-institution replay.

Acceptance test: at least two outside operators independently reproduce both a confirmation and a falsification without using project-held private keys.

### RV-007 — Supply-chain and runtime identity are incomplete (high)

Provenance export supports W3C PROV and SLSA/in-toto, but that does not itself make builds hermetic or reproducible. Compiler, build flags, base system, optional model runtime, and release approval are not uniformly bound to every transition.

Required remediation: hermetic build definition, SBOM, signed release manifest, independent rebuild, dependency audit, and mandatory binary/policy identity in transition statements.

Acceptance test: independent builders reproduce the declared artifact or a narrowly specified equivalence proof, and the runtime refuses an unlisted binary.

### RV-008 — Secrets and private data remain co-resident (high)

The workspace visibly contains environment configuration, TLS key material, uploads, databases, models, logs, and large organism state. File permissions and ignore rules are not process isolation. Diagnostics, backups, model output, or a compromised connector may disclose them.

Required remediation: secret-service integration, per-component identities and filesystem views, pre-persistence redaction, retention enforcement, canary-secret tests, and encrypted independent backups.

Acceptance test: seeded secrets cannot be recovered from any UI, model response, memory capsule, log, crash artifact, backup, or connector output.

### RV-009 — Resource evidence is incomplete and partly operator declared (medium)

Elapsed time, CPU ticks, peak RSS, I/O, and optional RAPL energy are bound. On this review host no readable energy counter was available, and the honest benchmark recorded that absence. Carbon, water, and PUE may be operator-declared. Kernel-enforced budgets and complete heat, wear, network, and proof-overhead measurements are not universal.

Required remediation: attested meters where available, explicit unavailable states elsewhere, cgroup or equivalent hard budgets, network accounting, and promotion policies that reject missing mandatory telemetry for high-cost work.

Acceptance test: adversarial workloads remain within declared budgets and every unavailable metric is explicit rather than estimated silently.

### RV-010 — Parser and state-size denial of service remain plausible (high)

The authoritative state and journal are large and verification commonly reads complete files. Capsule and JSON parsing can allocate before semantic limits are checked. Several vectors have local caps, but there is no uniform byte-size, recursion-depth, event-count, or verification-time envelope before parsing.

Required remediation: bounded streaming parsers, file-size checks before reads, nesting limits, journal segmentation, resumable verification, and cancellable resource quotas.

Acceptance test: oversized, deeply nested, duplicate-key, decompression, and long-journal corpora fail within fixed memory and time bounds without changing authoritative state.

### RV-011 — Recovery lacks an independent failure domain (high)

Atomic persistence and checkpoints address interruption, not correlated deletion, ransomware, or hostile-host rewriting. Portable capsules improve mechanics but are not backups unless exported and anchored elsewhere.

Required remediation: encrypted append-only backups under independent administration, periodic isolated restore drills, quantified recovery objectives, and external head anchoring.

Acceptance test: destroy the live host and recover to the externally anchored head using only offline materials.

### RV-012 — Audit completion can itself become proof theater (high)

The full-vision gate correctly refuses completion while any requirement is merely bounded, absent, or external. Its JSON evidence is still authored within the project; changing a status and test expectation is not proof that the underlying claim is true.

Required remediation: evidence objects must identify executable tests, artifacts, independent signers where required, dates, environments, and expiry. External requirements must never be converted to satisfied solely by internal code or prose.

Acceptance test: the completion gate independently resolves every evidence reference and refuses stale, missing, self-signed-when-independent, or scope-mismatched evidence.

## Priority order after review

1. Make the new governance, capsule-manifest, epistemic, simulation, resource, and actuation protocols mandatory at the authoritative transition dispatcher.
2. Publish and differentially test one canonical commitment encoding.
3. Enforce signed transitions with protected keys and an independently anchored head.
4. Add bounded streaming parsing and kernel-enforced resource ceilings.
5. Produce a hermetic signed release and independent rebuild.
6. Run external threat review and independently administered challenge/reproduction ceremonies.
7. Attach a calibrated sensor and reversible actuator in an isolated testbed before any consequential deployment.

## Residual-risk decision

Deployment remains acceptable only for local research and bounded, reversible experiments. Network, secret access, promotion, and physical actuation must remain denied by default. The project should not claim V-01 complete until an outside reviewer signs a review artifact and the project publishes its disposition of every finding.
