# Troubleshooting Checklist

## Table of Contents

- [Certificate \& TLS](#certificate--tls)
- [Entity Configuration](#entity-configuration)
- [Federation Status](#federation-status)
- [Trust Mark Signature Verification](#trust-mark-signature-verification)
- [URL Format Consistency](#url-format-consistency)
- [Credential Offer Link](#credential-offer-link)
- [SATOSA Logs](#satosa-logs)
- [SimpleSAMLphp Metadata](#simplesamlphp-metadata)

## Certificate & TLS

- Confirm the TLS certificate is valid and properly served:

  ```bash
  echo | openssl s_client -connect <host>:<port>
  ```

  - Look for `Verify return code: 0` (or `19` for self-signed; ensure it is
    trusted by all affected components).
  - Check that the root CA is known and trusted by the wallet.

## Entity Configuration

- Fetch and decode the Entity Configuration:

  ```bash
  curl -s "<Entity Identifier>/.well-known/openid-federation" | \
  cut -d '.' -f2 | tr '_-' '/+' | base64 -d 2>/dev/null | jq .
  ```

  - `sub` must match the Entity Identifier.
  - Ensure metadata sections are present: `jwks`, `authorization_endpoint`,
    `credential_endpoint`, etc.
  - Confirm that valid `trust_marks` are included.

## Federation Status

- Verify the entity is known to the Trust Anchor:

  ```bash
  curl "https://openidfed-test-1.sunet.se:7001/list"
  ```

  - Confirm the entity identifier appears.

- Resolve the trust chain:

  ```bash
  curl "https://openidfed-test-1.sunet.se:7001/resolve?sub=<Entity Identifier>&anchor=https://openidfed-test-1.sunet.se:7001"
  ```

  - Should return a fully resolved trust chain.
  - If empty or broken, verify trust marks or check registration.

## Trust Mark Signature Verification

- Trust Marks are signed by the Trust Mark Issuer, not the Trust Anchor.

Steps to verify:

1. Extract `kid` from JWT header.
2. Retrieve the Trust Mark Issuer’s entity configuration:

   `https://<trust-mark-issuer>/.well-known/openid-federation`

3. Locate matching key in `jwks.keys`.
4. Verify signature (e.g., using [jwt.io](https://jwt.io)).

> Also verify the Trust Mark Issuer’s own trust chain using the `resolve`
> endpoint.

## URL Format Consistency

- Ensure identifiers are consistent:
  - Do not include trailing slashes in `Entity Identifier`
  - Credential offer links
  - Redirect URIs

## Credential Offer Link

- Inspect `credential_offer` for:
  - Correct `credential_issuer`
  - Valid `credential_configuration_ids`
  - Properly base64url-encoded `issuer_state`

## SATOSA Logs

- Look for:
  - SAML redirect errors
  - TLS failures
  - Trust validation issues
  - Entity not found errors

## SimpleSAMLphp Metadata

- Metadata reflects how SATOSA connects to SimpleSAMLphp (hostname, port).
- If internal Docker hostname is used, metadata will be incorrect externally.
- Use `baseurlpath` to override in `config.php`:

  ```php
  'baseurlpath' => 'https://<external-hostname>:<port>/simplesaml/',
  ```
