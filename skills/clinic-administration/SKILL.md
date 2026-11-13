---
name: clinic-administration
description: Run a DodoDentist dental practice's schedule and website through its connector — free slots, providers, treatment prices, calendar blocks, daily appointment counts and stock, a patient the user names and that patient's or one day's appointments, booking, moving or cancelling appointments, adding or updating a patient's contact details, correcting stock, and the practice's website settings and website builder draft. Use when a dentist or practice manager asks about availability, how busy a day is, prices, closures, stock, asks to book, reschedule or cancel a patient, add a patient or change their phone or email, or asks to change their website. Never use it for clinical records, diagnosis, triage, prescribing or treatment decisions.
license: Apache-2.0
compatibility: Requires an MCP client with remote-server support and a DodoDentist account authorized through OAuth at https://mcp.dododentist.com/mcp
metadata:
  author: DodoDentist
  version: '1.3.0'
  repository: https://github.com/dododentist/skills
---

# Clinic administration with DodoDentist

This skill drives the hosted DodoDentist MCP connector. The user must connect
DodoDentist and authorize an account that can access the intended
organization. If authorization fails, ask the user to connect or reauthorize;
do not guess identifiers or retry with another tenant. Never request, embed or
repeat a password, OAuth token or API key.

## What the connector changes

The read tools change nothing. Nine tools write, as the signed-in member, with
that member's DodoDentist role: `book_appointment`, `reschedule_appointment`,
`cancel_appointment`, `create_patient`, `update_patient`,
`set_inventory_quantity`, `update_website_settings`, `write_website_files` and
`restore_website_checkpoint`. DodoDentist's own rules apply to every write: a
dentist sees and changes only what their role allows (the website needs the
Website permission), and the plan's monthly appointment limit counts bookings
made here. At the limit a booking is refused (`plan_limit_reached`); the
connector never moves the practice to a larger plan, the owner does that in
the app.

If the write tools are missing, the connection was authorized to read only
(every connection made before November 2026 was). Ask the user to disconnect
DodoDentist and connect it again, allowing DodoDentist to create and update
data.

Confirm every write with the user before calling it: which patient, which
practice, when, and what will change. Booking, moving and cancelling reach the
patient: a booking schedules the practice's reminder messages (so does giving
a date to a visit still to schedule), a move moves them, and a cancellation
sends the practice's cancellation message when the practice set one up. Say so
when you ask.

The connector cannot delete anything, take or refund payments, send a message
of its own, change reminder settings, create a repeating series of
appointments, plan a visit without a date, or change procedures, prices, staff
or roles. Direct the user to the DodoDentist app for those.

## Hard privacy and clinical boundary

Being a patient of a dental practice is itself health information. The
connector therefore shows a patient only when the user names them, and only
what is needed to book them:

- `find_patient` needs at least three characters of the name; every word given
  must be in the stored name. It returns at most five matches with id, stored
  name, practice and edit time. When two share a name it adds the last two
  digits of each one's phone, so the user can say which. More than five matches
  return nobody: ask for more of the name.
- `list_appointments` shows one patient's appointments, or one day's, with
  patient and provider ids, times, status and edit time.

It never lists, browses or counts patients, and never returns a phone number,
email, date of birth, address, note, the procedures an appointment holds, or
any clinical record: anamnesis, tooth charts, findings, treatments,
prescriptions, imaging, documents or consents. No tool accepts those as input
either; a booking takes no note. Ask for a patient's phone or email only to
add or update that patient, never to answer a question.

Never ask the user to paste health data into chat, infer it from appointment
times, or imply that a missing tool can be approximated from another result.
The connector does not diagnose, triage, prescribe, recommend treatment,
determine urgency or make any medical decision.

## Resource identifiers

Keep these identifiers distinct:

- `clinicId` selects an owned practice. A write may leave it out when the
  account has one practice.
- `patientId` names one patient (from `find_patient` or `create_patient`);
  `appointmentId` one appointment; `inventoryItemId` one stock item.
- `providerIds` are the ids `list_clinic_providers` returns; `procedureIds`
  the ids `list_procedures` returns.
- `lastEditTime` is a record's version. Moving or cancelling an appointment,
  updating a patient and setting a stock quantity take it as
  `expectedLastEditTime` and are refused with `conflict` when the record
  changed since it was read.
- `requestId` is a unique id you choose for each new booking or patient, such
  as a UUID. Calling again with the same `requestId` returns the first result
  instead of booking or adding twice.
- `websiteId` names one website started in the website builder (from
  `get_website_settings`); a checkpoint is one saved version of its files, a
  40-character id.

