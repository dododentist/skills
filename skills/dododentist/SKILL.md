---
name: dododentist
description: Manage a dental practice with the `dododentist` CLI — list clinics, search and edit patient records, book, reschedule and cancel appointments, read recorded treatments, and create invoices and payments. Use when asked about DodoDentist clinics, patients, appointments, treatments, invoices or payments, about the DodoDentist REST API, or when a task involves the `dododentist` command or app.dododentist.com data.
license: Apache-2.0
compatibility: Requires Node.js 18+, network access, and a DodoDentist account (https://app.dododentist.com)
metadata:
  author: DodoDentist
  version: '1.0.0'
  repository: https://github.com/dododentist/skills
allowed-tools: Bash(dododentist:*) Bash(npx dododentist:*)
---

# DodoDentist CLI

`dododentist` reads and writes the practice-management records of a dental
clinic on [app.dododentist.com](https://app.dododentist.com): clinics,
patients, appointments, treatments, invoices and payments.

Run it as `dododentist` when installed globally
(`npm install -g dododentist`) or as `npx dododentist` otherwise.
`dododentist --help` and `dododentist <command> --help` are accurate and
self-sufficient; prefer them over memory when unsure of a flag.
`dododentist skills list` names the guides bundled with the installed version,
and `dododentist skills get dododentist` prints the copy of this guide
matching it.

## Handle patient data carefully

Patients, appointments, treatments, invoices and payments are health records
of identifiable people. Everything below acts on the real records of the
authenticated organization — there is no sandbox.

- Read no more than the task needs. Filter and project (`--fields`) instead of
  dumping whole collections.
- Do not copy patient records into files, issues, commits, chat transcripts or
  any third-party service without the practice explicitly asking for it.
- Confirm with the user before any `add`, `update` or `delete`. Deletions are
  immediate and permanent, and `delete` accepts several ids at once.
- Prefer a key scoped to the narrowest set of resources the task needs, and to
  a single clinic when the practice runs more than one.

## Setup

No account yet? Create one without leaving the terminal — the generated
password prints exactly once, the first clinic is created with the account,
and the session is stored so every other command works immediately:

```bash
dododentist signup --email dentist@example.com --json
```

Two ways to authenticate an existing account; both persist in
`~/.dododentist/`:

```bash
dododentist login               # interactive: choose browser or API key
dododentist login --browser     # opens app.dododentist.com to authorize; stores session tokens
dododentist login --with-key    # masked prompt for an API key secret; verified before storing
dododentist logout              # clears the stored session and any stored key
```

Headless agents skip `login` entirely: export
`DODODENTIST_API_KEY=<key secret>` and every command authenticates with it.
Keys are created by an organization administrator in the DodoDentist web app,
optionally locked to a single clinic; new keys default to read-only scopes.
Precedence: a stored browser session wins over a stored key, which wins over
the environment variable. The secret is never accepted as a command
argument — prompt or environment only.

Browser sessions refresh automatically. Without a terminal and without
credentials, commands fail fast with exit code 1 and instructions on stderr —
nothing ever opens a browser headlessly. A `401` means the session or key is
missing, expired, or wrong: run `dododentist login` (or fix
`DODODENTIST_API_KEY`) and retry.

## Scopes

A key carries a list of scopes and a request missing the one an endpoint
requires is refused with `403`:

`patients:read` · `patients:write` · `appointments:read` ·
`appointments:write` · `clinics:read` · `treatments:read` · `invoices:read` ·
`invoices:write` · `payments:read` · `payments:write`

There is no `clinics:write` and no `treatments:write`, which is why
`dododentist clinics` and `dododentist treatments` are read-only here: the
create/update/delete routes for those two exist only for signed-in staff in
the web app.

## Where to start

| You want                                  | Run                                             |
| ----------------------------------------- | ----------------------------------------------- |
| The clinics this credential can reach     | `dododentist clinics list`                      |
| Find a patient by email                   | `dododentist patients list --email …`            |
| A day's or week's schedule                | `dododentist appointments list --startTime … --endTime …` |
| One record in full                        | `dododentist patients read <patientId>`          |
| Book a visit                              | `dododentist appointments add …`                |
| Treatments recorded for a patient         | `dododentist treatments list --patientId …`      |
| Billing for a patient                     | `dododentist invoices list --patientId …`        |

Start with `dododentist clinics list`: almost every write needs a `clinicId`,
and every list narrows usefully with one. A key locked to a single clinic
supplies it implicitly, so `--clinicId` can be omitted there.

Every resource follows the same four-verb shape:

- `list` — filtered summary lines, or raw JSON with `--json`
- `get <id>` — raw JSON for one record
- `read <id>` — the same record formatted for a terminal
- `add` / `update <id>` / `delete <id...>` — where the scopes allow it

## Clinics (read-only)

```bash
dododentist clinics list                 # id, name, city/country
dododentist clinics get <clinicId>       # raw JSON for one clinic
dododentist clinics read <clinicId>      # formatted single-clinic view
```

## Patients

```bash
dododentist patients list --clinicId <id> -n 25
dododentist patients list --email patient@example.com
dododentist patients list --clinicId <id> --fields name,email,phone
dododentist patients list --_ids <id1>,<id2>
dododentist patients get <patientId>
dododentist patients read <patientId>

dododentist patients add --clinicId <id> --name "Ada Lovelace" --email ada@example.com --phone "+15555550100"
dododentist patients update <patientId> --phone "+15555550111"
dododentist patients delete <patientId>          # variadic: delete <id1> <id2> …
```

`list` sorts by `name`, `lastEditTime` or `_id` only — any other
`--sortField` is ignored server-side rather than rejected, so results come
back unsorted instead of erroring. Deleting a patient also removes their
attached anamnesis records, signatures and uploaded documents; it cannot be
undone.

## Appointments

```bash
dododentist appointments list --clinicId <id> \
  --startTime 2026-09-01T00:00:00.000Z --endTime 2026-09-08T00:00:00.000Z
dododentist appointments list --patientId <id> --sortField startTime --sortDirection ASC
dododentist appointments get <appointmentId>
dododentist appointments read <appointmentId>

dododentist appointments add --clinicId <id> --patientId <id> \
  --startTime 2026-09-01T09:00:00.000Z --endTime 2026-09-01T09:30:00.000Z
dododentist appointments update <appointmentId> --startTime 2026-09-02T09:00:00.000Z
dododentist appointments update <appointmentId> --status CANCELED
dododentist appointments delete <appointmentId>
```

Times are ISO 8601 instants. `--startTime` matches appointments starting at or
after that instant and `--endTime` those ending strictly before it, so a range
needs both. Statuses are `REQUESTED`, `SCHEDULED` (the default),
`PATIENT_ARRIVED`, `NO_SHOW` and `CANCELED`.

Cancelling is usually `update --status CANCELED`, which keeps the record and
its history; `delete` removes it outright and drops its scheduled reminders.
Ask which one the practice means. Booking a new appointment can be refused
with `402` when the clinic is at its plan's monthly appointment limit —
editing or cancelling an existing one never is.

## Treatments (read-only)

```bash
dododentist treatments list --patientId <id>
dododentist treatments list --clinicId <id> --startTime 2026-01-01T00:00:00.000Z
dododentist treatments get <treatmentId>
dododentist treatments read <treatmentId>
```

Treatments are clinical records. Summarize them for the practice's own staff;
do not restate them anywhere else, and never present them as medical advice.

## Invoices and payments

```bash
dododentist invoices list --patientId <id>
dododentist invoices get <invoiceId>
dododentist invoices read <invoiceId>
dododentist invoices add --clinicId <id> --patientId <id> --status DRAFT
dododentist invoices update <invoiceId> --status PAID
dododentist invoices delete <invoiceId>

dododentist payments list --clinicId <id> --startTime 2026-08-01T00:00:00.000Z
dododentist payments get <paymentId>
dododentist payments read <paymentId>
dododentist payments add --clinicId <id> --patientId <id> --amount 120
dododentist payments update <paymentId> --note "paid in cash"
dododentist payments delete <paymentId>
```

Invoice statuses are `DRAFT`, `PENDING`, `PARTIALLY_PAID`, `PAID`, `OVERDUE`,
`REFUNDED` and `UNCOLLECTIBLE`. Money lives in `lines` (or the legacy
`procedures`) rather than one scalar field, so build those with `--body`:

```bash
dododentist invoices add --clinicId <id> --patientId <id> \
  --body '{"lines":[{"name":"Cleaning","price":80,"quantity":1}]}'
```

A payment may instead carry a flat `--amount`, which is what the card-terminal
flow writes. A manually recorded payment has no `status` at all and still
counts as collected; `SUCCEEDED`, `FAILED` and `CANCELED` come from the card
flow.

## Output discipline

Every data subcommand accepts `--json`: real, parseable JSON on stdout — no
colors, no ANSI escapes. Without the flag, output is for human reading.
Mutations with `--json` print a small result object
(`{ "ok": true, "id": … }`). Errors always go to stderr with exit code 1;
in `--json` mode the error is a single JSON line
(`{"error":{"message":…,"status":…}}`), so stdout stays parseable.

`dododentist schema` prints the whole command tree (options, arguments,
subcommands) as JSON — discover capabilities from it instead of scraping help
text:

```bash
dododentist schema                    # the full command tree as JSON
dododentist schema patients list      # one subtree only
```

When exploring, project instead of dumping — pipe through `jq` and keep only
the fields you need:

```bash
dododentist patients list --clinicId <id> --json | jq '.[] | {id: ._id, name}'
```

## Configuration

`DODODENTIST_API_KEY` supplies an API key secret for headless auth (see
Setup). `DODODENTIST_API_URL` overrides the API endpoint — only needed when
testing against a non-production deployment.

## Errors → remediation

| Symptom                                | Fix                                                                   |
| -------------------------------------- | --------------------------------------------------------------------- |
| `401` on any command                   | `dododentist login` (or set `DODODENTIST_API_KEY`), then retry         |
| `403` mentioning a scope               | Key scopes too narrow — widen them in the DodoDentist web app, or log in |
| `missing clinicId` on a write          | Pass `--clinicId`, or use a key locked to one clinic                   |
| `404` on a record you can see elsewhere | It belongs to another organization, or to a clinic outside the key's scope |
| `402` when booking                     | The clinic hit its plan's monthly appointment limit — upgrade, or edit an existing appointment instead |
| `429` on a burst of calls              | Per-key rate limit — slow down and batch with filters instead of looping |
| `list` returns unsorted results        | `--sortField` was not one this endpoint allows; it is ignored, not rejected |
