# Sain 7056 threat model

Status: internal engineering threat model, 2026-07-31; not an independent third-party review

This document evaluates the authoritative Rust organism and its local
persistence boundary. It is intentionally stricter than the capability
verification matrix. Passing a benchmark is not evidence that the benchmark,
runtime, host, operator, or claimed real-world conclusion is trustworthy.

## Security objectives

Sain should eventually make five independently testable guarantees:

1. **Computational integrity:** a defined transition was evaluated correctly.
2. **Provenance integrity:** inputs, code, models, policies, and predecessor
   states are identified without silent substitution.
3. **Origin authenticity:** an authorized principal or device created the
   record and cannot be impersonated by merely recomputing a hash.
4. **Epistemic validity:** claims preserve evidence type, uncertainty,
   assumptions, and falsifiers instead of treating replay as truth.
5. **Normative authorization:** capabilities and actions are explicitly
   permitted by a machine-enforced constitution.

The current implementation provides useful local deterministic replay,
invariant validation, domain-separated SHA-256 commitments, a hash-linked
journal, a mirrored checkpoint, bounded typed execution, and a deny-by-default
network policy. It does not yet establish objectives 3–5, and it establishes
objectives 1–2 only within the trusted local implementation and host.

## Protected assets

- Persistent identity and lineage continuity.
- Cognitive genomes, learned numerical state, memories, causal models, and
  evaluation seals.
- Event ordering, benchmark preregistration, and promotion decisions.
- Network and future actuator authority.
- Model/runtime artifacts used by the optional language cortex.
- Human credentials, private data, challenge data, and unreleased evaluations.
- Availability of rollback state and the ability to distinguish corruption
  from an authorized transition.

## Trust boundaries

```text
untrusted world / user data / web
              |
              v
      network + ingestion membrane
              |
              v
  Rust process and optional Python UI ------ local model process
              |
              v
 state JSON <-> checkpoint JSON -> append-only-in-name journal
              |
              v
  power_house and HOP-Corr dependencies / compiler / operating system
```

The process, operating system, filesystem owner, compiler, linked crates, and
the code defining every verifier are currently trusted. The state and journal
are not externally anchored. The checkpoint is a second local copy, not an
independent trust domain.

## Adversaries

- Accidental corruption, interrupted writes, disk loss, and operator error.
- Malformed or adversarial inputs attempting denial of service or semantic
  contamination.
- A remote caller with UI credentials.
- A compromised dependency, compiler, local model executable, or host process.
- A local attacker able to read and write organism files.
- A benchmark author or self-modification process able to shape evaluators,
  leak answers, exploit selection bias, or optimize the metric rather than the
  intended property.
- A malicious sensor, institution, validator majority, or external data source.

The initial target does not claim resistance to a fully compromised operating
system. That requires hardware-rooted attestation and an independent verifier.

## Principal findings

### TM-001 — Origin authentication (partially mitigated; residual high)

Most fields named `proof_digest` remain domain-separated SHA-256 commitments.
The runtime now supports Ed25519 transition attestations that bind identity,
generation, state digest, lineage, event and predecessor heads, experiment,
capability, and artifact commitment. Verification can pin an external trust
root, enforce a signature-activation generation, reject downgrade/key
substitution, and follow old-key-authorized rotation certificates. Unsigned
legacy events are counted rather than retroactively represented as signed.

Residual risk remains high when signing is not configured, when the trust root
is learned from the same writable journal, or when a host attacker can read the
software signing key. Independent anchoring and hardware/threshold protection
are still required for hostile-host resistance.

Required controls:

- Define a versioned attestation envelope distinct from the artifact payload.
- Sign accepted transitions with an offline-rotatable identity key or a
  threshold authorization set.
- Anchor selected heads in an independently operated transparency service.
- Bind verifier version, build identity, policy identity, and trusted-component
  measurements into every signed statement.
- Retain the present hashes as content identities; do not rename signatures as
  hashes or hashes as signatures.

Acceptance test: rewriting and fully resealing local state without possession
of an authorized signing capability must be rejected by an independent
implementation.

### TM-002 — Verification is self-referential (critical)

The same codebase generally creates and verifies benchmark reports. A defect or
compromise shared by producer and verifier can certify an invalid result. The
current Power House binding does not remove that trusted-code dependency.

Required controls:

- Publish canonical schemas and transition semantics independent of Rust
  implementation details.
- Build a second verifier in a different language and organization.
- Add known-answer vectors, negative vectors, differential fuzzing, and
  mutation corpora.
