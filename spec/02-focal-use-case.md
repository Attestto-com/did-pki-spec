# Focal Use Case & Requirements

## The Problem: PKI Islands

A Costa Rican notary signs a property transfer document using their BCCR Firma Digital certificate. The buyer is a German company conducting due diligence. The German company's compliance system receives the signed PDF.

**Today:** The compliance system cannot verify the signature. Costa Rica's CA RAÍZ NACIONAL is not in Mozilla's, Google's, or Microsoft's trust stores. The system has no knowledge of BCCR's PKI hierarchy. The German company must either:

1. Manually install Costa Rica's root certificate (security risk, operational burden);
2. Build custom verification tooling that bundles CR's PKI chain (N-country scaling problem);
3. Accept the document on faith (defeats the purpose of digital signatures).

**With did:pki:** The signed PDF includes the signer's certificate, which chains to `CA SINPE - PERSONA FISICA - COSTA RICA`. The compliance system resolves `did:pki:cr:sinpe:persona-fisica` through any Universal Resolver, receives a DID Document containing the CA's public key in JWK format, validates the certificate chain mathematically, and confirms the signature — all without any Costa Rica-specific code.

## Use Cases

### UC-1: Cross-Border Document Verification

A law firm in Panama receives a notarized document from Costa Rica. The document is signed with a BCCR Firma Digital certificate. The firm's document management system resolves the issuing CA's DID, validates the chain to the Costa Rican root, and confirms authenticity — programmatically, in milliseconds.

### UC-2: Foreign Bidder Verification in Public Procurement

A Colombian company bids on a Costa Rican government tender via SICOP. The tender requires digital signatures. The Colombian company's certificate was issued by Certicámara (Colombia's CA). Costa Rica's SICOP system resolves `did:pki:co:certicamara:raiz` to verify the Colombian signature — without requiring the company to obtain a BCCR certificate.

### UC-3: Professional Credential Verification

A Costa Rican engineer (CFIA-certified) applies for a position at a multinational firm in the EU. Their professional certification document is digitally signed using a BCCR Firma Digital certificate. The EU employer's HR system extracts the signer's certificate from the document, resolves the issuing CA's `did:pki` DID, obtains the CA's public key, and validates the certificate chain — no country-specific integration needed.

### UC-4: eIDAS Cross-Recognition

An EU Qualified Trust Service Provider (QTSP) issues a qualified electronic signature. A Costa Rican institution receives a document signed by this QTSP. Instead of establishing a bilateral recognition agreement, the institution resolves `did:pki:eu:fr:docusign` and verifies the chain against the EU Trusted List — through the same DID resolution protocol used for domestic signatures.

### UC-5: Multi-Country Supply Chain

A pharmaceutical company sources materials from Costa Rica, Colombia, and Brazil. Each supplier signs compliance certificates using their national PKI. The company's compliance system resolves `did:pki:cr:...`, `did:pki:co:...`, and `did:pki:br:...` through a single protocol — verifying all three countries' signatures uniformly.

### UC-6: International Legal Cooperation

A Costa Rican court issues a rogatory letter to a Panamanian court, digitally signed by the judge. The Panamanian court's system resolves the Costa Rican judicial CA's DID and verifies the signature cryptographically — replacing the current process of manual authentication via apostille or consular legalization.

## Requirements

| ID | Requirement | Rationale |
|---|---|---|
| R1 | Deterministic derivation from X.509 certificate | Any implementation must derive the same DID from the same cert |
| R2 | Resolution returns standard DID Document | Interoperable with all W3C DID-aware systems |
| R3 | Public keys in JWK format | Universal key representation |
| R4 | Trust chain navigable via DID relationships | Parent CA resolvable via controller/alsoKnownAs |
| R5 | Certificate lifecycle reflected in DID state | Expired/revoked cert → deactivated DID |
| R6 | Multi-registry operation | No single point of failure or control |
| R7 | Offline-capable resolution | Resolver can bundle trust stores for air-gapped verification |
| R8 | No PII in DID or DID Document | Only CA organizational identity, never personal data |
| R9 | Country code follows ISO 3166-1 alpha-2 | Universal, unambiguous country identification |
| R10 | Backward compatible with X.509 verification | did:pki adds a resolution layer; existing PKCS#7/CMS verification unchanged |
