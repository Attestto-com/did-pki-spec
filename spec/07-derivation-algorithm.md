# Derivation Algorithm

## Overview

The derivation algorithm converts an X.509 CA certificate into a `did:pki` identifier. The algorithm is **deterministic** — the same certificate MUST always produce the same DID. This property is essential for multi-registry operation: independent resolvers that index the same certificate will derive the same DID without coordination.

## Algorithm

```
Input: X.509 CA certificate (DER or PEM encoded)
Output: did:pki identifier string

1. Extract the Subject Distinguished Name (DN):
   - Parse the certificate's Subject field
   - Extract C (Country), O (Organization), CN (Common Name)

2. Determine country-code:
   - Use the C field value
   - Convert to lowercase
   - Validate against ISO 3166-1 alpha-2
   - If C field is absent or invalid, check the certificate's
     issuer hierarchy for a country code
   - If no country can be determined → reject (cannot derive DID)

3. Determine ca-path segments:
   a. Start with the Organization (O) field:
      - Normalize to kebab-case (see Normalization below)
      - This becomes the first path segment (or omitted if
        the organization IS the country PKI authority)
   b. Process the Common Name (CN):
      - Remove the country name suffix if present
        (e.g., " - COSTA RICA" → removed)
      - Remove "CA " prefix if present
      - Split on known separators: " - ", " / ", " – "
      - Normalize each part to kebab-case
      - These become path segments

4. Determine hierarchy level:
   - If Basic Constraints CA:TRUE and the certificate is
     self-signed → root level (single segment)
   - If Basic Constraints CA:TRUE and signed by root →
     policy level (two segments)
   - If Basic Constraints CA:TRUE and signed by policy →
     issuing level (path follows hierarchy)

5. Assemble the DID:
   did = "did:pki:" + country-code + ":" + join(ca-path, ":")

6. Optional: append generation suffix
   - If the same CA identity has multiple certificate generations
     with overlapping validity, append ":" + year(notBefore)
   - Generation suffix is ONLY added when disambiguation is needed
```

## Normalization Rules

The normalization function converts a certificate field value to a `ca-segment`:

```
Input: UTF-8 string (e.g., "CA SINPE - PERSONA FÍSICA")
Output: ASCII kebab-case string (e.g., "sinpe:persona-fisica")

1. Transliterate to ASCII:
   - ñ → n, ü → u, ç → c, á → a, é → e, í → i, ó → o, ú → u
   - ß → ss, ø → o, å → a, æ → ae, ö → o, ä → a
   - Any remaining non-ASCII character → removed

2. Convert to lowercase

3. Remove known prefixes:
   - "ca " (Certificate Authority prefix)
   - "ac " (Autoridad Certificadora prefix, Spanish/Portuguese)

4. Remove known suffixes:
   - " - " + country name (e.g., " - costa rica", " - brasil")
   - " de costa rica", " of costa rica", etc.

5. Replace separator sequences with colon:
   - " - " → ":"
   - " / " → ":"
   - " – " → ":" (en-dash)

6. Replace remaining whitespace with hyphen:
   - " " → "-"

7. Remove consecutive hyphens and colons:
   - "--" → "-"
   - "::" → ":"

8. Trim leading/trailing hyphens and colons
```

## Worked Examples

### Costa Rica Root CA

```
Certificate Subject: CN="CA RAIZ NACIONAL - COSTA RICA", C=CR

Step 1: C="CR", CN="CA RAIZ NACIONAL - COSTA RICA"
Step 2: country-code = "cr"
Step 3:
  - CN = "CA RAIZ NACIONAL - COSTA RICA"
  - Remove "CA " prefix → "RAIZ NACIONAL - COSTA RICA"
  - Remove " - COSTA RICA" suffix → "RAIZ NACIONAL"
  - Lowercase → "raiz nacional"
  - Replace space → "raiz-nacional"
Step 4: Self-signed + CA:TRUE → root
Step 5: did:pki:cr:raiz-nacional
```

### Costa Rica Issuing CA (Persona Física)

```
Certificate Subject: CN="CA SINPE - PERSONA FISICA", O="BCCR", C=CR
Issuer: CN="CA POLITICA PERSONA FISICA - COSTA RICA"

Step 1: C="CR", O="BCCR", CN="CA SINPE - PERSONA FISICA"
Step 2: country-code = "cr"
Step 3:
  - CN = "CA SINPE - PERSONA FISICA"
  - Remove "CA " prefix → "SINPE - PERSONA FISICA"
  - Split on " - " → ["SINPE", "PERSONA FISICA"]
  - Normalize → ["sinpe", "persona-fisica"]
Step 4: CA:TRUE, signed by policy CA → issuing
Step 5: did:pki:cr:sinpe:persona-fisica
```

### Spain FNMT Root

```
Certificate Subject: CN="AC RAIZ FNMT-RCM", O="FNMT-RCM", C=ES

Step 1: C="ES", CN="AC RAIZ FNMT-RCM"
Step 2: country-code = "es"
Step 3:
  - CN = "AC RAIZ FNMT-RCM"
  - Remove "AC " prefix (Spanish CA prefix) → "RAIZ FNMT-RCM"
  - No country suffix
  - No separator split needed
  - Normalize → "fnmt:raiz" (Organization used as primary, CN as qualifier)
Step 5: did:pki:es:fnmt:raiz
```

## Determinism Guarantee

Two independent implementations processing the same X.509 certificate MUST produce the same `did:pki` identifier. To verify conformance, implementers SHOULD test against the reference test vectors published at `spec.attestto.com/did-pki/test-vectors/`.

The test vector set includes certificates from at least 10 countries with varied Subject DN formats, character sets, and hierarchy depths.