- Use a real proof system only where its statement and trusted setup are
  explicit; continue to state that the proof does not establish truth or
  intent.

Acceptance test: independently developed verifiers accept every valid corpus
item, reject every adversarial mutation, and agree on canonical identity.

### TM-003 — State and journal commit atomicity (mitigated locally; residual medium)

The native runtime now prepares a committed transaction containing the next
state record and event, installs state and checkpoint atomically, appends the
event, and removes the transaction only after synchronized completion. Startup
idempotently completes transactions left at each durable stage. The remaining
risk is correlated storage/host compromise and the absence of an independently
anchored recovery domain.

Required controls:

- Specify a write-ahead transaction protocol with sequence numbers.
- Persist a prepared record, sync it and its directory, commit state, then mark
  the transaction committed.
- On startup, deterministically complete or roll back interrupted transactions.
- Fault-inject at every write, sync, rename, and directory-sync boundary.

Acceptance test: process termination at every persistence boundary yields
exactly the prior state or exactly the next state, never an ambiguous lineage.

### TM-004 — Concurrent writer exclusion (mitigated locally; residual medium)

Every state mutation now takes an operating-system advisory lock, recovers any
prepared transaction, and compares both persisted-state and journal heads with
the proposed predecessor before preparing the next transition. Stale writers
fail closed. The residual risk is a writer outside the protocol or a
compromised host, which requires signed transitions and external anchoring.

Required controls:

- Acquire an exclusive, owner-identifying lease for every state mutation.
- Re-read and compare the expected state head inside the lease.
- Reject rather than silently merge competing writers.
- Define an explicit fork object for intentional concurrency.

Acceptance test: hundreds of competing writers produce one accepted linear
head and explicit rejected/forked transitions with no lost accepted event.

### TM-005 — Constitutional coverage is incomplete (critical before actuation)

A proof-bound, deny-by-default constitution now separates proposal, evaluation,
promotion, network observation, signing, key rotation, policy change, resource
spending, and physical action. Cognitive-dynamics promotion and attestation-key
rotation exercise scoped tokens; physical actuation has no grant. This is not
yet a complete boundary: other learning and architecture mutation paths remain
outside the promotion ledger, principals are not yet backed by independent
signing domains, and environment variables and process authority remain ambient.

Required controls:

- Introduce deny-by-default capability tokens scoped by action, resource,
  duration, budget, and predecessor policy head.
- Keep proposal, evaluation, promotion, and action signers separate.
- Require stronger quorum and evidence for irreversible actions.
- Make policy changes create reviewable forks with delay and rollback.

Acceptance test: a model or module with proposal authority cannot promote its
own update, open the network, access secrets, or actuate a device.

### TM-006 — Execution evidence can be mistaken for factual evidence (critical)

New memory entries now carry a closed epistemic type plus proof-bound source
references, assumptions, uncertainty, falsifiers, expiry, and revocation.
Unknown labels fail closed, elevation to observation or policy is forbidden,
and expired or revoked entries are excluded from retrieval. Legacy entries are
explicitly classified as `legacy_unclassified`. Residual risk remains because
free-form evidence enters the classifier before an authoritative typed-ingest
API, and sensor calibration or external truth is not established by the type.

Required controls:

- Replace free-form evidence labels at authoritative boundaries with a closed,
  versioned epistemic type system.
- Bind claims to sources, measurement uncertainty, assumptions, derivations,
  counterevidence, falsifiers, expiration, and revocation.
- Reject inference-to-observation and proposal-to-authorization promotions.

Acceptance test: property tests and adversarial migrations cannot elevate a
hypothesis, model output, or simulation into direct observation.

### TM-007 — Benchmark and evaluator capture (high)

Many current results are bounded synthetic tests generated and judged inside
the same project. They are valuable regression evidence but cannot establish
open-world intelligence. Repeated development against visible tasks creates
selection pressure for test-specific behavior.

Required controls:

- Commit evaluator identity and promotion criteria before revealing challenge
  data.
- Separate development, validation, private challenge, and post-deployment
  surveillance distributions.
- Preserve negative and null results.
- Require external baselines, independent challenge authors, compute budgets,
  and correction for repeated selection.

Acceptance test: frozen candidates are evaluated once against independently
held challenges and reproduce under a second institution's verifier.

### TM-008 — Network policy is syntactic, not a complete connector sandbox (high)

The native membrane is closed by default and rejects literal non-global IPs,
which is a sound starting point. Host allowlisting alone does not bind DNS
resolution, redirects, proxy behavior, TLS identity, response size, content
type, or connector capability. DNS rebinding and a compromised allowlisted
host remain relevant once network execution exists.

