# DID Document

## Structure

A `did:pki` DID Document represents a Certificate Authority within a national PKI hierarchy. It contains the CA's public keys, its position in the trust chain, certificate metadata, and verification material.

## Example: Costa Rica — CA SINPE Persona Física

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/security/suites/jws-2020/v1",
    "https://spec.attestto.com/v1/pki.jsonld"
  ],
  "id": "did:pki:cr:sinpe:persona-fisica",
  "controller": "did:pki:cr:politica:persona-fisica",
  "alsoKnownAs": [
    "urn:oid:2.16.188.1.1.1.1.1",
    "https://fdi.sinpe.fi.cr/repositorio/CA%20SINPE%20-%20PERSONA%20FISICA.crt"
  ],
  "verificationMethod": [
    {
      "id": "did:pki:cr:sinpe:persona-fisica#key-2023",
      "type": "JsonWebKey2020",
      "controller": "did:pki:cr:sinpe:persona-fisica",
      "publicKeyJwk": {
        "kty": "RSA",
        "n": "...",
        "e": "AQAB",
        "alg": "RS256",
        "x5t#S256": "b04839cecfc73d5fb0db8da4d0613d690f604cb7"
      }
    },
    {
      "id": "did:pki:cr:sinpe:persona-fisica#key-2026",
      "type": "JsonWebKey2020",
      "controller": "did:pki:cr:sinpe:persona-fisica",
      "publicKeyJwk": {
        "kty": "RSA",
        "n": "...",
        "e": "AQAB",
        "alg": "RS256",
        "x5t#S256": "d1dfd32ba8977afcc7976e9a36ef5289faddf122"
      }
    }
  ],
  "assertionMethod": [
    "did:pki:cr:sinpe:persona-fisica#key-2023",
    "did:pki:cr:sinpe:persona-fisica#key-2026"
  ],
  "service": [
    {
      "id": "did:pki:cr:sinpe:persona-fisica#crl",
      "type": "CertificateRevocationList",
      "serviceEndpoint": "http://fdi.sinpe.fi.cr/repositorio/CA%20SINPE%20-%20PERSONA%20FISICA%20v2(2).crl"
    },
    {
      "id": "did:pki:cr:sinpe:persona-fisica#ocsp",
      "type": "OCSPResponder",
      "serviceEndpoint": "http://ocsp.sinpe.fi.cr/"
    },
    {
      "id": "did:pki:cr:sinpe:persona-fisica#repository",
      "type": "CertificateRepository",
      "serviceEndpoint": "https://www.bccr.fi.cr/firma-digital/informaci%C3%B3n-general/certificados-y-listas-de-revocaci%C3%B3n"
    }
  ],
  "pkiMetadata": {
    "country": "CR",
    "countryName": "Costa Rica",
    "hierarchy": "Jerarquía Nacional de Certificadores Registrados",
    "administrator": "Banco Central de Costa Rica (BCCR)",
    "supervisor": "Dirección de Gobernanza Digital (MICITT)",
    "level": "issuing",
    "parentDid": "did:pki:cr:politica:persona-fisica",
    "rootDid": "did:pki:cr:raiz-nacional",
    "endEntityHints": {
      "nationalIdField": "serialNumber",
      "nationalIdFormat": "CR-cedula",
      "nationalIdPattern": "^[0-9]{9,12}$",
      "nameField": "CN",
      "emailField": "SAN:rfc822Name",
      "professionalIdField": "OU",
      "documentationUrl": "https://www.bccr.fi.cr/firma-digital/certificados-de-personas-f%C3%ADsicas"
    },
    "generations": [
      {
        "keyId": "#key-2023",
        "notBefore": "2023-01-15T00:00:00Z",
        "notAfter": "2031-01-15T23:59:59Z",
        "serialNumber": "4e00000002738de677fe8e6840000000000002",
        "sha256Fingerprint": "b04839cecfc73d5fb0db8da4d0613d690f604cb7",
        "status": "active"
      },
      {
        "keyId": "#key-2026",
        "notBefore": "2026-02-01T00:00:00Z",
        "notAfter": "2034-02-01T23:59:59Z",
        "serialNumber": "4e00000006e605fb3f791d077f000000000006",
        "sha256Fingerprint": "d1dfd32ba8977afcc7976e9a36ef5289faddf122",
        "status": "active"
      }
    ]
  }
}
```

## Example: Root CA

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/security/suites/jws-2020/v1",
    "https://spec.attestto.com/v1/pki.jsonld"
  ],
  "id": "did:pki:cr:raiz-nacional",
  "controller": "did:pki:cr:raiz-nacional",
  "verificationMethod": [
    {
      "id": "did:pki:cr:raiz-nacional#key-1",
      "type": "JsonWebKey2020",
      "controller": "did:pki:cr:raiz-nacional",
      "publicKeyJwk": {
        "kty": "RSA",
        "n": "...",
        "e": "AQAB",
        "alg": "RS256",
        "x5t#S256": "68a24d2b2cb5c4ce9fe300e3b6fcd1dfeeb9c311"
      }
    }
  ],
  "assertionMethod": [
    "did:pki:cr:raiz-nacional#key-1"
  ],
  "pkiMetadata": {
    "country": "CR",
    "countryName": "Costa Rica",
    "hierarchy": "Jerarquía Nacional de Certificadores Registrados",
    "administrator": "Banco Central de Costa Rica (BCCR)",
    "supervisor": "Dirección de Gobernanza Digital (MICITT)",
    "level": "root",
    "childDids": [
      "did:pki:cr:politica:persona-fisica",
      "did:pki:cr:politica:persona-juridica",
      "did:pki:cr:politica:sellado-de-tiempo"
    ],
    "generations": [
      {
        "keyId": "#key-1",
        "notBefore": "2015-02-24T00:00:00Z",
        "notAfter": "2039-02-24T23:59:59Z",
        "serialNumber": "74b8cf638fab72bc46c1d2bacd00050c",
        "sha256Fingerprint": "68a24d2b2cb5c4ce9fe300e3b6fcd1dfeeb9c311",
        "status": "active"
      }
    ]
  }
}
```

