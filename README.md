# `did:me` Method Spec

`did:me` is a decentralized identifier method designed for high-assurance deployments, with integrated post-quantum cryptography support and compatibility with SD-JWT and zero-knowledge proof workflows.

This repository contains the official normative specification, JSON-LD context, and related method materials for `did:me`.

## What Is did:me?

`did:me` is a stable, non-key-derived DID method for long-lived identifiers with deterministic lifecycle control.

It uses:
- CID-versioned canonical core snapshots (DAG-CBOR)
- cryptographic update rules (`sequence`, `prev`, and signed core attestations)
- optional Data Integrity proof anchoring for interoperability and zero-knowledge verification workflows

This design keeps the DID stable across key rotation and cryptographic upgrades, including post-quantum migration.

## Why did:me?

As digital identity systems move toward eIDAS 2.0 and EUDI Wallet deployment, implementers need all of the following at once:
- high assurance and auditability
- privacy-preserving verification pathways
- long-term cryptographic agility

`did:me` was designed to balance those requirements in one method architecture, while preserving decentralized verification semantics. The goal is practical deployability: predictable lifecycle behavior for operators and verifiable state integrity for relying parties.

## Decentralization Model

`did:me` does not require a single global permissioned ledger for method validity.

- Validity is derived from the signed canonical core chain.
- Documents may be self-hosted and independently verified.
- Optional directories can mirror DID Documents for discoverability.
- This is a hybrid registry model: optional discovery infrastructure with cryptographic state validation independent of any single registry operator.

## Comparison to Common DID Methods

`did:me` targets deployments requiring stable identifiers, cryptographic agility, and privacy-preserving proof integrations. The table below is a high-level orientation aid; implementers should evaluate each method against its own specification and governance model.

| Feature | did:key | did:web | did:ebsi | did:me |
|---|---|---|---|---|
| Stable Identifier Across Rekey | No | Yes | Yes | Yes |
| Built-in Cryptographic Agility Model | Limited | Manual | Policy-Dependent | Integrated |
| Optional Zero-Knowledge-Compatible Proof Anchor | Limited | Integrator-Defined | Supported | Integrated |
| Offline Verifiability Potential | Limited | Medium | Medium/High | High |
| Resolution Dependency | Key Material | DNS/HTTPS | Network Governance | Signed Core Chain (optional directory mirrors) |
| Domain Verification (DNS TXT / `/.well-known`) | External Pattern | DNS Native | Profile-Dependent | First-Class Method Field |

## Ecosystem Capability Matrix

This matrix is an ecosystem-oriented view (method + common deployment pattern), not a normative conformance table.