Required controls:

- Resolve and validate every address at connection time and after redirects.
- Pin the validated address/host relationship for the request.
- Bound bytes, time, redirects, decompression, content types, and concurrency.
- Run connectors out of process with no state-write or signing capability.
- Preserve response provenance and treat content as adversarial data.

Acceptance test: rebinding, redirect, proxy, archive-bomb, slow-response, and
prompt-injection corpora fail closed without contaminating authoritative state.

### TM-009 — Local rollback is not durable disaster recovery (high)

The checkpoint mirrors the state on the same trust boundary. A hostile writer,
filesystem loss, ransomware event, or correlated hardware failure can replace
or destroy state, checkpoint, and journal together. Portable schema-v2 capsules
now package and independently verify the complete state and journal, support
explicit forks and migration, bind evolved branches to an exact parent capsule
and byte-identical journal prefix, restore atomically into a new directory, and
use an append-only revocation registry. This improves portability and recovery
mechanics but does not create an independent failure domain or external head
anchor by itself.

Required controls:

- Maintain encrypted, versioned, append-only backups in independent failure
  domains.
- Periodically restore into an isolated verifier.
- Anchor heads externally and document key-recovery procedures.
- Define retention, deletion, privacy, and cryptographic-erasure rules.

Acceptance test: loss of the live host can be recovered to an externally
anchored head with quantified recovery point and recovery time.

### TM-010 — Supply-chain identity is incomplete (high)

Crate versions are pinned in the native manifest/lockfile, and the optional
model has an expected digest in code, but the running binary, compiler,
dependencies, build flags, operating system, and release approval are not bound
to each organism transition. Reproducible and signed releases are not currently
demonstrated by this workspace.

Required controls:

- Produce SBOM, SLSA/in-toto provenance, hermetic build instructions, and
  signed release manifests.
- Bind the binary and policy digest into transition attestations.
- Run dependency audit, license policy, and malicious-package review gates.
- Reproduce releases on independent builders.

Acceptance test: two independent builders produce the specified canonical
artifact or a documented equivalence proof, and runtime attestation identifies
that artifact.

### TM-011 — Secrets and private data share the project trust domain (high)

The workspace contains local environment configuration, TLS key material,
uploads, datasets, databases, model state, and logs. Ignore rules reduce
accidental version-control inclusion but do not isolate runtime readers or
prevent disclosure through diagnostics, learned memory, backups, or generated
output.

Required controls:

- Move secrets into an OS or hardware-backed secret service.
- Give each component a minimal filesystem view and dedicated account.
- Redact before persistence, not only before display.
- Add secret scanning without printing matched values, data classification,
  retention policy, and machine-enforced export controls.

Acceptance test: seeded canary secrets cannot be retrieved through UI, memory,
logs, model output, crash reports, backups, or network connectors.

### TM-012 — Resource enforcement and physical telemetry are incomplete (medium)

Searches are bounded in several VMs, but state JSON is large, verification can
be expensive, and CLI benchmarks can consume substantial CPU. Every new
authoritative transition now carries a proof-bound receipt for monotonic elapsed
time, process CPU ticks, peak resident memory, and process I/O. Hardware joules
are included only when a readable energy counter exists; carbon, water, and PUE
are explicitly labeled operator-declared context. These receipts measure and
expose cost but do not yet impose kernel-enforced budgets or attest the physical
meter and environmental declarations.

Required controls:

- Enforce per-capability CPU, wall-time, memory, I/O, and output budgets.
- Measure actual resource use and bind it to the transition record.
- Apply quotas before allocation and use resumable, externally cancellable jobs.
- Include verifier and proof-generation cost in promotion decisions.

Acceptance test: adversarial requests cannot exceed declared budgets by more
than a documented tolerance and leave the authoritative state recoverable.

## Risk-prioritized implementation order

1. Publish the assurance taxonomy in machine-readable verification output.
2. Specify canonical transition and attestation envelopes.
3. Add single-writer transactional persistence and exhaustive crash injection.
4. Add signatures, key rotation, and independent head anchoring.
5. Build a second verifier and conformance corpus.
6. Add epistemically typed claims and forbidden promotion rules.
7. Add constitutional capability separation before any real actuation.
8. Add isolated connectors, reproducible builds, resource receipts, and
   independently administered challenge infrastructure.

## Non-claims

This threat model does not claim that the current organism is general,
superhuman, conscious, safe for autonomous real-world action, or protected from
a compromised host. It also does not diminish the value of existing bounded
experiments. It specifies what their evidence can support and what must be
invented next.
