# DID Syntax

## Method Name

The method name is `pki`.

## Method-Specific Identifier

```abnf
did-pki           = "did:pki:" country-code ":" ca-path
country-code      = 2ALPHA                          ; ISO 3166-1 alpha-2 (lowercase)
ca-path           = ca-segment *( ":" ca-segment )
ca-segment        = 1*( ALPHA / DIGIT / "-" )       ; kebab-case, lowercase
```

## Construction Rules

1. `country-code` MUST be a valid ISO 3166-1 alpha-2 code, lowercased (e.g., `cr`, `es`, `br`, `de`, `us`).
2. `ca-path` encodes the position of the CA within the national hierarchy, from root to leaf, using colon-separated segments.
3. Each `ca-segment` is derived from the CA certificate's Subject Common Name (CN) or Organization (O) field, normalized to kebab-case lowercase ASCII.
4. The normalization algorithm (Section 7) is deterministic — the same certificate always produces the same `ca-segment`.
5. Special characters, diacritics, and non-ASCII characters are transliterated to ASCII equivalents (e.g., `ñ` → `n`, `ü` → `u`, `ç` → `c`).
6. For regional groupings (e.g., EU Trusted Lists), the pseudo-country code `eu` MAY be used, with the QTSP country as the first `ca-segment`.

## Examples

### Costa Rica (BCCR hierarchy)

```
did:pki:cr:raiz-nacional
  Root CA: "CA RAIZ NACIONAL - COSTA RICA"

did:pki:cr:politica:persona-fisica
  Policy CA: "CA POLITICA PERSONA FISICA - COSTA RICA"

did:pki:cr:sinpe:persona-fisica
  Issuing CA: "CA SINPE - PERSONA FISICA"

did:pki:cr:sinpe:persona-fisica:2023
  Issuing CA (generation-specific): "CA SINPE - PERSONA FISICA" (Jan 2023 - Jan 2031)

did:pki:cr:sinpe:persona-juridica
  Issuing CA: "CA SINPE - PERSONA JURIDICA"

did:pki:cr:politica:sellado-de-tiempo
  Policy CA: "CA POLITICA SELLADO DE TIEMPO - COSTA RICA"
```

### Spain (FNMT hierarchy)

```
did:pki:es:fnmt:raiz
  Root CA: "AC RAIZ FNMT-RCM"

did:pki:es:fnmt:representacion
  Issuing CA: "AC FNMT Usuarios - Representación"

did:pki:es:fnmt:componentes
  Issuing CA: "AC Componentes Informáticos"
```

### Brazil (ICP-Brasil hierarchy)

```
did:pki:br:icp:raiz-v10
  Root CA: "AC Raiz ICP-Brasil v10"

did:pki:br:icp:serpro:rfb
  Issuing CA: "AC SERPRO RFB v5"
```

### European Union (QTSP via eIDAS Trusted List)

```
did:pki:eu:fr:docusign
  EU QTSP: DocuSign France SAS

did:pki:eu:de:d-trust
  EU QTSP: D-Trust GmbH (Germany)

did:pki:eu:ee:sk-id
  EU QTSP: SK ID Solutions AS (Estonia)
```

### United States (Federal PKI)

```
did:pki:us:fpki:common-policy
  Root CA: "Federal Common Policy CA G2"

did:pki:us:fpki:sha256-medium-assurance
  Issuing CA: "SHA-256 Medium Assurance"
```

## Path Depth

There is no fixed limit on path depth. The path follows the actual hierarchy of the national PKI:

- 1 segment: Root CA (`did:pki:cr:raiz-nacional`)
- 2 segments: Issuing CA (`did:pki:cr:sinpe:persona-fisica`)
- 3+ segments: Deeper hierarchies where they exist (`did:pki:br:icp:serpro:rfb`)

## Generation Disambiguation

When a national PKI reissues a CA certificate with the same name but different validity period (common during key rollovers), the year of the `notBefore` date MAY be appended as the final segment:

```
did:pki:cr:sinpe:persona-fisica          → current/active generation
did:pki:cr:sinpe:persona-fisica:2023     → specific generation (2023-2031)
did:pki:cr:sinpe:persona-fisica:2026     → newer generation (2026-2034)
```

When the generation suffix is omitted, the resolver MUST return the currently active generation. When specified, the resolver MUST return the exact generation requested, or an error if not found.
