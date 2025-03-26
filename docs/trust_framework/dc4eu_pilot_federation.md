# DC4EU Pilot Federation

## Disclaimer

This document provides a practical overview of key processes related to metadata
and Trust Mark validation in the OpenID Federation. It is not exhaustive and
does not cover all aspects of the OpenID Federation 1.0 specification. For
complete details, including advanced use cases and comprehensive workflows,
refer to the [OpenID Federation 1.0
specification](https://openid.net/specs/openid-federation-1_0.html).

Implementers are advised to consult the official specification to ensure full
compliance and alignment with federation standards.

## Table of Contents

- [Disclaimer](#disclaimer)
- [Introduction](#introduction)
- [Purpose of the Federation](#purpose-of-the-federation)
- [Key Features](#key-features)
- [Scope](#scope)
- [Terminology](#terminology)
- [Federation Overview](#federation-overview)
- [Federation Architecture and Core Entities](#federation-architecture-and-core-entities)
  - [Key Entities and Their Roles](#key-entities-and-their-roles)
- [Trust Workflow](#trust-workflow)
- [Standards and Protocols](#standards-and-protocols)
- [Trust Establishment and Validation](#trust-establishment-and-validation)
- [Role of the Trust Anchor](#role-of-the-trust-anchor)
- [Trust Mark Issuance](#trust-mark-issuance)
  - [Process](#process)
  - [Trust Marks in the Federation](#trust-marks-in-the-federation)
- [Entity Validation](#entity-validation)
- [Dynamic Trust Relationships](#dynamic-trust-relationships)
- [Examples of Trust Workflows](#examples-of-trust-workflows)
- [Federation Nodes](#federation-nodes)
  - [Trust Anchor](#trust-anchor)
  - [Trust Mark Issuer](#trust-mark-issuer)
  - [Wallet Provider](#wallet-provider)
  - [Credential Issuer](#credential-issuer)
- [Usage Notes](#usage-notes)
  - [Metadata Retrieval](#metadata-retrieval)
  - [Decoding Metadata](#decoding-metadata)
- [Validating Metadata Signatures](#validating-metadata-signatures)
- [Security Notes](#security-notes)

## Introduction

The OpenID Federation (OIDF) forms the backbone of the DC4EU Pilot Wallet
ecosystem, enabling secure, scalable, and interoperable interactions between
diverse entities such as credential issuers, verifiers, and wallet providers.

## Purpose of the Federation

- **Trust Framework:** Establish and maintain a scalable trust model based on
  cryptographic proofs and hierarchical relationships.
- **Interoperability:** Enable cross-border, multi-stakeholder interaction while
  ensuring technical, semantic, and organizational compatibility.
- **Ecosystem Integration:** Support the issuance, storage, and verification of
  credentials such as EHIC and PDA1 within the EUDIW ecosystem.

## Key Features

- **Dynamic Metadata Exchange:** Entities within the federation dynamically
  exchange metadata, ensuring up-to-date trust information.
- **Standards-Based Protocols:** Built on OpenID Connect extensions like
  OpenID4VCI for credential issuance and OpenID4VP for presentation.
- **Scalability and Flexibility:** The federation’s design supports dynamic
  trust relationships, allowing seamless onboarding of new entities and
  adaptation to evolving requirements.

## Scope

This documentation focuses on the OpenID Federation's architecture, its role in
the DC4EU interoperability lab, and how it aligns with the overarching goals of
the EUDIW ecosystem.

## Terminology

This section defines key terms and concepts used throughout the document to
ensure clarity and consistency when discussing the federation.

- **Authority Hints**  
  - **Definition**: A list of superior entities that an entity trusts,
    typically pointing to the Trust Anchor or intermediary entities.  
  - **Purpose**: Defines the trust chain and establishes hierarchical trust
    relationships.

- **Entity Configuration Document**  
  - **Definition**: A JSON Web Token (JWT) that defines an entity’s identity,
    role, and trust relationships within the federation.  
  - **Purpose**: Used for dynamic metadata exchange and trust validation.

- **Trust Anchor (TA)**  
  - **Definition**: The root of trust in the federation, responsible for
    signing and validating metadata for subordinate entities.  
  - **Purpose**: Establishes the federation’s trust framework.

- **Trust Mark**  
  - **Definition**: A cryptographically signed JWT issued by a Trust Mark
    Issuer to certify that an entity complies with specific federation
    policies.  
  - **Purpose**: Provides assurance that the entity meets federation standards.

- **Trust Mark Issuer (TMI)**  
  - **Definition**: An entity that issues Trust Marks to certify compliance
    with federation policies.  
  - **Purpose**: Ensures entities meet operational and policy-based
    requirements.

- **Wallet Provider**  
  - **Definition**: An entity within the federation that provides metadata and
    enables Wallet Instance registration for wallets.  
  - **Purpose**: Facilitates wallet interaction with the federation.

- **Credential Issuer**  
  - **Definition**: An entity responsible for issuing verifiable credentials
    (e.g., EHIC, PDA1) to wallets.  
  - **Purpose**: Provides credentials that can be verified by other entities.

- **JWKS (JSON Web Key Set)**  
  - **Definition**: A set of public keys in JSON format.  
  - **Purpose**: Used for verifying the signatures of JWTs.

- **JWT (JSON Web Token)**  
  - **Definition**: A compact, URL-safe means of representing claims between
    two parties. Often used for metadata and Trust Marks in the federation.  
  - **Purpose**: Encodes information securely with optional signing or
    encryption.

- **Metadata**  
  - **Definition**: Information published by entities in the federation that
    describes their identity, roles, and capabilities.  
  - **Purpose**: Enables dynamic discovery and trust establishment.

- **Dynamic Metadata Exchange**  
  - **Definition**: The process by which entities in the federation retrieve
    and validate each other.  
  - **Purpose**: Allows for scalable and flexible trust relationships.

- **Key ID (kid)**  
  - **Definition**: An identifier included in a JWT header that specifies which
    key in a JWKS should be used for signature verification.  
  - **Purpose**: Helps identify and validate the correct public key.

- **Selective Disclosure**  
  - **Definition**: A feature of credentials that allows holders to share only
    specific claims or data fields with verifiers.  
  - **Purpose**: Enhances privacy and minimizes data exposure.

## Federation Overview

The OpenID Federation in the DC4EU Wallet ecosystem operates as a dynamic trust
framework, enabling seamless and scalable interactions between issuers, wallets,
and verifiers. By leveraging cryptographic proofs and hierarchical
relationships, the federation ensures secure credential issuance, management,
and verification.

## Federation Architecture and Core Entities

The federation follows a hierarchical structure, with the Trust Anchor (TA)
serving as the root of trust. Subordinate entities, including Trust Mark
Issuers, Credential Issuers, Wallet Providers, and Verifiers, establish their
roles through dynamic metadata exchanges and trust chains.

### Key Entities and Their Roles

- **Trust Anchor (TA)**  
  - Acts as the root of trust, defining federation policies and signing
    metadata.  
  - Ensures the authenticity and integrity of the federation’s trust framework.

- **Trust Mark Issuer (TMI)**  
  - Issues Trust Marks to certify entities’ compliance with trust and
    interoperability requirements.  
  - Plays a critical role in policy enforcement within the federation.

- **Credential Issuers**  
  - Generate and issue verifiable credentials (e.g., EHIC, PDA1).  
  - Interface with authentic sources to retrieve user attributes.

- **Wallet Providers**  
  - Enable Wallet Instance registration and interaction with the federation.  
  - Act as the primary interface for wallets, which are external to the
    federation.

- **Verifiers**  
  - Validate credentials presented by holders.  
  - Ensure that credentials adhere to defined schemas and trust models.

## Trust Workflow

The trust workflow involves metadata discovery, validation, and trust
establishment:

1. **Metadata Exchange**  
   - Entities publish their Entity Configuration Document at
     `/.well-known/openid-federation`.  
   - Contents of the Entity Configuration Document include (but are not limited
     to):
     - **Entity ID**: Unique identifier for the entity.
     - **Entity Role(s)**: Specifies the role of the entity (e.g., Credential
       Issuer, Trust Anchor).
     - **Authority Hints**: Lists trusted superior entities, such as the Trust
       Anchor.
     - **JWKS**: Either a `jwks_uri` pointing to a JSON Web Key Set, or a `jwks`
       object directly embedded in the Entity Configuration. Used for verifying
       digital signatures.
     - **Trust Marks**: Optional. May include signed trust marks issued *to* the
       entity (to demonstrate compliance) or *by* the entity (if it acts as a
       Trust Mark Issuer).
2. **Validation**  
   - Metadata published by entities is validated by other federation
     participants using the Trust Anchor's public keys.  
   - Trust Marks issued by the TMI are used to attest to an entity's conformance
     with a well-scoped set of trust and interoperability requirements, as
     determined by an accreditation authority.
3. **Dynamic Trust Relationships**  
   - Unlike static configurations, trust is established dynamically through
     metadata exchange, enabling scalability and flexibility.

## Standards and Protocols

1. **OpenID Connect:** Forms the foundation for authentication and API security.
2. **OpenID Federation:** Extends OpenID Connect with dynamic metadata exchange
   and hierarchical trust.
3. **OpenID4VCI:** Enables credential issuance using OAuth2 flows.
4. **OpenID4VP:** Facilitates credential presentation for selective disclosure.

## Trust Establishment and Validation

In the OpenID Federation, trust is established and validated dynamically through
cryptographic proofs, hierarchical relationships, and metadata exchange. This
ensures secure interactions among entities while supporting scalability and
flexibility across the DC4EU Wallet ecosystem.

## Role of the Trust Anchor

The Trust Anchor (TA) is the root of trust for the federation. It serves as the
foundational authority, enabling trust relationships by:

- **Defining Policies:** Establishing the rules and compliance criteria for
  entities within the federation.
- **Signing Subordinate Statements:** Issuing signed statements about
  subordinates, which include attributes such as keys and roles, to establish a
  chain of trust that allows relying parties to validate entities.
- **Anchoring Trust:** Acting as the central point of trust verification for the
  federation hierarchy.

## Trust Mark Issuance

The Trust Mark Issuer (TMI) plays a critical role in attesting to an entity’s
conformance with a well-scoped set of trust and interoperability requirements as
determined by an accreditation authority.

### Process

1. **Request Submission**  
   An entity (e.g., a Credential Issuer) submits a request for a Trust Mark to
   the TMI.
2. **Evaluation**  
   The TMI evaluates whether the entity meets the predefined trust and
   interoperability requirements.
3. **Issuance**  
   If compliant, the TMI issues a Trust Mark, which is a signed JSON Web Token
   (JWT) containing, but not limited to:

   - **`iss`**: The issuer of the Trust Mark (the TMI).
   - **`sub`**: The entity receiving the Trust Mark.
   - **`id`**: The Trust Mark identifier, representing the specific policy or
     requirement being attested to.
   - **`iat`**: The "issued at" timestamp indicating when the Trust Mark was
     created.
   - **`exp`**: The expiration timestamp defining the Trust Mark’s validity
     period.

### Trust Marks in the Federation

1. **EHIC Credential Trust Mark**
   - **ID**: `http://dc4eu.example.com/EHICCredential/se`
   - **Purpose**: Attests to the entity’s compliance with EHIC credential
     issuance standards.
2. **PDA1 Credential Trust Mark**
   - **ID**: `http://dc4eu.example.com/PDA1Credential/se`
   - **Purpose**: Certifies the entity’s adherence to PDA1 requirements.

## Entity Validation

Entities within the federation validate each other using cryptographic
signatures, hierarchical trust relationships, and dynamic metadata discovery.
To optimize performance, signature validation is recommended as the final step,
following successful completion of other checks.

- **Metadata Retrieval**
  - Retrieve metadata from an entity’s published
    `/.well-known/openid-federation` endpoint.
- **Trust Chain Validation**
  - Validate the trust chain by resolving superior entities as indicated in the
    metadata.
  - Ensure the trust chain terminates at a recognized Trust Anchor.
- **Preliminary Validation**
  - Check the metadata for completeness, correctness, and adherence to
    federation requirements.
  - Confirm consistency between the entity’s declared roles and its expected
    behavior.
- **Signature Validation**
  - Validate cryptographic signatures to ensure the authenticity and integrity
    of the metadata and any associated endorsements.

## Dynamic Trust Relationships

Dynamic trust relationships in the federation enable scalable and flexible
interactions between entities. This approach relies on:

- **Dynamic Metadata Exchange**: Entities publish their metadata at endpoints
  like `/.well-known/openid-federation`, allowing others to discover and
  validate them in real time.
- **Trust Chain Validation**: Trust is established through hierarchical
  relationships, ensuring that each entity in the chain is endorsed by its
  superior and ultimately rooted in a Trust Anchor.

**Benefits of Dynamic Trust**:

1. **Scalability**: New entities can join the federation without requiring
   preconfigured relationships or manual updates.
2. **Flexibility**: Trust relationships adapt to changes in roles, policies, or
   compliance requirements.
3. **Efficiency**: Decentralized validation ensures that entities interact
   directly without relying on a central authority for every operation.

## Examples of Trust Workflows

1. **Credential Issuance Flow**
   - Wallets interact with the Wallet Provider to register as Wallet Instances.
   - The Wallet Provider’s metadata is discovered dynamically from its
     `/.well-known/openid-federation` endpoint.
   - The registration process includes securely exchanging information to
     establish a connection and configure supported capabilities.
   - Upon successful registration, the Wallet Provider confirms the instance’s
     configuration and readiness to interact with Credential Issuers for
     credential issuance.
2. **Credential Presentation Flow**
   - A Holder presents credentials to a Verifier during an interaction.
   - The Verifier validates the credentials by:
     - Retrieving and validating metadata from the Credential Issuer’s
       `/.well-known/openid-federation` endpoint.
     - Validating the associated Trust Marks to certify the entity's
       conformance with specific trust and interoperability requirements.
     - Ensuring the credential aligns with expected schemas and is issued by a
       trusted Credential Issuer.

## Federation Nodes

The following are the endpoints of the federation, along with the Trust Anchor's
associated cryptographic keys.

### Trust Anchor

- **Role**: The Trust Anchor (TA) serves as the root of trust, signing and
  validating metadata for subordinate entities.
  - **Endpoint**: `https://openidfed-test-1.sunet.se:7001`
  - **Public Keys**:

```json
{
   "https://openidfed-test-1.sunet.se:7001": {
      "keys": [
            {
               "kty": "RSA",
               "use": "sig",
               "kid": "UFpoajluZU42dTNUUXo5RnhBVEJnRk9JY2NtU1JKdlVYUk1RUFRyVkFFRQ",
               "n": "p9S2whcSjmBdxerp80tIJreUUmZiGNGXIocJlNjx9pgD5_WD2l6mBNuEZMpP-QUB_TSV3VesNiqmOdydGp1wkfQ-NmVdoso29FjEdgrckLIwirAVmVQ6bGQQnXJrR56mRz0QqENi11vVpbDj6hsprxK1EZBQL-sQ2kem289B_BCNT-NvwVHrYJlaQA32z7cs1a7W8wt9eLxA10PeiYMgDVU_69wKBw4YrjjozOHKMRGchUQEjQhfSZfk49bip_5TNz4dmBmSCIbdE2yilFrfRSNrh7q2myuyDE3k2QZbSOXXGGT1LtHO74WIY58v-M3A7_zxp0f2Eo9ZD3N4h-InIw",
               "e": "AQAB"
            },
            {
               "kty": "EC",
               "use": "sig",
               "kid": "Nm82cTJKMDkydXhxOUMtTm0teFpMWlZiR0ZVa2U3YVVtbkJTV3hBd3FqOA",
               "crv": "P-256",
               "x": "69XlQkKYfWJDXAv_Vbrqyfz9gfAhu1qQ4mtLde18-Cg",
               "y": "ntBwdhy4_cS2PRBS-xdKkNwcO1yQP8TdoOHbHN9Yjv8"
            }
      ]
   }
}
```

### Trust Mark Issuer

- **Role**: The Trust Mark Issuer (TMI) certifies entities’ compliance with
  federation policies by issuing cryptographic Trust Marks.
  - **Endpoint**: `https://openidfed-test-1.sunet.se:6001`
  - **Notes:** Metadata for the TMI is accessible at:
    `https://openidfed-test-1.sunet.se:6001/.well-known/openid-federation`

### Wallet Provider

- **Role**: Acts as the intermediary for wallets to interact with the
  federation, supporting Wallet Instance registration.
  - **Endpoint**: `https://openidfed-test-1.sunet.se:5001`
  - **Notes**: Metadata for the Wallet Provider is accessible at:
  `https://openidfed-test-1.sunet.se:5001/.well-known/openid-federation`

### Credential Issuer

- **Role**: Issues credentials (e.g., EHIC, PDA1) to wallets upon successful
  interaction.
- **Endpoint**: `https://satosa-test-1.sunet.se/`
- **Notes**: Supports credential issuance based on OpenID4VCI protocols.

## Usage Notes

### Metadata Retrieval

Retrieve the Entity Configuration Document from a federation node’s
`/.well-known/openid-federation` endpoint:

```bash
curl https://openidfed-test-1.sunet.se:7001/.well-known/openid-federation
```

The metadata is encoded as a JSON Web Token (JWT), which must be validated and
decoded securely

### Decoding Metadata

Decode the JWT to inspect the payload:

- Extract the payload (the second segment of the JWT).
- Decode it using tools such as `base64` and  use tools like `jq` to explore the
  JSON payload:

```bash
echo '<Base64URL-encoded payload>' | tr '_-' '/+' | base64 - | jq
```

## Validating Metadata Signatures

To validate the signature of metadata, the public keys of the entity can be
represented in one of three ways in the metadata. Each representation has
specific handling requirements.

- **`jwks` (JSON Web Key Set by Value)**:
  - The keys are embedded directly in the metadata. They are inherently
    validated as part of the metadata’s cryptographic signature. No additional
    fetch operation is required.
- **`jwks_uri` and `signed_jwks_uri`**:
  - **`jwks_uri`**: A URI pointing to the JSON Web Key Set hosted by the
  entity.
  - **`signed_jwks_uri`**: A URI pointing to a signed JWT containing the key
    set.
  - In both cases, the keys must be fetched from the URI:  
    ``` curl <jwks-uri> ```
  - For `signed_jwks_uri`, the fetched JWT must also be validated before the
    keys can be used.

**Signature Validation**:  
Use a library or tool such as **CryptoJWT** to validate the signature.
**CryptoJWT** is a Python library specifically designed for OpenID-related
standards, including JSON Web Token (JWT) handling and signature verification.

## Security Notes

**Always validate JWT signatures** using trusted public keys before using the
data. Ensure the key’s `kid` (Key ID) in the JWT header matches a key in the
`jwks.json` document.
