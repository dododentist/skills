---
name: clinic-administration
description: Review a DodoDentist clinic's safe administrative configuration and capacity — clinic metadata, bookable availability, provider display names, procedure catalog, non-identifying calendar blocks, and aggregate schedule counts. Use for read-only clinic operations. Never use it for patient records, individual appointments, clinical information, medical advice, diagnosis, triage, prescribing, or treatment decisions.
license: Apache-2.0
compatibility: Requires an MCP client with remote-server support and a DodoDentist account authorized through OAuth at https://mcp.dododentist.com/mcp
metadata:
  author: DodoDentist
  version: '1.0.0'
  repository: https://github.com/dododentist/skills
---

# Clinic administration with DodoDentist

This skill drives the hosted DodoDentist MCP connector. The user must connect
DodoDentist and authorize an account that can access the intended
organization. If authorization fails, ask the user to connect or reauthorize;
do not guess identifiers or retry with another tenant.

## Hard privacy and clinical boundary

This connector is administrative-only. It intentionally does not expose:

- patient identities, records, contact details or record counts;
- individual appointment records, identifiers, statuses, procedures or times;
- diagnoses, treatment plans, procedure instances, prescriptions, medical
  history, anamnesis, tooth charts, images or notes;
- provider identifiers/contact details, credentials or organization internals;
  or
- tools that create, update, message, cancel or delete records.

Never ask the user to paste excluded health data into chat, infer it from
aggregate counts, or imply that a missing tool can be approximated from
another result. The connector does not diagnose, triage, prescribe, recommend
treatment, determine urgency or make any medical decision.

Bookable availability is unassigned clinic capacity. A procedure catalog is
administrative configuration. Neither is evidence that a particular person
has an appointment or receives a treatment.

## Available tools

| Tool                          | Use it for                                                        |
| ----------------------------- | ----------------------------------------------------------------- |
| `list_clinics`                | Resolve organization-owned clinic IDs and safe metadata           |
| `get_clinic`                  | Read safe metadata for one owned clinic                           |
| `get_clinic_availability`     | Review up to seven days and 100 bookable slots                    |
| `list_clinic_providers`       | List up to 100 provider display names                             |
| `get_clinic_schedule_summary` | Count scheduled records by UTC day for a range of at most 31 days |
| `list_procedures`             | Page through procedure-catalog names, prices and edit times       |
| `list_calendar_events`        | Page through non-appointment block times, scope, effect and type  |
| `show_clinic_overview`        | Render a bounded administrative overview for one clinic           |

Every tool is read-only. There is no connector operation for booking,
rescheduling, cancellation, messaging, patient lookup, clinical-record access,
diagnosis or treatment.

## Identifiers and pagination

Never guess a clinic ID. Start with `list_clinics`, choose the clinic that
matches the user's request, and reuse its returned 24-character hexadecimal
ID. If several clinics match, ask which one they mean.

`list_procedures` and `list_calendar_events` return opaque cursors. Reuse a
cursor only with the same clinic, time range and limit that produced it. Start
a new listing when a filter changes. Do not interpret or edit a cursor.

Provider and procedure identifiers are intentionally absent. Do not claim that
a display name or catalog name uniquely identifies an internal record.

## Administrative overview workflow

When the user wants a broad clinic snapshot:

1. Call `list_clinics`.
2. Resolve the intended clinic without guessing.
3. Call `show_clinic_overview` with that clinic ID.
4. Interpret only the safe metadata, aggregate next-seven-day schedule,
   provider/availability counts and procedure-catalog preview rendered by the
   card.

Counts with `+` are lower bounds from bounded reads. Do not describe them as
complete totals.

## Schedule and capacity workflow

For workload over a period:

1. Resolve the clinic with `list_clinics`.
2. Call `get_clinic_schedule_summary` with ISO 8601 start/end values. Omit
   both for the default seven-day range. Never request more than 31 days.
3. Report only the aggregate total and UTC day counts.
4. Use `get_clinic_availability` separately when the user wants open capacity.

The schedule summary deliberately omits individual times and records. Do not
try to reconstruct them. Availability can show exact open slot start/end times
because those slots are unassigned.

## Providers and procedure catalog

Use `list_clinic_providers` for bookable provider display names. It provides no
IDs, contact details or role records.

Use `list_procedures` for administrative catalog names/prices. Follow
`nextCursor` only when the user needs more results. Never describe catalog
entries as a person's diagnosis, treatment history or recommended care.

## Calendar blocks

Use `list_calendar_events` for closures, meetings, courses, days off and other
non-appointment blocks. It exposes only block times, all-day status, scope,
effect and type. Titles, notes, event IDs and staff references are
intentionally unavailable.

Do not infer a reason or person from a block. If the user needs the hidden
free-text details, explain that the connector does not provide them.

## Unsupported requests

For a patient/clinical request, state that the public connector cannot access
that data. For a write request, state that all available tools are read-only.
For diagnosis, urgency, medication or treatment advice, do not invoke
DodoDentist as a clinical authority; follow the host's current safety guidance
and make clear that the connector provides no medical decision.
