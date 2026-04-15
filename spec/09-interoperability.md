# Interoperability

## Standards Alignment

| Layer | Standard | did:pki Integration |
|---|---|---|
| **DID Resolution** | W3C DID Core v1.1, DID Resolution v0.3 | Universal Resolver driver |
| **Key Representation** | IETF RFC 7517 (JWK) | CA public keys in JWK format |
| **Certificate Format** | ITU-T X.509 v3 | Source material; fingerprints in DID Document |
| **Signature Formats** | ETSI PAdES, CAdES, XAdES, JAdES | did:pki enables chain validation for all four |
| **Trust Lists** | ETSI TS 119 612 (EU Trusted Lists) | EU LOTL indexable as did:pki DIDs |
| **Revocation** | RFC 5280 (CRL), RFC 6960 (OCSP) | Service endpoints in DID Document |
| **Credential Format** | W3C Verifiable Credentials | CA DID usable as VC issuer verification |
| **Agent Frameworks** | Credo-TS, Aries, Walt.id | Resolver module for did:pki |
| **JSON-LD Context** | W3C JSON-LD 1.1 | Published at spec.attestto.com/v1/pki.jsonld |

## ETSI Advanced Electronic Signature (AdES) Integration

The four ETSI AdES signature formats (PAdES, CAdES, XAdES, JAdES) all require certificate chain validation to a trusted root. Today, this requires the verifier to maintain trust stores per jurisdiction. With `did:pki`, the verifier resolves the issuing CA's DID and obtains the trust anchor dynamically:

```
Traditional:
  Signed PDF → extract cert → look up CA in local trust store → validate chain
  (requires maintaining N country trust stores locally)

With did:pki:
  Signed PDF → extract cert → derive issuing CA's did:pki → resolve → validate chain
  (requires only a DID resolver — trust stores maintained by resolver network)
```

This is particularly valuable for the **B-LT** (long-term) and **B-LTA** (long-term archival) validation levels, where historical trust state must be reconstructable. `did:pki` resolvers that maintain generation history enable this naturally via the `versionTime` resolution parameter.

## EU Trusted Lists (LOTL) Mapping

The EU maintains a List of Trusted Lists (LOTL) under Regulation (EU) No 910/2014 (eIDAS). Each member state publishes a national Trusted List of Qualified Trust Service Providers (QTSPs). These map directly to `did:pki`:

```
EU LOTL → per-country Trusted Lists → QTSPs → CA certificates → did:pki DIDs

did:pki:eu:fr:docusign        → DocuSign France SAS
did:pki:eu:de:d-trust         → D-Trust GmbH
did:pki:eu:ee:sk-id           → SK ID Solutions AS
did:pki:eu:es:fnmt            → FNMT-RCM (Spain)
did:pki:eu:it:infocert        → InfoCert S.p.A.
```

The `eu` pseudo-country code groups QTSPs under the eIDAS framework, with the member state as the first path segment.

## eIDAS 2.0 and EU Digital Identity Wallets

eIDAS 2.0 (Regulation (EU) 2024/1183) mandates EU Digital Identity Wallets (EUDIW) based on the Architecture and Reference Framework (ARF). EUDIWs will issue and verify electronic attestations of attributes — functionally equivalent to W3C Verifiable Credentials.

`did:pki` enables interoperability between eIDAS QTSPs and the broader DID/VC ecosystem:

1. A QTSP's CA certificate → `did:pki` DID
2. The QTSP issues a qualified electronic signature
3. A non-EU verifier resolves the QTSP's `did:pki` DID
4. Chain validation proceeds using the JWK from the DID Document
5. The verifier can assess the QTSP's qualification level from `pkiMetadata`

## GLEIF vLEI Alignment

The Global Legal Entity Identifier Foundation (GLEIF) issues verifiable Legal Entity Identifiers (vLEIs) through Qualified vLEI Issuers (QVIs). QVIs operate under GLEIF's trust framework but issue W3C Verifiable Credentials using the KERI/ACDC profile.

`did:pki` complements the vLEI ecosystem by bridging the gap between:
- **vLEI credentials** (W3C VC / KERI native) — for legal entity identity
- **National PKI signatures** (X.509 / PKCS#7) — for document signing by natural persons within those entities

A company verified by vLEI can have its employees' document signatures verified via `did:pki` — providing end-to-end trust from entity identity (vLEI) through individual signer identity (national PKI → did:pki).

## ISO 18013-5 (mDL) Convergence

ISO 18013-5 (mobile driving license) defines its own trust infrastructure with Issuing Authority CAs. These CAs can be represented as `did:pki` DIDs, enabling:

- Cross-border mDL verification via DID resolution
- Unified trust infrastructure for both document signatures and mobile credentials
- Alignment with the convergence between mDL and W3C VC ecosystems

## DSS (Digital Signature Service) Integration

The European Commission's DSS library (`github.com/esig/dss`) is the reference implementation for ETSI AdES signature validation. DSS uses pluggable `TrustListSource` implementations for certificate chain validation.

A `did:pki`-based `TrustListSource` for DSS would:

1. Receive a certificate chain from a signed document;
2. Derive the issuing CA's `did:pki` DID;
3. Resolve the DID to obtain the CA's public key;
4. Return the trust anchor to DSS for chain validation.

This eliminates the need to maintain local trust stores for each jurisdiction — DSS delegates trust resolution to the `did:pki` network.

## Latin American PKI Coverage

`did:pki` is particularly valuable for Latin American PKIs that are not included in global browser trust stores or EU Trusted Lists:

| Country | PKI Authority | Root CA | did:pki |
|---------|--------------|---------|---------|
| Costa Rica | BCCR (SINPE) | CA RAIZ NACIONAL | `did:pki:cr:raiz-nacional` |
| Brazil | ITI (ICP-Brasil) | AC Raiz ICP-Brasil | `did:pki:br:icp:raiz` |
| Colombia | ONAC / Certicámara | AC Raíz | `did:pki:co:certicamara:raiz` |
| Mexico | SAT | AC del SAT | `did:pki:mx:sat:raiz` |
| Chile | Ministerio de Economía | multiple CAs | `did:pki:cl:{ca-name}` |
| Peru | INDECOPI | ECEP | `did:pki:pe:indecopi:ecep` |
| Ecuador | ARCOTEL / BCE | multiple CAs | `did:pki:ec:{ca-name}` |
| Uruguay | AGESIC | AC Raíz Nacional | `did:pki:uy:agesic:raiz` |
| Argentina | AC Raíz de la Rep. Argentina | ONTI | `did:pki:ar:onti:raiz` |
| Panama | Dirección de Firma Electrónica | — | `did:pki:pa:{ca-name}` |

Each country's trust store can be contributed to any `did:pki` resolver as an open-source community contribution.

## Universal Resolver

The `did:pki` Universal Resolver driver resolves any `did:pki` DID by querying configured resolver endpoints. Any party can operate a resolver — the driver is not tied to any specific provider.

Configuration:
```json
{
  "driver": "did-pki",
  "resolvers": [
    "https://pki-resolver.example.com/",
    "https://another-resolver.example.org/"
  ],
  "fallbackToDirectCRL": true
}
```