Never guess an id, treat an opaque id as authority, or reuse an id from
another organization. If several practices match, ask which one they mean.

## Available tools

| Tool                          | Use it when someone asks…                                                                    |
| ----------------------------- | -------------------------------------------------------------------------------------------- |
| `list_clinics`                | which practices they run; also resolves the clinic ID other tools take                       |
| `get_clinic`                  | about one practice's currency, language or timezone                                          |
| `get_clinic_availability`     | when there are free appointment times (up to 7 days with openings)                           |
| `list_clinic_providers`       | which dentists, nurses or assistants can be booked, with their ids                           |
| `get_clinic_schedule_summary` | how busy each UTC day is (appointment counts, at most 31 days)                               |
| `list_procedures`             | what a treatment costs, or which treatment id to book                                        |
| `list_calendar_events`        | when the practice is closed or a dentist or seat is blocked                                  |
| `find_patient`                | for the id of a patient they named                                                           |
| `list_appointments`           | when a patient is booked, what is booked on one day, or what a patient still has to schedule |
| `list_inventory_items`        | how much of a supply is left, or what is running low                                         |
| `show_clinic_overview`        | for a one-card overview of one practice                                                      |
| `book_appointment`            | to book a patient in                                                                         |
| `reschedule_appointment`      | to move an appointment to another day or time, or give a date to one still to schedule       |
| `cancel_appointment`          | to cancel an appointment                                                                     |
| `create_patient`              | to add a new patient                                                                         |
| `update_patient`              | to change a patient's name, phone, email or language                                         |
| `set_inventory_quantity`      | to correct or update a stock count                                                           |
| `get_website_settings`        | about their website: its address, colours, social links, booking window or builder drafts    |
| `update_website_settings`     | to change the website's colours, social links, WhatsApp number or online booking window      |
| `read_website_files`          | what the website builder draft says, before changing it                                      |
| `write_website_files`         | to change the wording, colours, layout or sections of the website builder draft              |
| `list_website_checkpoints`    | what changed on the website builder draft, or which version to go back to                    |
| `restore_website_checkpoint`  | to undo changes to the website builder draft                                                 |

`list_procedures` and `list_calendar_events` return opaque cursors. Reuse a
cursor only with the same practice, time range and limit that produced it.
`list_appointments` takes one day at a time without a `patientId`, and up to a
year with one; report `hasMore` rather than claiming completeness.

## Booking workflow

1. Find the patient with `find_patient` and the name the user gave. If nobody
   matches, offer to add them with `create_patient`, then use the id it
   returns. If several match, ask which one, using the phone digits when
   given.
2. Use `get_clinic` for the practice's timezone, and `get_clinic_availability`
   when the user has no time in mind.
3. Use `list_procedures` for the treatments asked for, and
   `list_clinic_providers` when the practice has several providers and the
   user named one or must choose.
4. Confirm the patient, practice, local day and time, treatments and provider
   with the user, then call `book_appointment` with a new `requestId`. Give
   `startTime` with the practice's UTC offset. Leave `durationMinutes` out for
   the practice's usual length.
5. Report the booked time in the practice's timezone, and that the practice's
   reminders will reach the patient.

`book_appointment` books only staff who take appointments and only a patient
of the same practice. With several providers and none named, it answers
`provider_required`: ask which one. Its result counts the treatments booked
without naming them.

## Moving or cancelling an appointment

1. Find the appointment with `list_appointments` (by `patientId`, or by day)
   and keep its `id` and `lastEditTime`. A provider's bookings are those whose
   `providerIds` hold the provider's id; move or cancel them one at a time,
   each confirmed.
2. Confirm the change with the user, then call `reschedule_appointment` (the
   patient, providers, treatments and length stay the same unless a new
   `durationMinutes` is given) or `cancel_appointment`. Cancelling keeps the
   appointment as canceled; it never deletes it.
3. On `conflict`, read the appointment again, tell the user what changed, and
   ask again before retrying with the new `lastEditTime`.

Requested and booked appointments can be moved or canceled, and one still to
schedule can be given a date. One the patient attended or missed stays as it
is.

## Booking a visit still to schedule

An appointment with status `TO_SCHEDULE` is a visit the dentist planned that
the practice has not given a date yet: its `startTime` and `endTime` are null,
it holds no slot and nothing about it has reached the patient.
`list_appointments` with a `patientId` lists them first; a day's list never
holds them.

1. Find the patient with `find_patient`, call `list_appointments` with their
   `patientId`, and keep the planned visit's `id` and `lastEditTime`.
