# Issuer Integration with the Trust Infrastructure

**Disclaimer**  
This document is a work in progress and subject to change. It describes the
integration steps for connecting an Issuer to the DC4EU Trust Infrastructure.
Some details may evolve as requirements are refined. Feedback is welcome.

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Key Configuration Points](#key-configuration-points)
  - [Entity Configuration Information](#entity-configuration-information)
  - [Trust Anchors](#trust-anchors)
  - [Authority Hints](#authority-hints)
  - [Registering the Issuer as a Subordinate Entity](#registering-the-issuer-as-a-subordinate-entity)
    - [Generating the Issuer Registration Document](#generating-the-issuer-registration-document)
      - [Option 1: Manual Method](#option-1-manual-method)
      - [Option 2: One-Liner Command (Automated)](#option-2-one-liner-command-automated)
      - [Final Step: Send the Document](#final-step-send-the-document)
  - [Trust Marks](#trust-marks)
    - [Types of Trust Marks](#types-of-trust-marks)
    - [Retrieving Trust Marks](#retrieving-trust-marks)
    - [Add Trust Marks to the `vc_up_and_running` Issuer](#add-trust-marks-to-the-vc_up_and_running-issuer)
    - [Testing Trust Marks](#testing-trust-marks)
  - [Steps to Connect the Issuer](#steps-to-connect-the-issuer)
  - [Testing the Trust Relationships](#testing-the-trust-relationships)

## Introduction

This guide explains how to connect an Issuer to the Trust Infrastructure,
including configuration of metadata, trust anchors, and trust marks.

## Prerequisites

Ensure the following are in place before continuing:

- Understanding of your trust architecture and credential types
- Network access from the Issuer to the Trust Anchor
- Externally accessible HTTPS endpoint for your Issuer

## Key Configuration Points

### Entity Configuration Information

The issuer must expose an entity configuration document at:

```bash
/.well-known/openid-federation
```

The configuration endpoint publishes the Entity Configuration document, which
provides the Issuer’s configuration details for participants in the Trust
Infrastructure.

- **Define the Endpoint:**
  The endpoint is defined under the following path: /.well-known/openid-federation
- **Implementation:**
  Ensure this endpoint serves the issuer’s Entity Configuration.
- **Example Entity Configuration:** Below is a shortened example of an Entity
  Configuration:

```json
{
  "sub": "https://issuer.example.com",
  "authority_hints": [
    "https://trust-anchor.example.org"
  ],
  "metadata": {
    "federation_entity": {
      "organization_name": "Example Issuer Org",
      "contacts": ["support@example.com"]
    },
    "oauth_authorization_server": {
      "token_endpoint": "https://issuer.example.com/token",
      "authorization_endpoint": "https://issuer.example.com/authorize",
      "jwks_uri": "https://issuer.example.com/jwks/oauth"
    },
    "openid_credential_issuer": {
      "credential_endpoint": "https://issuer.example.com/credential",
      "credential_configurations_supported": {
        "ExampleCredential": {
          "format": "vc+sd-jwt",
          "id": "example.credential.id",
          "vct": "ExampleCredential"
        }
      }
    }
  },
  "jwks": {
    "keys": [
      {
        "kty": "RSA",
        "use": "sig",
        "kid": "example-key-id",
        "e": "AQAB",
        "n": "example-modulus"
      }
    ]
  }
}
```

### Trust Anchors

The Trust Anchor (TA) is the root of the Trust Infrastructure’s trust chain.

- **Trust Anchor URL**:
  For your setup, the Trust Anchor URL is:
  `https://openidfed-test-1.sunet.se:7001`
- **Trust Anchor Keys**: Add the Trust Anchor’s  and public keys to your
  configuration:

```yaml
trust_anchors:
  https://openidfed-test-1.sunet.se:7001:
    keys:
      - kty: RSA
        use: sig
        kid: UFpoajluZU42dTNUUXo5RnhBVEJnRk9JY2NtU1JKdlVYUk1RUFRyVkFFRQ
        n: p9S2whcSjmBdxerp80tIJreUUmZiGNGXIocJlNjx9pgD5_WD2l6mBNuEZMpP-QUB_TSV3VesNiqmOdydGp1wkfQ-NmVdoso29FjEdgrckLIwirAVmVQ6bGQQnXJrR56mRz0QqENi11vVpbDj6hsprxK1EZBQL-sQ2kem289B_BCNT-NvwVHrYJlaQA32z7cs1a7W8wt9eLxA10PeiYMgDVU_69wKBw4YrjjozOHKMRGchUQEjQhfSZfk49bip_5TNz4dmBmSCIbdE2yilFrfRSNrh7q2myuyDE3k2QZbSOXXGGT1LtHO74WIY58v-M3A7_zxp0f2Eo9ZD3N4h-InIw
        e: AQAB
      - kty: EC
        use: sig
        kid: Nm82cTJKMDkydXhxOUMtTm0teFpMWlZiR0ZVa2U3YVVtbkJTV3hBd3FqOA
        crv: P-256
        x: 69XlQkKYfWJDXAv_Vbrqyfz9gfAhu1qQ4mtLde18-Cg
        y: ntBwdhy4_cS2PRBS-xdKkNwcO1yQP8TdoOHbHN9Yjv8
```

- **Purpose**: Trust Anchors validate Trust Marks’ signatures and establish
   trust within the Trust Infrastructure.

### Authority Hints

The `authority_hints` parameter specifies the URL of the Intermediate Entities
or Trust Anchors that are Immediate Superiors of the Entity.  This helps other
Trust Infrastructure participants understand upstream trust relationships.

- **Add to Configuration**: Add `authority_hints` in your issuer’s metadata
  configuration:

```yaml
authority_hints:
  - "https://openidfed-test-1.sunet.se:7001"
```

- **Purpose**: This parameter establishes hierarchical trust relationships from
   your issuer to the Trust Anchor.

### Registering the Issuer as a Subordinate Entity

In the Trust Infrastructure, the Issuer must be registered as a Subordinate
Entity under a Superior Entity (e.g., a Trust Anchor or an Intermediate Entity).
This ensures the Issuer's formal inclusion in the trust hierarchy.

#### Generating the Issuer Registration Document

To register the Issuer with the Trust Infrastructure, you need to create a JSON
document containing the Issuer’s public keys. You can do this **manually** or by
using a **one-liner command** to automate the process. Choose the method that
best suits your setup

##### Option 1: Manual Method

If you prefer to create the document manually, follow these steps:

Copy the Public Keys File to `issuer_registration.json`

```bash
cp satosa/public/pid_fed_keys.json issuer_registration.json
```

Edit issuer_registration.json:

- Open the file in a text editor of your choice
- Modify the contents to match the following structure,
  replacing **`<issuer-entity-id>`** with the actual **Issuer Entity
  Identifier** (e.g., `https://issuer.example.com`):

```json
{
  "<issuer-entity-id>": {
    "entity_types": [
      "federation_entity",
      "openid_credential_issuer",
      "oauth_authorization_server"
    ],
    "jwks": 
  }
}
```

Move the Public Keys into `jwks`:

- Locate the `"keys"` array already present in `issuer_registration.json`.
- Move it **inside** the `"jwks"` section so the structure looks like this:

```json
{
  "https://issuer.example.com": {
    "entity_types": [
      "federation_entity",
      "openid_credential_issuer",
      "oauth_authorization_server"
    ],
    "jwks": {
      "keys": [
        {
          "kty": "RSA",
          "use": "sig",
          "kid": "example-kid",
          "n": "example-n-value",
          "e": "AQAB"
        }
      ]
    }
  }
}
```

##### Option 2: One-Liner Command (Automated)

For users who prefer a quick and automated approach, use this single command to
generate the JSON document:

**Replace** `"https://issuer.example.com"` with the actual **Issuer Entity
URI.**

```bash
issuer_entity_uri="https://issuer.example.com" && jq --arg uri "$issuer_entity_uri" '{($uri): {"entity_types": ["federation_entity", "openid_credential_issuer", "oauth_authorization_server"], "jwks": .}}' satosa/public/pid_fed_keys.json > issuer_registration.json
```

##### Final Step: Send the Document

Once the file **`issuer_registration.json`** is created using either method,
send it to:  
**[support@dc4eu.eu](mailto:support@dc4eu.eu)**

---

### Trust Marks

Trust Marks are JWTs issued by a Trust Mark Issuer to validate compliance with
Trust Infrastructure policies.

#### Types of Trust Marks

The following Trust Marks are available for issuance:

- **EHIC Credential**:
  - **ID:** `http://dc4eu.example.com/EHICCredential/se`
- **PDA1 Credential**:
  - **ID:** `http://dc4eu.example.com/PDA1Credential/se`

#### Retrieving Trust Marks

For now, Trust Marks will be supplied when the entity is added to the **Trust Infrastructure**.

1. **Inputs to Trust Mark Issuer**:
   - **`id`**: The identifier for the Trust Mark (e.g., `http://dc4eu.example.com/EHICCredential/se`).
   - **`sub`**: The entity's Entity Identifier.
2. **Steps**:
   - Supply the **`id`** and **`sub`** to the Trust Mark Issuer.
   - Retrieve the issued Trust Mark as a signed JWT.
3. **Validation**:
   - Use a JWT library to verify the Trust Mark's signature using the Trust Mark
    Issuer's public key:
      - Retrieve public keys from the Trust Mark Issuer's /.well-known/jwks.json
        endpoint.
      - Validate claims such as `iss`, `sub`, `id`, and `iat` for compliance.
4. **Include in Metadata**: Add issued Trust Marks to your issuer’s metadata:

```yaml
config:
  op:
    trust_marks:
  - "eyJhbGciOiJIUzI1NiIsInR..."
  - "eyJhbGciOiJIUzI1NiIsInR..."
```

#### Add Trust Marks to the `vc_up_and_running` Issuer

To update the trust marks, you need to modify the `trust_marks` section of the
`satosa/plugins/oidc_frontend.yaml` file. Follow the steps below to replace the
existing trust marks with the ones received from the Trust Infrastructure
operator.

1. **Locate the trust marks section**  
   In the current configuration, the trust marks are defined under:

   ```yaml
   config:
     op:
       trust_marks:
         - "eyJhbGciOiJIUzI1NiIsInR..."
         - "eyJhbGciOiJIUzI1NiIsInR..."
   ```

   Replace these values with the updated trust marks provided by the operator.

2. **Example update**

   If the operator provided the following new trust marks:

   ```yaml
   eyJhbGciOiJSUzI1NiIsImtpZCI6IjM2NWQ2MjY3LTI5MzQtNGJhNy05YjEyLWU4ZmFkNTYwj9...
   eyJhbGciOiJSUzI1NiIsImtpZCI6IjkwNTFjZTgzLTY1NzEtNDliYi04ODdjLTc3OWQzMDNmJ9...
   ```

   Then update your configuration as follows:

   ```yaml
   trust_marks:
     - eyJhbGciOiJSUzI1NiIsImtpZCI6IjM2NWQ2MjY3LTI5MzQtNGJhNy05YjEyLWU4ZmFkNY...
     - eyJhbGciOiJSUzI1NiIsImtpZCI6IjkwNTFjZTgzLTY1NzEtNDliYi04ODdjLTc3OWQyJ9...
   ```

3. **Restart the Issuer to Apply Changes**

   Once you've updated the configuration file, restart the Issuer container to
   apply the changes:

   ```bash
   ./stop.sh && \
   ./start.sh
   ```

4. **Verify the Changes**  
   After restarting the Issuer, verify that the new Trust Marks are correctly
   applied:

   ```bash
   curl -k -s https://<issuer-host>:8000/.well-known/openid-federation | \
   cut -d '.' -f2 | tr '_-' '/+' | base64 -d 2>/dev/null | jq .
   ```

Look for the updated **trust_marks** in the JSON response.

#### Testing Trust Marks

1. **Decode JWT**: Use tools like `jwt.io` to inspect the Trust Mark's claims
   and ensure all required fields are present.
2. **Verify Signature**: Validate the JWT signature against the Trust Mark
   Issuer's public key.
3. **Check Expiration**: Ensure the `exp` claim (if present) has not expired.
4. **Validate References**: Follow the `ref` URL (if provided) to confirm
   compliance with human-readable policy documents.

---

### Steps to Connect the Issuer

1. **Configure the Issuer**:
   - Update the Issuer’s configuration to include `authority_hints`,
     `trust_marks`, and `trust_anchors`.
2. **Register with the Trust Infrastructure:**
   - Share your **`issuer_registration.json`** with the Trust Anchor or superior
     entity for registration.
3. **Validate Configuration**:
   - Test the issuer using testing tools or with a sandbox environment.
4. **Monitor the Connection**:
   - Regularly verify the status and ensure Trust Marks are up-to-date.

### Testing the Trust Relationships

1. **Validate trust marks**  
   Use tools like [jwt.io](https://jwt.io) to decode and verify trust marks
   using
   the Trust Anchor's public keys.

2. **Retrieve metadata**  
   Ensure the `.well-known/openid-federation` endpoint correctly serves the
   issuer’s entity configuration:

   ```bash
   curl -X GET https://your-issuer.example.com/.well-known/openid-federation
   ```

3. **Check authority hints**  
   Verify that `authority_hints` points to the correct Trust Anchor:

   ```yaml
   authority_hints:
     - https://openidfed-test-1.sunet.se:7001
   ```

4. **Validate public keys**  
   Confirm that the Trust Anchor’s public keys match those provided in your
   local configuration.
