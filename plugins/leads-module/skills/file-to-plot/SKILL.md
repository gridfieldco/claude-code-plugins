---
description: >
  File documents into the correct gridfield plot folder on the local Google Drive-synced
  directory. Use whenever saving a document to a plot's folder, creating a new plot folder,
  looking up where documents are stored for a plot, or updating a folder's index. Also use
  when processing emails with attachments or when any workflow produces a document that
  belongs to a specific plot.
disable-model-invocation: true
---

# File to Plot

Organize documents for gridfield plots in a local Google Drive-synced folder. Each plot
gets one flat folder. Each folder has an `index.md` describing every file inside it.

## Setup

The Drive-synced folder must be mounted via `request_cowork_directory`. If not mounted, ask
the user to select it. Notion MCP is required for plot lookups.

## Folder naming

```
[Nearest Metro] — [Plot Name] — [Source]
```

- **Metro**: from Notion `Nearest Metro`. Default: `Unknown Metro`
- **Name**: from Notion `Name` field (typically address or site identifier)
- **Source**: Organisation name > Primary Contact name > Lead Source value > `Unknown source`
- Replace filesystem-illegal characters (`/\:*?"<>|`) with `-`

Examples: `Frankfurt — Industriepark Höchst — Müller Immobilien GmbH`,
`Berlin — Siemensstadt 42 — Broker Schmidt`

## Filing a document

1. **Identify the plot** — from user context, conversation history, or Notion search
2. **Fetch plot properties** — Name, Nearest Metro, Primary Contact, Organisation, Lead Source, Drive Folder URL
3. **Find or create folder** — list base directory, fuzzy-match existing folders on metro + name. Create with `mkdir -p` if missing
4. **Place the file** — keep descriptive filenames as-is. Rename generic ones (`document.pdf`, `image001.png`) to `[Type] — [Description].[ext]`
5. **Update index.md** — analyze the document and add an entry. If the file doesn't exist yet, create it with the header line first. Insert the new `## [filename]` block immediately after the `# Document Index` header line, before any existing entries, so the newest document is always at the top.

```markdown
# Document Index — [Plot Name]

## [new-file.pdf]            <-- new entry goes here, right after the header
- **Added:** YYYY-MM-DD
- **Type:** Exposé | Site plan | Grid confirmation | Zoning extract | Energy data | Correspondence | Research | Photo | Map | Contract | Other
- **Summary:** 2-4 sentences on contents, key figures, dates.
- **Keywords:** searchable terms including German planning/energy terms (Bebauungsplan, Gewerbegebiet, Netzanschluss), names, MVA values, addresses

---

## [older-file.pdf]          <-- previous entries follow below
- **Added:** ...
...
```

6. **Update Notion** — if `Drive Folder URL` is empty, ask user to copy the Google Drive link once it syncs, then set it. Update `Last Updated` to today.

## Looking up documents

1. Search Notion for the plot
2. Share `Drive Folder URL` if set
3. Find local folder, read `index.md`, present contents
4. If no index.md exists but files do, generate one by analyzing all files

## Edge cases

- **No metro yet**: use `Unknown Metro`, rename folder later when set
- **Multiple files at once**: file each, batch index.md into one write
- **Duplicate filename**: ask user to overwrite or rename
- **Folder not found**: list all folders, try to match, create new if no match
