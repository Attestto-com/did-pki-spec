# Derivation Algorithm

## Overview

The derivation algorithm converts an X.509 CA certificate into a `did:pki` identifier. The algorithm is **deterministic** — the same certificate MUST always produce the same DID. This property is essential for multi-registry operation: independent resolvers that index the same certificate will derive the same DID without coordination.

The algorithm uses BOTH the Organization (O) and Common Name (CN) fields from the certificate's Subject Distinguished Name. The O field determines the first path segment (unless the organization is the country's PKI authority), and the CN provides qualifier segments.

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

3. Determine if O is a country PKI authority:
   - Each country has a list of known PKI authority organizations
     (see Country Authority Table below)
   - If O matches a known authority → OMIT O from the path
   - If O does not match → O becomes the first path segment

4. Process the Organization (O) field (when NOT a country authority):
   a. Check against known abbreviations table
      (e.g., "ICP-Brasil" → "icp", "Federal PKI" → "fpki")
   b. Strip known corporate/legal suffixes:
      "-RCM", " S.A.", " GmbH", " Inc.", " Ltd.", " SAS", " AS", " AG"
   c. Normalize to kebab-case lowercase ASCII (see Normalization below)
   d. This becomes the first ca-path segment

5. Process the Common Name (CN):
   a. Remove version suffixes: " v2", " v10", etc.
   b. Remove generation suffixes: " G2", " G3", etc.
   c. Remove CA/AC prefixes: "CA ", "AC "
   d. Remove country name suffixes: " - COSTA RICA", " de Brasil", etc.
   e. If O was used as first segment (step 4):
      - Remove the O name from CN to avoid duplication
        (match full org name first, then first word)
   f. Split on hierarchy level keywords:
      - If CN starts with a known level keyword (e.g., "POLITICA")
        followed by a space, treat the boundary as a segment separator
   g. Split on known separators: " - ", " / ", " – ", " — "
   h. Normalize each part to kebab-case lowercase ASCII
   i. These become the remaining path segments

6. Determine hierarchy level:
   - If Basic Constraints CA:TRUE and the certificate is
     self-signed → root level
   - If Basic Constraints CA:TRUE and signed by root →
     policy level
   - If Basic Constraints CA:TRUE and signed by policy →
     issuing level

7. Assemble the DID:
   did = "did:pki:" + country-code + ":" + join(ca-path, ":")

8. Optional: append generation suffix
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

5. Split on hierarchy level keywords:
   - If the string starts with a known level keyword followed
     by a space, insert a segment boundary after the keyword.
   - Known keywords: "politica" (CR policy CA naming convention)

6. Replace separator sequences with colon:
   - " - " → ":"
   - " / " → ":"
   - " – " → ":" (en-dash)

7. Replace remaining whitespace with hyphen:
   - " " → "-"

8. Remove consecutive hyphens and colons:
   - "--" → "-"
   - "::" → ":"

9. Trim leading/trailing hyphens and colons
```

## Country Authority Table

When the Organization (O) field matches one of these values (case-insensitive), the O is OMITTED from the DID path and the CN alone determines the path segments.

| Country | Known Authority O Values |
|---------|------------------------|
| CR | "BCCR", "Banco Central de Costa Rica", "MICITT" |
| US | "U.S. Government" |
| BR | "Instituto Nacional de Tecnologia da Informação", "ITI" |

This table is extensible. When adding a new country, identify the national PKI authority organization(s) and add them here.

## Known Abbreviations Table

When the Organization (O) field matches one of these values (case-insensitive), the abbreviated form is used as the first path segment.

| O Field Value | Abbreviated Segment | Country |
|--------------|-------------------|---------|
| "ICP-Brasil" | `icp` | BR |
| "Federal PKI" | `fpki` | US |

## Corporate Suffix Stripping

The following suffixes are stripped from Organization names before normalization:

```
-RCM        (Real Casa de la Moneda, Spain)
 S.A.       (Sociedad Anónima)
 GmbH       (Gesellschaft mit beschränkter Haftung)
 Inc.       (Incorporated)
 Ltd.       (Limited)
 SAS        (Société par Actions Simplifiée)
 AS         (Aksjeselskap, Nordic)
 AG         (Aktiengesellschaft)
```

## Worked Examples

### Costa Rica Root CA

```
Certificate Subject: CN="CA RAIZ NACIONAL - COSTA RICA v2", O="MICITT", C=CR

