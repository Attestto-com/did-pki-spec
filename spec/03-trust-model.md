# Trust Model

## Overview

`did:pki` does not create trust — it makes existing trust verifiable. The trust originates in national PKI hierarchies, which are governed by each country's legal framework. `did:pki` provides a universal resolution protocol that allows any verifier to navigate these trust hierarchies without country-specific knowledge.

## Trust Anchors

Trust in `did:pki` is anchored in two independent sources:

### 1. National PKI Publications (Primary)

Each country's PKI administrator (e.g., BCCR for Costa Rica, FNMT for Spain, ICP-Brasil for Brazil) publishes its CA certificates through official channels — government websites, certificate repositories, and trusted list publications. These publications are the authoritative source of truth.

A `did:pki` resolver mirrors these publications and serves them as DID Documents. The resolver does not vouch for the certificates — it faithfully represents them. Verifiers trust the certificates because they trust the national PKI authority, not because they trust the resolver.

### 2. EU Trusted Lists and Equivalent Registries (Secondary)

For countries that participate in formal trust frameworks (e.g., EU Trusted Lists under eIDAS, GLEIF's QVI registry), the resolver can include trust framework membership as metadata in the DID Document. This enables verifiers to distinguish between:

- A CA that is part of a recognized trust framework (e.g., EU QTSP)
- A CA that is published by a national authority but not part of a cross-border framework
- A CA whose certificates have been revoked or expired

## Trust Hierarchy Mapping

National PKI hierarchies follow a consistent pattern that maps directly to `did:pki` DID relationships:

```
Root CA (self-signed)                    did:pki:cr:raiz-nacional
  ├── Policy CA (Persona Física)         did:pki:cr:politica:persona-fisica
  │   └── Issuing CA (SINPE PF)         did:pki:cr:sinpe:persona-fisica
  │       └── End-entity cert           (not a did:pki DID — personal cert)
  ├── Policy CA (Persona Jurídica)       did:pki:cr:politica:persona-juridica
  │   └── Issuing CA (SINPE PJ)         did:pki:cr:sinpe:persona-juridica
  └── Policy CA (Sellado de Tiempo)      did:pki:cr:politica:sellado-de-tiempo
      └── Issuing CA (TSA)              did:pki:cr:sinpe:sellado-de-tiempo
```

Each CA in the hierarchy is a `did:pki` DID. The `controller` property of each DID Document points to the parent CA's DID, enabling chain traversal:

```json
{
  "id": "did:pki:cr:sinpe:persona-fisica",
  "controller": "did:pki:cr:politica:persona-fisica"
}
```

End-entity certificates (individual citizens, employees) are NOT represented as `did:pki` DIDs. The method resolves CAs only — the entities that anchor trust chains, not the leaf certificates.

## Resolver Trust

A `did:pki` resolver is a service that mirrors national PKI publications and serves them as DID Documents. Multiple resolvers can coexist, and verifiers SHOULD use more than one for critical operations.

### Resolver Integrity Verification

Each DID Document MUST include a `proof` section containing the SHA-256 fingerprint of the original X.509 certificate. This allows a verifier to independently confirm the mapping:

1. Resolve `did:pki:cr:sinpe:persona-fisica` from any resolver
2. Extract the `x509CertificateSha256` from the proof
3. Download the original certificate from BCCR's official repository
4. Compare fingerprints

If the fingerprints match, the resolver's representation is faithful. If they don't, the resolver is compromised or stale. This verification is optional but RECOMMENDED for high-assurance scenarios.

### Resolver Discovery

Resolvers are discovered through:

1. **Universal Resolver:** The W3C Universal Resolver driver for `did:pki` delegates to configured resolver endpoints.
2. **Well-Known URI:** `https://{resolver-domain}/.well-known/did-pki-configuration` returns resolver metadata.
3. **DNS TXT records:** `_did-pki.{resolver-domain} TXT "endpoint=https://..."` enables DNS-based discovery.

## No Central Authority

`did:pki` has no central authority. Any party can operate a resolver by:

1. Downloading CA certificates from official national PKI publications;
2. Deriving `did:pki` DIDs using the deterministic algorithm (Section 7);
3. Serving DID Documents at a public endpoint;
4. Registering as a Universal Resolver driver.

The reference resolver operated by Attestto at `trust.attestto.com` is a convenience — not a requirement. The specification is open, the derivation algorithm is deterministic, and any implementation that follows the spec produces identical results.
