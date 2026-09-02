# CLAUDE.md

## Project

AI Perfumery Engine — ระบบผู้ช่วยคำนวณและออกแบบกลิ่นน้ำหอมด้วยฟิสิกส์เคมี

## Product Context

The AI Perfumery Engine uses a chemical/physical dataset of 100 aroma materials.

The engine applies:
- Synergy rules
- Masking rules
- Evaporation over time
- Thresholds
- Material groups

The system displays predicted results through a dashboard.

The surrounding system includes:
- User accounts
- Login
- Saved formulas
- Evaluation panels
- Agreements

## Core Data

The system contains:

### Personal Data
Examples:
- User name
- Email
- Role
- Formula creator/editor
- Client brief contact
- Panel tester identity
- Interview notes

### Sensitive Personal Data

Examples:
- Allergy/sensitisation information
- Patch-test results
- Pregnancy status
- Religion-revealing preferences

### Owner Intellectual Property

The following are confidential:
- 100-material dataset
- Interaction rules
- Thresholds
- Material groups
- Saved formulas

These must not be sent to unapproved third-party AI/API/cloud services.

## Security Rules

- Never expose passwords or sensitive personal data.
- Perform authorization on the server.
- Do not rely only on client-side UI restrictions.
- Do not expose confidential formula information.
- Do not commit the real material dataset to a public repository.
- Do not send confidential product data to unapproved external services.

## Privacy

Personal-data features must follow `rule.md`.

Before storing personal data:
- Record consent.
- Define the purpose.
- Apply data minimisation.
- Define retention.
- Provide appropriate user data rights.

Sensitive personal data requires explicit owner approval before implementation.

## Access Logging

If authentication exists, the system must implement the access logging requirements defined in `rule.md`.

Important actions include:
- Login
- Logout
- Password changes
- Permission changes
- Formula creation/update/deletion
- Calculation runs
- Export/download
- Administrative access to another user's data

Access logs must follow the retention and append-only requirements in `rule.md`.

## Electronic Agreements

Agreements and approvals must follow the requirements in `rule.md`.

The system must preserve:
- Signer identity
- Timestamp
- Document version
- Document hash
- Authentication method
- Relevant signing metadata

High-risk actions require stronger authentication according to `rule.md`.

## AI / Calculation Rules

The calculation engine must be explainable.

Every calculated result should identify:
- The rules used
- Relevant thresholds
- Source data

If the engine has insufficient information, it must return:

"insufficient data"

It must not guess unsupported chemical/material interactions.

Changes to chemistry rules, thresholds, or material groups require approval from the domain expert.

Rule changes must be versioned.

Stored calculations must record the applicable rule version.

## Offline/Core Workflow

The core workflow should remain usable when an AI/model service is unavailable.

## Compliance Source

`rule.md` is the authoritative project-specific legal and compliance rule file.

Before modifying features involving:
- Personal data
- Access logs
- Agreements
- Owner IP
- AI-generated results

read and follow `rule.md`.

## Development Principles

- Do not invent requirements that are not supported by the product requirements or `rule.md`.
- Prefer data minimisation.
- Keep confidential data inside approved infrastructure.
- Add tests for important business logic.
- Document security-sensitive changes.
- Keep requirements traceable to implementation and audit items.