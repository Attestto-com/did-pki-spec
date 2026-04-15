# Privacy Considerations

## No Personal Data

`did:pki` resolves Certificate Authority identities only — never end-entity (individual) certificates. CA certificates contain organizational identity (country, organization name, organizational unit) but no personal data.

End-entity certificates, which may contain personal information (name, national ID number, email), are explicitly excluded from `did:pki` resolution (see Security Considerations, Section 8).

## Resolution Privacy

Resolving a `did:pki` DID does not reveal any information about the document being verified or the individual who signed it. The resolver sees only:

- Which country's PKI hierarchy is being queried
- Which CA within that hierarchy is being queried

This is equivalent to the privacy exposure of fetching a CA certificate from a public repository — which is the current practice.

## No Tracking of Verification Activity

`did:pki` resolvers SHOULD NOT log resolution requests in a way that correlates them to specific documents or signers. Resolution logs, if maintained for operational purposes, MUST be subject to data minimization principles.

Verifiers concerned about resolver-side tracking can:

1. Cache DID Documents locally (they change infrequently — typically only at certificate rollover);
2. Operate their own resolver by mirroring national PKI publications;
3. Use the offline resolution mode with bundled trust stores.

## GDPR and Data Protection Compliance

`did:pki` DID Documents contain only publicly available information from national PKI publications. No consent is required because:

1. CA certificates are public by design — they must be publicly distributable for the PKI to function;
2. No personal data is included in `did:pki` DID Documents;
3. The `pkiMetadata` contains only organizational and jurisdictional information.

For jurisdictions with data protection frameworks (EU GDPR, Costa Rica Ley 8968, Brazil LGPD), `did:pki` introduces no new data processing obligations beyond what already applies to public certificate repositories.
