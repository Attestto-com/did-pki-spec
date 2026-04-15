# did:pki — Bridging National PKI Hierarchies to the Decentralized Identity Ecosystem

**Specification Version:** v0.1.0
**Status:** Draft
**Method Name:** `pki`
**DID Method Specification:** [W3C DID Core v1.0](https://www.w3.org/TR/did-core/)
**Last Updated:** 2026-04-14
**Editor:** [Attestto](https://attestto.org)
**Author:** Eduardo Chongkan

---

## Abstract

`did:pki` is a DID method that bridges national Public Key Infrastructure (PKI) hierarchies into the W3C Decentralized Identifier ecosystem. It provides a globally resolvable, standards-based representation of Certificate Authorities (CAs) and their trust chains — enabling any DID-aware system to verify signatures issued under any national PKI without country-specific integration.

Today, over 100 countries operate sovereign PKI hierarchies for digital signatures, identity documents, and government services. These hierarchies are islands: a certificate issued by Costa Rica's BCCR is unverifiable in Panama, Spain's FNMT certificates are opaque to Brazilian systems, and no universal resolution protocol exists. Each cross-border verification requires custom tooling that bundles the target country's root certificates — an N-countries x M-verifiers integration problem that does not scale.

> [!NOTE]
> **`did:pki` solves this by:**
>
> 1. **Assigning a stable DID** to every CA in a national PKI hierarchy, derived deterministically from the certificate's Subject Distinguished Name and country code;
>
> 2. **Resolving to a DID Document** containing the CA's public keys in standard JWK format, its position in the trust chain, and metadata about the PKI hierarchy;
>
> 3. **Enabling programmatic trust anchor resolution** for any signature issued under any national PKI by any W3C-compliant DID resolver — without country-specific code, without bundling root certificates, and without manual trust decisions. The actual signature verification uses standard cryptographic libraries; `did:pki` provides the missing piece: the CA's public key.

The method is **read-only by design**. `did:pki` DIDs are not created, updated, or deactivated by their subjects (CAs). They are derived deterministically from existing X.509 certificates and resolved against a trust registry that mirrors authoritative national PKI publications. The DID lifecycle follows the underlying certificate lifecycle — when a CA certificate expires or is revoked, the corresponding DID Document reflects this state.

### Design Principles

- **Bridge, don't replace.** National PKIs continue to operate exactly as they do today. `did:pki` adds a resolution layer; it does not modify, govern, or compete with existing certificate authorities.
- **Deterministic derivation.** Given the same X.509 certificate, any implementation MUST derive the same `did:pki` identifier. No registration, no coordination, no central authority required for DID creation.
- **Read-only.** CRUD operations are limited to Read. The DID exists because the certificate exists. No subject can "create" or "deactivate" a `did:pki` DID — only the certificate lifecycle controls it.
- **Verifiable provenance.** Every DID Document includes the fingerprint of the original X.509 certificate, enabling verifiers to download the certificate from the national PKI's official repository and independently confirm the mapping.
- **Multi-registry.** Any party can operate a `did:pki` resolver by mirroring national PKI publications. No single registry operator is required or privileged.