Step 1: C="CR", O="MICITT", CN="CA RAIZ NACIONAL - COSTA RICA v2"
Step 2: country-code = "cr"
Step 3: O="MICITT" is a known CR authority → OMIT O
Step 5: CN processing:
  - Remove " v2" → "CA RAIZ NACIONAL - COSTA RICA"
  - Remove "CA " prefix → "RAIZ NACIONAL - COSTA RICA"
  - Remove " - COSTA RICA" suffix → "RAIZ NACIONAL"
  - Lowercase → "raiz nacional"
  - Replace space → "raiz-nacional"
Step 7: did:pki:cr:raiz-nacional
```

### Costa Rica Issuing CA (Persona Física)

```
Certificate Subject: CN="CA SINPE - PERSONA FISICA v2", O="BANCO CENTRAL DE COSTA RICA", C=CR

Step 1: C="CR", O="BANCO CENTRAL DE COSTA RICA", CN="CA SINPE - PERSONA FISICA v2"
Step 2: country-code = "cr"
Step 3: O="BANCO CENTRAL DE COSTA RICA" is a known CR authority → OMIT O
Step 5: CN processing:
  - Remove " v2" → "CA SINPE - PERSONA FISICA"
  - Remove "CA " prefix → "SINPE - PERSONA FISICA"
  - Split on " - " → ["SINPE", "PERSONA FISICA"]
  - Normalize → ["sinpe", "persona-fisica"]
Step 7: did:pki:cr:sinpe:persona-fisica
```

### Costa Rica Policy CA (Persona Física)

```
Certificate Subject: CN="CA POLITICA PERSONA FISICA - COSTA RICA v2", O="MICITT", C=CR

Step 1: C="CR", O="MICITT", CN="CA POLITICA PERSONA FISICA - COSTA RICA v2"
Step 2: country-code = "cr"
Step 3: O="MICITT" is a known CR authority → OMIT O
Step 5: CN processing:
  - Remove " v2" → "CA POLITICA PERSONA FISICA - COSTA RICA"
  - Remove "CA " prefix → "POLITICA PERSONA FISICA - COSTA RICA"
  - Remove " - COSTA RICA" suffix → "POLITICA PERSONA FISICA"
  - Level keyword: starts with "POLITICA " → split → "POLITICA - PERSONA FISICA"
  - Split on " - " → ["POLITICA", "PERSONA FISICA"]
  - Normalize → ["politica", "persona-fisica"]
Step 7: did:pki:cr:politica:persona-fisica
```

### Spain FNMT Root

```
Certificate Subject: CN="AC RAIZ FNMT-RCM", O="FNMT-RCM", C=ES

Step 1: C="ES", O="FNMT-RCM", CN="AC RAIZ FNMT-RCM"
Step 2: country-code = "es"
Step 3: O="FNMT-RCM" is NOT a known ES authority → USE O
Step 4: O processing:
  - Strip "-RCM" suffix → "FNMT"
  - Normalize → "fnmt"
  - First segment: "fnmt"
Step 5: CN processing:
  - Remove org "FNMT-RCM" from CN → "AC RAIZ"
  - Remove "AC " prefix → "RAIZ"
  - Normalize → ["raiz"]
Step 7: did:pki:es:fnmt:raiz
```

### Brazil ICP-Brasil Root

```
Certificate Subject: CN="AC Raiz ICP-Brasil v10", O="ICP-Brasil", C=BR

Step 1: C="BR", O="ICP-Brasil", CN="AC Raiz ICP-Brasil v10"
Step 2: country-code = "br"
Step 3: O="ICP-Brasil" is NOT a known BR authority → USE O
Step 4: O processing:
  - Known abbreviation: "ICP-Brasil" → "icp"
  - First segment: "icp"
Step 5: CN processing:
  - Remove " v10" → "AC Raiz ICP-Brasil"
  - Remove org "ICP-Brasil" from CN → "AC Raiz"
  - Remove "AC " prefix → "Raiz"
  - Normalize → ["raiz"]
Step 7: did:pki:br:icp:raiz
```

## Determinism Guarantee

Two independent implementations processing the same X.509 certificate MUST produce the same `did:pki` identifier. To verify conformance, implementers SHOULD test against the reference test vectors published in the `test-vectors/` directory of the specification repository.

The determinism depends on:
1. The normalization rules (transliteration, prefix/suffix removal, separator handling)
2. The Country Authority Table (which O values to omit)
3. The Known Abbreviations Table (which O values to abbreviate)
4. The Corporate Suffix Stripping list

All four tables are published as part of this specification and MUST be used by conformant implementations. New entries MAY be added in future specification versions as new countries are onboarded.

The reference implementation is available at [github.com/Attestto-com/did-pki-resolver](https://github.com/Attestto-com/did-pki-resolver).