| Capability | did:key | did:jwk | did:web | did:ebsi | did:cheqd | did:ion | did:me |
|---|---|---|---|---|---|---|---|
| Individuals: can mint their own DIDs | Yes | Yes | Only if domain owned | Policy-dependent | Yes (fees/governance apply) | Yes (anchored flow) | Yes |
| Individuals: can issue credentials | Yes | Yes | Yes | Profile/governance dependent | Yes | Yes | Yes |
| Individuals: can rotate keys | No (identifier changes) | No (identifier bound to key material) | Yes | Yes | Yes | Yes | Yes |
| Individuals: can use PQ key suites | Limited/extension-dependent | Limited/extension-dependent | Deployment-dependent | Policy-dependent | Deployment-dependent | Deployment-dependent | Native support |
| Groups: can mint group-controlled DIDs | Limited | Limited | Yes (domain-governed) | Policy/governance dependent | Yes | Yes | Yes |
| Groups: can issue credentials | Yes | Yes | Yes | Yes (approved roles) | Yes | Yes | Yes |
| Groups: can rotate keys | No (identifier changes) | No (identifier bound to key material) | Yes | Yes | Yes | Yes | Yes |
| Groups: can use PQ key suites | Limited/extension-dependent | Limited/extension-dependent | Deployment-dependent | Policy-dependent | Deployment-dependent | Deployment-dependent | Native support |
| Membership Orgs: can mint org DIDs | Limited | Limited | Yes | Yes (governed) | Yes | Yes | Yes |
| Membership Orgs: can issue credentials | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| Membership Orgs: can rotate keys | No (identifier changes) | No (identifier bound to key material) | Yes | Yes | Yes | Yes | Yes |
| Membership Orgs: can use PQ key suites | Limited/extension-dependent | Limited/extension-dependent | Deployment-dependent | Policy-dependent | Deployment-dependent | Deployment-dependent | Native support |
| Entities (institutions): can mint DIDs | Limited | Limited | Yes | Yes | Yes | Yes | Yes |
| Entities (institutions): can issue credentials | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| Entities (institutions): can rotate keys | No (identifier changes) | No (identifier bound to key material) | Yes | Yes | Yes | Yes | Yes |
| Entities (institutions): can use PQ key suites | Limited/extension-dependent | Limited/extension-dependent | Deployment-dependent | Policy-dependent | Deployment-dependent | Deployment-dependent | Native support |
| Infrastructure Dependency | None (key material only) | None (key material only) | DNS + HTTPS hosting | Permissioned network/governance | Blockchain network + fees | Bitcoin anchoring infrastructure | Signed core chain; optional directory mirrors |

## Key Technical Pillars

- **Canonical Core Integrity**: authoritative state is signed canonical DAG-CBOR, addressed by CID.
- **Cryptographic Agility**: supports classical and post-quantum suites in the same method.
- **Privacy-Preserving Proof Compatibility**: optional ES256 Data Integrity proof over `currentCore` for interoperable verifier and circuit pipelines.
- **Sole-Control Evolution Proof**: the monotonic update chain (`sequence`, `prev`) provides cryptographic evidence that each state transition was authorized by the controlling key set at the previous state.
- **Auditability**: monotonic update chain (`sequence`, `prev`, `currentCore`, `keyHistory`) enables deterministic state reconstruction.

## How did:me Supports ZK and Merkle-Style Workflows

`did:me` keeps DID state authoritative on the signed canonical core. The optional `proof` object (cryptosuite `es256-jws-cid-2025`) signs only the `currentCore` CID string.

This enables:
- **Zero-knowledge-compatible anchoring**: verifiers and circuits can operate on a compact committed value (`currentCore`) rather than full-document canonicalization logic.
- **Merkle-compatible composition**: the CID commitment can be embedded into larger Merkle/batch attestation structures while preserving deterministic linkage to canonical core bytes (`coreCbor`).

## Cryptographic Suite (v1)

### Authoritative Core Rail
- Canonical state: DAG-CBOR core snapshot
- Addressing: CIDv1 (`dag-cbor`, `sha2-256`, base32)
- Core attestations: base64url signatures over canonical core bytes
- Typical signing suites: Ed25519 and ML-DSA-87

### Optional Interoperability Rail
- Data Integrity proof type: `DataIntegrityProof`
- Cryptosuite: `es256-jws-cid-2025` (did:me-defined profile)
- Signature form: compact JWS (`ES256`) over `currentCore` CID payload
- Role: explicitly non-authoritative anchor for interoperability, policy frameworks, and zero-knowledge integrations

### Key Agreement / PQC
- X25519
- ML-KEM-768
- ML-KEM-1024

## Cryptography Profiles (Policy Guidance)

`did:me` supports multiple cryptographic operating modes. The method architecture supports both classical and post-quantum stacks; the **recommended production posture today is hybrid**.

For root/update control and authoritative core attestations, the preferred hybrid pair is:
- ML-DSA-87
- Ed25519

This gives immediate operational interoperability plus post-quantum resilience during the transition period.

