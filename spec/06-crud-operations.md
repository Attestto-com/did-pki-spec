# CRUD Operations

## Overview

`did:pki` is a **read-only** DID method. DIDs are derived deterministically from existing X.509 certificates — they are not created, updated, or deactivated by any party. The DID lifecycle is entirely determined by the underlying certificate lifecycle.

This is a fundamental design choice: `did:pki` bridges existing trust infrastructure into the DID ecosystem without adding governance overhead.

## Create

**Not applicable.** A `did:pki` DID exists if and only if the corresponding X.509 CA certificate exists in a national PKI hierarchy. The DID is derived, not registered.

The derivation algorithm (Section 7) is deterministic: given the same X.509 certificate, any conformant implementation MUST produce the same `did:pki` identifier. No coordination between implementations is required.

A new `did:pki` DID comes into existence when:
1. A national PKI authority issues a new CA certificate; AND
2. A resolver indexes that certificate and makes it available for resolution.

The first condition is controlled by national PKI authorities. The second is controlled by resolver operators. Neither requires action by the DID subject.

## Read (Resolve)

Resolution is the primary operation. Given a `did:pki` identifier, a resolver returns the corresponding DID Document.

### Resolution Algorithm

```
Input: did:pki DID string
Output: DID Resolution Result (DID Document + metadata) or error

1. Parse the DID string:
   - Extract country-code (2 chars after "did:pki:")
   - Extract ca-path (remaining colon-separated segments)
   - Extract optional generation suffix (last segment if numeric/year)

2. Validate country-code against ISO 3166-1 alpha-2:
   - If invalid → return error (invalidDid)

3. Look up the CA in the trust registry:
   - Key = (country-code, ca-path)
   - If not found → return error (notFound)

4. If generation suffix specified:
   - Find the certificate generation matching the year
   - If not found → return error (notFound)
   - Return DID Document for that specific generation

5. If no generation suffix:
   - Find the currently active generation(s)
   - If all generations expired/revoked → return deactivated DID Document

6. Construct the DID Document:
   - Set id = the input DID
   - Set controller = parent CA's DID (or self for root)
   - For each active generation:
     - Add verificationMethod with JWK-encoded public key
     - Add assertionMethod reference
   - Add service endpoints (CRL, OCSP, repository)
   - Add pkiMetadata

7. Construct didDocumentMetadata:
   - Set created = earliest notBefore of any generation
   - Set updated = most recent certificate issuance date
   - Set deactivated = true if all generations expired/revoked
   - Set nextUpdate = nearest notAfter (hint for cache refresh)
   - Set versionId = SHA-256 of concatenated certificate fingerprints

8. Return DID Resolution Result
```

### Resolution Inputs

| Parameter | Required | Description |
|-----------|----------|-------------|
| `did` | Yes | The `did:pki` identifier to resolve |
| `accept` | No | Preferred DID Document representation (default: `application/did+json`) |
| `versionId` | No | Specific version (generation) to resolve |
| `versionTime` | No | Resolve the state at a given timestamp |

### Error Codes

| Error | Condition |
|-------|-----------|
| `invalidDid` | DID string does not conform to did:pki syntax |
| `invalidDidUrl` | DID URL parsing failed |
| `notFound` | No CA certificate matches the path |
| `representationNotSupported` | Requested representation not available |
| `internalError` | Resolver error (registry unavailable, etc.) |

## Update

**Not applicable.** The DID Document changes only when:

1. **New generation issued:** The national PKI authority issues a new CA certificate with the same identity but new keys/validity. The resolver detects this and adds the new generation to the DID Document.
2. **Certificate revoked:** The CA certificate appears on the parent CA's CRL or returns "revoked" via OCSP. The resolver marks the generation as revoked.
3. **Certificate expired:** The `notAfter` date has passed. The resolver marks the generation as expired.

All updates are triggered by changes in the national PKI — never by the DID subject or resolver operator.

## Deactivate

**Not applicable.** A `did:pki` DID is deactivated when ALL generations of the corresponding CA certificate are either expired or revoked. The DID Document is still resolvable but includes `"deactivated": true` in the metadata.

A deactivated DID is never deleted — it remains resolvable for historical verification purposes. A signature made while the certificate was active should still be verifiable after the certificate expires, provided the verifier checks `versionTime` against the signature timestamp.
