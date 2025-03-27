# Integrate Authentic Source with Datastore

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Format and validate documents](#format-and-validate-documents)
  - [Validate with `check-jsonschema`](#validate-with-check-jsonschema)
- [Test and verify functionality](#test-and-verify-functionality)
  - [Use the VC UI](#use-the-vc-ui)
    - [Configure the UI](#configure-the-ui)
    - [Log in to the UI](#log-in-to-the-ui)
    - [Explore the UI](#explore-the-ui)
    - [Search for documents](#search-for-documents)
      - [Perform document actions](#perform-document-actions)
    - [Upload documents from a CSV file](#upload-documents-from-a-csv-file)
    - [Upload documents as JSON](#upload-documents-as-json)
  - [Use Swagger UI](#use-swagger-ui)
- [Use cases](#use-cases)
  - [Upload document to datastore](#upload-document-to-datastore)
    - [Create JSON for EHIC document](#create-json-for-ehic-document)
    - [Add metadata to EHIC JSON](#add-metadata-to-ehic-json)
    - [POST document to `/upload`](#post-document-to-upload)
    - [Verify created document](#verify-created-document)
  - [Issue credential from document](#issue-credential-from-document)

---

## Introduction

This guide explains how to integrate the `vc_up_and_running` datastore by creating and managing documents and issuing credentials within the lab setup.

This documentation refers to `vc_up_and_running` tag **v0.5.17** and [API Specification v2.8](https://github.com/dc4eu/vc/blob/main/standards/api_specification.md).

---

## Prerequisites

Before you begin, make sure you have:

- A local instance of `vc_up_and_running`. See [System Components and Setup](https://github.com/dc4eu/dc4eu_documentation/blob/main/docs/setup_wallet_lab/system_components_and_setup.md) for details.

---

## Format and validate documents

You must format all documents according to their respective schemas within the `document_data` object. The API **does not validate documents** upon upload.

Validate documents manually using [check-jsonschema](https://pypi.org/project/check-jsonschema) or similar tools:

- [EHIC JSON Schema](https://github.com/dc4eu/vc/blob/main/standards/schema_ehic.json)
- [PD A1 JSON Schema](https://github.com/dc4eu/vc/blob/main/standards/schema_pda1.json)

### Validate with `check-jsonschema`

To install and run the validator:

```bash
pip install check-jsonschema
check-jsonschema --schemafile schemas/ehic-schema.md /path/to/ehic-document.json
```

Expected output:

```
ok -- validation done
```

---

## Test and verify functionality

You can interact with the datastore in multiple ways. This section outlines how to create, view, and manage documents and credentials using the VC UI or Swagger UI.

### Use the VC UI

The VC UI provides a graphical interface for document and credential operations.

#### Configure the UI

Update your `docker-compose.yaml` file to expose the UI externally:

```yaml
ui:
  ports:
    - 8081:8080
```

Then restart the services:

```bash
./stop.sh
./start.sh
```

Access the UI via your browser at `http://<your-host>:8081`.

#### Log in to the UI

Default credentials are:

```
Username: admin
Password: secret123
```

Update these in the `config.yaml` file if needed.

#### Explore the UI

Available features:

- Upload documents (manually, via JSON, or CSV)
- Search, view, and delete documents
- Upload business decisions
- Create and verify credentials
- Monitor container health status

#### Search for documents

Navigate to **Dev/test support → Search documents** to use advanced search parameters.

##### Perform document actions

Each search result includes a dropdown menu with:

- **View complete document** – View raw JSON
- **View QR** – See the credential offer QR code
- **Create credential** – Generate credential from document
- **Delete document** – Remove from datastore

#### Upload documents from a CSV file

See CSV format examples in the [upload_csv_templates](https://github.com/dc4eu/vc/tree/main/internal/ui/upload_csv_templates) directory.

#### Upload documents as JSON

Go to **Dev/test support → Upload document as JSON**, paste your JSON, and click **Upload**.

Successful response:

```
Response status: 200 (OK)
Content-Type: application/json; charset=utf-8
```

Payload:

```json
null
```

### Use Swagger UI

Access Swagger UI at:

```
http://<API-Gateway>/swagger/index.html
```

You can review endpoints and test requests interactively.

---

## Use cases

This guide covers common operations:

- Uploading EHIC or PD A1 documents
- Creating QR codes and credentials
- Revoking and deleting documents
- Managing test subject consent status

### Upload document to datastore

Use the `/upload` endpoint to create documents. See [API specification](https://github.com/dc4eu/vc/blob/main/standards/api_specification.md) for full reference.

#### Create JSON for EHIC document

Sample EHIC document:

```json
{
  "competent_institution": {
    "institution_country": "DE",
    "institution_id": "DE:123456",
    "institution_name": "German Health Insurance"
  },
  "document_id": "EHIC-DE-2024-123456",
  "period_entitlement": {
    "starting_date": "2024-01-01",
    "ending_date": "2025-01-01"
  },
  "social_security_pin": "1234567890",
  "subject": {
    "date_of_birth": "1990-05-15",
    "family_name": "Doe",
    "forename": "John",
    "other_elements": {
      "family_name_at_birth": "Doe",
      "forename_at_birth": "Johnathan",
      "sex": "01"
    }
  }
}
```

#### Add metadata to EHIC JSON

Wrap the above document in a full payload with metadata:

```json
{
  "document_data": { ... },
  "document_data_version": "1.0.0",
  "document_display": {
    "type": "EHIC",
    "version": "1.0.0",
    "description_structured": {
      "description_long": "European Health Insurance Card (EHIC)...",
      "valid_from": "1738368800",
      "valid_to": "1788368800"
    }
  },
  "identities": [ ... ],
  "meta": {
    "authentic_source": "DE-AS",
    "collect": {
      "id": "COLL-987654",
      "valid_until": 1750000000
    },
    "credential_valid_from": 1738368800,
    "credential_valid_to": 1758368800,
    "document_id": "EHIC-DE-2024-123456",
    "document_type": "EHIC",
    "document_version": "1.0.0",
    "real_data": false
  }
}
```

⚠️ In `v0.5.4`, omitting `collect` causes a `500 Internal Server Error`.

#### POST document to `/upload`

Save the document as `data.json` and run:

```bash
curl -X POST http://localhost:8080/api/v1/upload \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d @data.json
```

Expected response: `200 OK`.

#### Verify created document

Create a file `get.json`:

```json
{
  "authentic_source": "DE-AS",
  "document_id": "EHIC-DE-2024-123456",
  "document_type": "EHIC"
}
```

Send the verification request:

```bash
curl -X POST http://localhost:8080/api/v1/document \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d @get.json
```

Response contains your document data.

---

### Issue credential from document

Once the document is stored, issue an SD-JWT credential by POSTing to `/credential`.

Create a file `credential.json`:

```json
{
  "authentic_source": "DE-AS",
  "collect_id": "COLL-987654",
  "credential_type": "SD-JWT",
  "document_type": "EHIC",
  "identity": {
    "authentic_source_person_id": "12345",
    "schema": {
      "name": "DE",
      "version": "1.0.0"
    }
  }
}
```

Send the request:

```bash
curl -X POST http://localhost:8080/api/v1/credential \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d @credential.json
```

The response includes your signed SD-JWT credential.