| Profile | Root / Attestation Keys | PQ Required | Intended Use |
|---|---|---|---|
| Default Interop Profile (spec Section 14) | ML-DSA-87 required; Ed25519 also present in the preferred stack | Yes | Public high-assurance ecosystems (recommended) |
| Preferred Hybrid Deployment | ML-DSA-87 + Ed25519 (dual rail) | Yes | Production deployments seeking strongest current balance |
| Minimal Classical Deployment | Ed25519-only (or equivalent classical-only policy) | No | Constrained or transitional environments (not preferred long-term) |

In short: post-quantum support is native and encouraged, but `did:me` can operate without PQ keys when deployment constraints require a classical-only policy.

For groups and membership organizations, multi-controller DID documents can be combined with policy-level controls (for example multi-signature or threshold authorization patterns) in resolver and governance layers.

## Positioning

`did:me` is intended for teams that need both standards alignment and operational clarity:
- stable identifiers that do not change when keys rotate
- deterministic, auditable state transitions
- support for hybrid classical + post-quantum cryptography during migration periods
- compatibility pathways for SD-JWT and zero-knowledge-oriented verification systems

In short: `did:me` emphasizes long-term identifier continuity, cryptographic agility, and verifiable lifecycle control without requiring a single global registry dependency.

## Security Considerations

Security-critical behavior is defined normatively in the method specification (`spec/v1/spec.md`, Section 11). At a high level:
- authoritative DID state is the signed canonical DAG-CBOR core chain
- any JSON-LD DID Document representation MUST be derived from the authoritative canonical core bytes
- rollback/replay/fork protections are enforced through `sequence` and `prev`
- `proof` (ES256 `es256-jws-cid-2025`) is explicitly non-authoritative metadata
- key custody, rotation policy, and backup controls remain deployment responsibilities

Detecting controller equivocation across independent verifiers is a deployment-layer responsibility and may be supported through optional witnessing, transparency logs, or directory-based monitoring.

## Credential Compatibility

`did:me` is designed for broad compatibility with OpenID4VCI and OpenID4VP credential ecosystems, including SD-JWT-based deployments and zero-knowledge-oriented verification workflows.

Compatibility should be interpreted at the profile and implementation level:
- method-level DID semantics are defined by `did:me`
- credential format conformance is defined by the credential/profile stack in use
- issuer/verifier policy determines final acceptance rules in production systems

## Trust Framework Alignment

`did:me` can operate within external trust frameworks (including regulated and sector-specific frameworks). Method conformance and trust-framework conformance are distinct:
- the DID method defines identifier lifecycle and cryptographic state validation
- trust frameworks define governance, legal responsibilities, policy controls, and assurance levels

## Identifier Scope and Publication Model

`did:me` is not designed as a globally broadcast, blockchain-style identity handle. In most real-world deployments—particularly in eIDAS and wallet-based ecosystems—individuals will generate distinct DIDs for different issuers, relying parties, or interaction contexts. These DIDs are often pairwise and need not be globally published. The authoritative state of a `did:me` identifier is derived from its signed canonical core chain, not from global consensus or public ledger anchoring. This model prioritizes privacy, compartmentalization, and lifecycle integrity over universal visibility. Global publication is optional and use-case dependent; decentralized verification is achieved through cryptographic state validation rather than network-wide state replication.

Optional public directories may mirror DID Documents for discoverability. Owners of DIDs may choose to publish to such directories based on their interoperability, policy, and ecosystem requirements.

## Specification

- Full specification: [spec/v1/spec.md](spec/v1/spec.md)
- JSON-LD context: [context/v1/context.jsonld](context/v1/context.jsonld)

## Related Resources

- DID Core (W3C): https://www.w3.org/TR/did-core/
- DID Specification Registries: https://www.w3.org/TR/did-spec-registries/
- Decentralized Identifiers Community Group: https://www.w3.org/community/did/

## License & Trademarks

- [License](LICENSE.md)
- [Trademarks](TRADEMARKS.md)