2. Agree the day and time with the user (`get_clinic_availability` shows free
   slots), confirm them, then call `reschedule_appointment` with the new
   `startTime` and the visit's `lastEditTime` as `expectedLastEditTime`. Leave
   `durationMinutes` out to keep the length the dentist planned, or the
   practice's usual length when none was planned.
3. It is then an ordinary booking and the practice's reminders reach the
   patient: say so when you ask, and report the booked time in the practice's
   timezone.

## Patients

`create_patient` needs a name; phone, email and language are optional. The
phone and email are saved for the practice's messages but never returned. Use
`update_patient` with the patient's `lastEditTime` from `find_patient` to
change a name, phone, email or language; only the fields given change.

## Stock

`list_inventory_items` shows each item's unit, quantity, minimum and whether
it is at or below its minimum. `set_inventory_quantity` replaces the stored
quantity with the number given: when the user reports a change ("we used
two boxes"), work out the new total from the quantity you read, confirm it,
and send that total with the item's `lastEditTime`.

## The practice's website

A practice has a public site at its own address, built from its records and
its website settings, and may also have drafts in the app's website builder.

Website settings show on the public site at once:

1. Call `get_website_settings` with the practice's `clinicId`.
2. Confirm the change with the user, then call `update_website_settings` with
   only the settings that change and the `lastEditTime` it returned as
   `expectedLastEditTime`. Colours are `#rrggbb`; social profiles take a
   handle or a link; `null` removes one.
3. Report what changed and where it shows. The subdomain, a custom domain, the
   logo and the banner change in the DodoDentist app. Opening hours, address,
   phone, treatments, prices and staff come from the practice's records, also
   changed in the app.

A website builder draft is a small set of files. It is not the public site:
the dentist sees it in the app's website builder, and the public site keeps
showing the practice's template website.

1. `get_website_settings` lists the practice's drafts with their `websiteId`.
   With none, the dentist starts one from a template in the app's builder.
2. Call `read_website_files` without `paths` for the file list and how the
   site is built, then read the files you will change. Keep the `checkpoint`
   it returns.
3. Make the smallest change that does what was asked. `src/copy.js` holds
   every word by language: change wording in every language it has.
   `src/theme.css` holds the colours. `src/App.jsx` orders the sections. Never
   write the practice's facts into the files, invent a qualification, price or
   result, or promise a treatment outcome.
4. Call `write_website_files` with every changed file complete, the
   `checkpoint` as `parentCheckpoint`, and a `message` in the owner's words
   ("Moved the reviews above the services"): the checkpoint is named by it.
   When the website changed since you read it, read it again and redo the
   change.
5. `list_website_checkpoints` lists the versions, newest first;
   `restore_website_checkpoint` goes back to one as a new checkpoint, keeping
   the later ones.

## When the answer is lost

`delivery_unknown` means the connection dropped after the request was sent,
so the change may have been saved. For a booking or a new patient, retry with
the same `requestId`: a record already saved is returned instead of written
twice. For anything else, read the record again before deciding to retry.

## Schedule and capacity

For workload over a period, call `get_clinic_schedule_summary` with ISO 8601
start/end values (seven days by default, never more than 31) and report the
aggregate counts, which include canceled appointments. Use
`get_clinic_availability` for open capacity: it shows unassigned slots inside
the practice's own online-booking window. Counts with `+` are lower bounds.

For a broad snapshot, call `list_clinics`, resolve the practice, then
`show_clinic_overview` with its id.

## Calendar blocks

Use `list_calendar_events` for closures, meetings, courses, days off and other
non-appointment blocks. It exposes only block times, all-day status, scope,
effect and type. Titles, notes and staff references are unavailable; do not
infer a reason or a person from a block.

## Unsupported requests

For a clinical request, state that the connector has no clinical records. For
a patient list, an export or a contact detail, state that it finds only a
patient the user names and never returns contact details. For deleting,
messaging, payments or settings other than the website's, point to the
DodoDentist app. A website builder draft cannot be published yet, in the app
either: say so rather than suggest a way. For diagnosis,
urgency, medication or treatment advice, do not invoke DodoDentist as a
clinical authority; follow the host's current safety guidance and make clear
that the connector provides no medical decision.

## Clinic sidebar

`open_dododentist` opens the clinic library without arguments. Select an owned
clinic to load `show_clinic_overview`, then inspect aggregate schedule counts,
availability counts or catalog prices. The selected clinic and view are shared
with the assistant. The sidebar itself changes nothing.
