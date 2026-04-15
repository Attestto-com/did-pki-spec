# Security Considerations

## Resolver Integrity

A `did:pki` resolver is a mirror of national PKI publications — it does not create trust, it reflects it. However, a compromised resolver could serve incorrect DID Documents (e.g., substituting a CA's public key). Mitigations:

### Certificate Fingerprint Verification

Every DID Document includes the `fingerprint` of the original X.509 certificate in `pkiMetadata.generations[].fingerprint`, along with a `fingerprintAlgorithm` field indicating the hash algorithm used (SHA-256 recommended; SHA-1 acceptable when the national PKI publishes only SHA-1 thumbprints). A verifier can independently:

1. Download the CA certificate from the national PKI's official repository;
2. Compute the hash using the algorithm specified in `fingerprintAlgorithm`;
3. Compare with the fingerprint in the DID Document.

If they match, the resolver's representation is faithful. This check is RECOMMENDED for high-assurance scenarios (financial transactions, legal documents, government processes).

### Multi-Resolver Verification

For critical operations, verifiers SHOULD resolve the same DID against multiple independent resolvers and confirm that the returned DID Documents are identical. Divergence indicates compromise or staleness in at least one resolver.

### Signed DID Documents

Resolvers MAY sign the DID Documents they serve using their own key. This allows verifiers to assess the resolver's identity but does NOT vouch for the underlying certificate — the fingerprint check is the authoritative proof.

## Staleness and Revocation

### Certificate Revocation

When a CA certificate is revoked:
1. The national PKI authority publishes the revocation via CRL or OCSP;
2. Resolvers detect the revocation through periodic CRL fetching or OCSP queries;
3. The corresponding generation in the DID Document is marked `"status": "revoked"`;
4. If all generations are revoked, the DID Document metadata includes `"deactivated": true`.

**Revocation propagation delay:** There is an inherent delay between when a national PKI authority revokes a certificate and when all `did:pki` resolvers reflect the revocation. The `didDocumentMetadata.nextUpdate` field provides a hint for cache refresh frequency. For real-time assurance, verifiers SHOULD perform direct OCSP checks using the `service` endpoints in the DID Document.

### Stale Resolver Data

A resolver that stops updating its mirror will serve stale DID Documents. Verifiers can detect staleness by:

1. Checking `didDocumentMetadata.updated` — if significantly old, the data may be stale;
2. Comparing with the `nextUpdate` hint;
3. Verifying the CA certificate's current status directly via CRL/OCSP.

## No Private Key Exposure

`did:pki` DID Documents contain ONLY public keys extracted from CA certificates. No private key material is ever stored, transmitted, or referenced. The security of the underlying PKI private keys is entirely the responsibility of the national PKI authority.

## End-Entity Certificate Exclusion

`did:pki` deliberately excludes end-entity certificates (individual citizen/employee certificates) from DID resolution. Only Certificate Authorities receive DIDs. This prevents:

1. **PII exposure:** End-entity certificates contain personal information (name, ID number) that must not be resolvable via a global protocol;
2. **Scale issues:** Millions of end-entity certificates would make the registry impractical;
3. **Scope creep:** End-entity identity is better served by purpose-built DID methods (e.g., `did:key`, `did:web`, `did:sns`).

A verifier who holds a signed document can:
1. Extract the signer's end-entity certificate from the document;
2. Identify the issuing CA from the certificate's Issuer field;
3. Resolve the issuing CA's `did:pki` DID;
4. Validate the certificate chain from end-entity to CA using the CA's public key from the DID Document.

This is the standard X.509 chain validation — `did:pki` simply makes the CA's public key universally resolvable.

## DNS and Network Security

Resolver discovery via DNS TXT records is subject to DNS spoofing. Verifiers SHOULD:

1. Use DNSSEC-validated responses when available;
2. Pin known resolver endpoints for critical operations;
3. Use HTTPS for all resolver communication;
4. Verify DID Document integrity via certificate fingerprints regardless of transport security.

## Governance Model

`did:pki` has no governance authority. The specification defines a protocol; implementations are independent. This means:

1. **No single point of failure:** The compromise of one resolver does not affect others;
2. **No censorship vector:** Any party can operate a resolver;
3. **No update authority:** The specification is versioned; backward-incompatible changes require a new version;
4. **National sovereignty preserved:** Each country's PKI authority retains full control over its certificates. `did:pki` does not modify, govern, or compete with national PKI operations.
