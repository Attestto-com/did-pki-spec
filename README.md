# did:pki Method Specification

**Bridging National PKI Hierarchies to the Decentralized Identity Ecosystem**

---

| | |
|---|---|
| **Specification Version** | v0.1.0 |
| **Status** | Draft |
| **Method Name** | `pki` |
| **Latest Editor's Draft** | [spec.attestto.com/did-pki](https://spec.attestto.com/did-pki) |
| **Conforms To** | [W3C DID Core v1.1](https://www.w3.org/TR/did-core/) |
| **Editor** | [Attestto](https://attestto.org) |
| **Author** | Eduardo Chongkan ([eduardo@attestto.com](mailto:eduardo@attestto.com)) |
| **Repository** | [github.com/Attestto-com/did-extensions](https://github.com/Attestto-com/did-extensions) |
| **License** | Apache 2.0 |
| **Last Updated** | 2026-04-14 |

---

## Abstract

`did:pki` is a read-only DID method that assigns globally resolvable Decentralized Identifiers to Certificate Authorities within national PKI hierarchies. It enables any DID-aware system to verify signatures issued under any national PKI — without country-specific integration, without bundling root certificates, and without manual trust decisions.

Over 100 countries operate sovereign PKI hierarchies. These hierarchies are islands: certificates issued by one country's CA are unverifiable in another without custom tooling. `did:pki` solves this by providing a deterministic mapping from X.509 CA certificates to W3C DIDs, resolvable through the Universal Resolver.

## Examples

```
did:pki:cr:raiz-nacional                → Costa Rica Root CA
did:pki:cr:sinpe:persona-fisica          → Costa Rica BCCR issuing CA (citizens)
did:pki:es:fnmt:raiz                     → Spain FNMT Root CA
did:pki:br:icp:raiz                      → Brazil ICP-Brasil Root CA
did:pki:eu:de:d-trust                    → Germany D-Trust (EU QTSP)
did:pki:us:fpki:common-policy            → US Federal PKI Common Policy CA
did:pki:co:certicamara:raiz              → Colombia Certicámara Root CA
did:pki:mx:sat:raiz                      → Mexico SAT Root CA
```

## Table of Contents

1. [Abstract](spec/01-abstract.md) — Problem statement, solution, design principles
2. [Focal Use Case & Requirements](spec/02-focal-use-case.md) — 6 cross-border scenarios, 10 requirements
3. [Trust Model](spec/03-trust-model.md) — National PKI anchors, EU Trusted Lists, resolver integrity, multi-registry
4. [DID Syntax](spec/04-did-syntax.md) — ABNF grammar, construction rules, examples for CR/ES/BR/EU/US
5. [DID Document](spec/05-did-document.md) — Full JSON examples, properties, pkiMetadata extension
6. [CRUD Operations](spec/06-crud-operations.md) — Read-only method, resolution algorithm, lifecycle
7. [Derivation Algorithm](spec/07-derivation-algorithm.md) — Deterministic X.509 → DID conversion, normalization, worked examples
8. [Security Considerations](spec/08-security-considerations.md) — Resolver integrity, revocation, staleness, governance
9. [Interoperability](spec/09-interoperability.md) — ETSI AdES, EU LOTL, eIDAS 2.0, GLEIF vLEI, ISO 18013-5, DSS, LATAM coverage
10. [Privacy Considerations](spec/10-privacy-considerations.md) — No PII, resolution privacy, GDPR compliance
11. [References](spec/11-references.md) — W3C, IETF, ETSI, eIDAS, national PKI authorities

## Key Properties

| Property | Value |
|----------|-------|
| **Read-only** | DIDs are derived from existing certificates, not registered |
| **Deterministic** | Same certificate → same DID, always, by any implementation |
| **Multi-registry** | Any party can operate a resolver; no central authority |
| **No PII** | Only CA organizational identity; never personal data |
| **Backward compatible** | Adds DID resolution layer; existing X.509 verification unchanged |
| **Bridge, don't replace** | National PKIs continue operating exactly as they do today |

## Related Specifications

- **[did:sns](https://spec.attestto.com/did-sns)** — DID method for Solana Name Service (alias-anchored identity)
- **[@attestto/trust](https://github.com/Attestto-com/attestto-trust)** — Multi-country PKI trust store (reference resolver data source)
- **[@attestto/verify](https://github.com/Attestto-com/attestto-verify)** — Web component for PDF/cert verification (reference verifier)

## Contributing

Country trust stores are the primary contribution opportunity. To add a country:

1. Obtain the country's CA certificates from official government PKI publications
2. Derive `did:pki` identifiers using the derivation algorithm (Section 7)
3. Create DID Documents following the schema (Section 5)
4. Submit a pull request to the `attestto-trust` repository

See [Contributing Guidelines](CONTRIBUTING.md) for details.