## DID Document Properties

### Standard W3C Properties

| Property | Usage in did:pki |
|----------|-----------------|
| `id` | The `did:pki` identifier |
| `controller` | Parent CA's DID (root CAs are self-controlling) |
| `alsoKnownAs` | OID arc and/or certificate download URL |
| `verificationMethod` | CA public keys in JWK format (one per generation) |
| `assertionMethod` | References to verification methods (CAs assert identity) |
| `service` | CRL distribution points, OCSP responders, repository URLs |

### Extension: pkiMetadata

The `pkiMetadata` property is defined in the `did:pki` JSON-LD context (`spec.attestto.com/v1/pki.jsonld`) and contains:

| Field | Type | Description |
|-------|------|-------------|
| `country` | string | ISO 3166-1 alpha-2 code |
| `countryName` | string | Human-readable country name |
| `hierarchy` | string | Name of the national PKI framework |
| `administrator` | string | Entity operating the CA |
| `supervisor` | string | Regulatory body overseeing the hierarchy |
| `level` | enum | `root`, `policy`, `issuing`, `timestamping` |
| `parentDid` | string | DID of parent CA (null for root) |
| `rootDid` | string | DID of the root CA |
| `childDids` | array | DIDs of child CAs (for root/policy CAs) |
| `generations` | array | Certificate generation metadata (see below) |

### End-Entity Certificate Parsing Hints

When a verifier receives a signed document, the signer's end-entity certificate is embedded in the document. The verifier needs to know how to extract identity information (national ID, professional credentials) from that certificate — but each country uses different fields and formats.

The `endEntityHints` property provides **machine-readable parsing instructions** for end-entity certificates issued by this CA. It contains NO personal data — only field names, formats, and patterns that describe the certificate structure.

| Field | Type | Description |
|-------|------|-------------|
| `nationalIdField` | string | X.509 field containing the signer's national ID (e.g., `serialNumber`, `UID`) |
| `nationalIdFormat` | string | Format identifier (e.g., `CR-cedula`, `BR-cpf`, `ES-dni`, `CO-cc`) |
| `nationalIdPattern` | string | Regex pattern for validation (e.g., `^[0-9]-[0-9]{4}-[0-9]{4}$` for CR cédula) |
| `nameField` | string | Field containing the signer's name (e.g., `CN`, `GN+SN`) |
| `organizationField` | string | Field containing the signer's organization, if applicable |
| `professionalIdField` | string | Field containing professional credentials (e.g., colegiado number in `OU`) |
| `emailField` | string | Field containing email (e.g., `emailAddress`, `SAN:rfc822Name`) |
| `documentationUrl` | string | URL to the CA's certificate policy document |

**Example for CR BCCR Persona Física:**

```json
"endEntityHints": {
  "nationalIdField": "serialNumber",
  "nationalIdFormat": "CR-cedula",
  "nationalIdPattern": "^[0-9]{9,12}$",
  "nameField": "CN",
  "emailField": "SAN:rfc822Name",
  "documentationUrl": "https://www.bccr.fi.cr/firma-digital/certificados-de-personas-f%C3%ADsicas"
}
```

**Example for BR ICP-Brasil Pessoa Física:**

```json
"endEntityHints": {
  "nationalIdField": "OID:2.16.76.1.3.1",
  "nationalIdFormat": "BR-cpf",
  "nationalIdPattern": "^[0-9]{11}$",
  "nameField": "CN",
  "emailField": "SAN:rfc822Name",
  "documentationUrl": "https://www.gov.br/iti/pt-br/assuntos/icp-brasil"
}
```

**Privacy note:** `endEntityHints` describes the *structure* of end-entity certificates, never their *content*. It is a schema definition — equivalent to a database column description, not a database row. No personal data is stored, transmitted, or resolvable through these hints. The actual personal data exists only within the end-entity certificate embedded in the signed document, which is outside the scope of `did:pki`.

### Generation Metadata

Each generation represents a CA certificate with a specific validity period:

| Field | Type | Description |
|-------|------|-------------|
| `keyId` | string | Reference to verificationMethod |
| `notBefore` | datetime | Certificate validity start (ISO 8601) |
| `notAfter` | datetime | Certificate validity end (ISO 8601) |
| `serialNumber` | string | X.509 serial number (hex) |
| `sha256Fingerprint` | string | SHA-256 hash of DER-encoded certificate |
| `status` | enum | `active`, `expired`, `revoked` |
