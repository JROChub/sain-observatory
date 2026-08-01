# Sain Public Observatory

This repository is the globally reachable, public challenge intake for bounded
Sain claims. Anyone with a GitHub account can submit a counterexample,
reproduction request, alternative dataset, adversarial environment, safety
probe, causal intervention, or branch-inconsistency report through the issue
form.

Each accepted issue remains public and triggers a pinned GitHub-hosted workflow
that commits to the issue body, creates a data-minimized intake receipt, signs
that receipt through GitHub OIDC and Sigstore, verifies the workflow identity,
and attaches the evidence to the run. The issue receives the receipt digest and
run link.

Do not submit secrets, personal data, private datasets, exploit payloads, or
instructions for harmful physical action. Commit to private evidence with a
SHA-256 digest and provide a safe disclosure process instead.

## Scope and independence

This service makes Sain independently challengeable; it does not make the
maintainer or GitHub an independent scientific institution. Intake proves
submission history and workflow execution, not that a claim or challenge is
true. Dispositions must preserve confirmed, falsified, inconclusive, and
contested outcomes rather than silently deleting unfavorable results.

The authoritative machine-readable service descriptor is observatory.json.

An explicit packet for independent threat-model and evidence review is in
[`review/`](review/README.md). Project-controlled comments do not count as
independent review.
