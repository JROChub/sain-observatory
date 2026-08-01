# Independent review request

Sain needs review by people and organizations that do not answer to the
project. The maintainer is explicitly requesting adversarial review of the
published threat model and challenge of the project's evidence.

The files in this directory are a frozen review packet. The internal review is
included to show known findings; it is **not** represented as independent.

## Useful review outcomes

An external reviewer can contribute any of the following:

1. Identify a missing asset, adversary, trust boundary, abuse case, or failure
   mode in `THREAT_MODEL.md`.
2. Reproduce or falsify a stated control or test.
3. Assess whether an internal finding or residual-risk decision is wrong.
4. Supply a competing public or commit-before-reveal private challenge.
5. Propose a safer anti-coercion, contestation, or institutional federation
   protocol.

Use the Observatory challenge form. State your relationship to the project,
method, evidence, limitations, and disposition (`confirmed`, `falsified`,
`inconclusive`, or `contested`). A name or organizational affiliation is
welcome but not required; independence cannot be inferred from a GitHub handle
alone.

No project-controlled response can satisfy the independent-review requirement.
The maintainer will preserve adverse findings and must not label a submission
"independent institutional review" without reviewer consent and a disclosed
conflict-of-interest statement.

## Packet digests

```text
9e8fa7e94af2b3d41c14bca70fad5896799fbc91f60cef01ce7d14aa7df68f99  THREAT_MODEL.md
9c76b6ea300e71e93583c4e27983e5c237876321ae6c64bb275d9acfe86fdbe7  INTERNAL_ADVERSARIAL_REVIEW.md
cd525e7c19d22d66af6812e526a2566b6305026123fd6205f6d0f30d8749c2b1  full_vision_audit.json
```
