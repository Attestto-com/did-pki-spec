# Test Vectors

Machine-readable test vectors for verifying `did:pki` resolver implementations.

## Format

Each country file (`<cc>.json`) contains:

```json
{
  "specVersion": "0.1.0",
  "country": "CR",
  "vectors": [
    {
      "did": "did:pki:cr:raiz-nacional",
      "description": "root CA: CA RAIZ NACIONAL - COSTA RICA v2",
      "derivation": [
        {
          "input": {
            "subjectCN": "CA RAIZ NACIONAL - COSTA RICA v2",
            "countryCode": "CR",
            "sha256": "fd9ea730..."
          },
          "normalizationResult": ["raiz-nacional"],
          "derivedDid": "did:pki:cr:raiz-nacional"
        }
      ],
      "expectedDidDocument": { ... },
      "expectedDidDocumentMetadata": { ... }
    }
  ]
}
```

## Available Countries

| File | Country | Vectors | Hierarchy Depth |
|------|---------|---------|-----------------|
| `cr.json` | Costa Rica | 5 | root → policy → issuing |

## How to Use

1. For each vector, verify that your derivation algorithm produces the same DID from the input `subjectCN` + `countryCode`
2. Verify that your resolver produces a DID Document matching `expectedDidDocument` (public key values may differ if extracted from the PEM differently — compare fingerprints)
3. Verify that metadata matches `expectedDidDocumentMetadata`

## Certificate Source

Test vectors are generated from certificates in [`@attestto/trust`](https://github.com/Attestto-com/attestto-trust), an independent public mirror of national PKI root and intermediate certificates.

## Adding Countries

To add test vectors for a new country:

1. Add the country's certificates to `attestto-trust/countries/<cc>/current/`
2. Add a `CountryConfig` to the resolver's `countries.ts`
3. Generate vectors using the resolver
