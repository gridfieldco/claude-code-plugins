---
description: >
  Manage gridfield's Notion Lead Module CRM — create and update Plots, Contacts, and
  Organisations with correct relations. Use this skill whenever creating a new lead, plot,
  contact, or organisation in Notion, when ingesting email leads, when the user mentions
  the CRM or pipeline, or when looking up existing records. Also use when asked about the
  Leads Module structure, what fields exist, or how the databases relate to each other.
disable-model-invocation: true
---

# gridfield Leads Module

Three Notion databases under the "Leads Module" page. Plots are the unit of work —
everything else exists in relation to a plot.

## Database IDs

- **PLOTS**: `collection://71b273e9-6438-40b2-9519-dc1ccdcb6c01`
- **CONTACTS**: `collection://4f02df73-9f54-4d76-9f8c-963dd6eca8d7`
- **ORGANISATIONS**: `collection://9b5c15ff-1bb0-4038-acab-1768df941638`

## Relations

```
ORGANISATIONS ──1:many── CONTACTS ──1:many── PLOTS
                              │                  │
                              └── Organisation ──┘
```

- Plot → Primary Contact (limit 1) → one person who is the main point of contact
- Plot → Organisation (limit 1) → entity that owns/controls the plot
- Contact → Organisation (limit 1) → where this person works
- All relations are bidirectional (reverse fields auto-created by Notion)

## Creating records — always in this order

### 1. Check for duplicates first

Before creating anything, search Notion for existing records:

- **Organisation**: search by company name. Try variations (GmbH vs full name, abbreviations)
- **Contact**: search by email address (most reliable), then by name
- **Plot**: search by address or site name

If a match exists, use it. If unsure, show the candidate to the user and ask.

### 2. Create Organisation (if new)

```
Parent: data_source_id = 9b5c15ff-1bb0-4038-acab-1768df941638
Required: Name
Set if known: Industry, Size, Website, Address, Status (default: "Prospect")
```

Industry options: `Land / Developer`, `DSO / Grid operator`, `Municipality`, `Broker / Agency`, `Investor`, `Industrial owner`, `Other`

### 3. Create Contact (if new)

```
Parent: data_source_id = 4f02df73-9f54-4d76-9f8c-963dd6eca8d7
Required: Name
Set if known: Email, Phone, Title / Role, LinkedIn, Contact Type, Organisation (link to org page URL)
```

Contact Type options: `Landowner`, `Broker`, `Developer`, `DSO`, `Municipality`, `Investor`, `Other`

### 4. Create Plot

```
Parent: data_source_id = 71b273e9-6438-40b2-9519-dc1ccdcb6c01
Required: Name, Stage (default: "0 - Lead")
Link: Primary Contact (contact page URL), Organisation (org page URL)
```

**Name convention**: `[Metro/City] — [Distinguishing feature] — [Address]`

- Start with the nearest metro or city
- Middle: something that makes this plot instantly recognizable — a landmark, site name,
  notable characteristic (e.g. "40MW substation", "former Siemens factory", "Expo-Gelände")
- End with the street name (no postcode or city — keep it short)
- If address is unknown: `[Metro/City] — [whatever is known] — Unknown address [YYYY-MM-DD]`

Examples:
- `Frankfurt — Industriepark Höchst — Industrieparkstr.`
- `Berlin — 50ha brownfield near BER — Am Seegraben`
- `Munich — Unknown site — Unknown address 2026-03-13`

Key fields to set when available:

| Field | Notes |
|-------|-------|
| Address | Full street + postcode + city |
| GMaps Link | Google Maps URL |
| Plot Size (m²) | Number, leave empty if unknown — never estimate |
| Available MVA | Grid capacity in MVA |
| DSO Name | e.g. Bayernwerk Netz, Netze BW |
| Nearest Metro | `Frankfurt` `Berlin` `Munich` `Hamburg` `Cologne` `Leipzig` `Stuttgart` `Düsseldorf` `Other` |
| Distance to Metro (km) | Driving distance |
| Lead Source | `Inbound email` `Referral` `Cold outreach` `Broker` `Other` |
| Ownership Type | `Private` `Municipality` `Developer` `Corporate` `Unknown` |
| Zoning Type | `GI - Gewerbe/Industrie` `GE - Gewerbegebiet` `SO - Sondergebiet` `Mixed` `Unknown` |
| Power Feasibility | `Promising` `Unclear` `Unlikely` — human judgment, leave empty if unsure |
| Tier | `Tier 1` `Tier 2` `Tier 3` — overall priority, leave empty if not yet assessed |
| Last Updated | Set to today |
| Notes | Prepend new entries: `YYYY-MM-DD — [text]` |

**Metro Tier** is auto-calculated: <50km = Close, <150km = Mid, else Remote.

## Stage pipeline

`0 - Lead` → `1 - Tool Check` → `2 - Get-to-know` → `3 - Teaser Deck` → `4 - Follow-up` → `5 - Proposal / NBO`

Terminal: `6.1 - Deprioritised`, `6.2 - Disqualified`, `6.3 - Lost`

New leads always start at `0 - Lead`.

## When information is missing

Leave fields empty rather than guessing. Never estimate Plot Size, MVA, or distances.
For Bebauungsplan Status, write `Unknown — verify via Gemeinde [municipality]` if unknown.
Ask the user if something critical is ambiguous (which plot a contact belongs to, whether
an organisation already exists under a different name, etc.).

## Fields the AI must never set autonomously

- **Power Feasibility** — human judgment only. Leave empty when creating from email.
- **Tier** — human judgment only. Leave empty when creating from email.
- **Assigned To** — leave empty unless explicitly told who to assign.

These fields require context that is not available in a single email.

## Updating existing records

- **Notes**: prepend at the top with `YYYY-MM-DD — [update text]`
- **Last Updated**: set to today whenever making a meaningful update
- When changing Stage, add a note explaining why
