---
description: >
  Batch-process the gridfield inbox for new leads. Fetches recent unprocessed emails,
  groups them by conversation, and spawns parallel subagents to ingest each conversation.
  Use when the user wants to process the mailbox, check for new leads, or run the inbox
  pipeline.
---

# Process Inbox

Batch-process unprocessed emails from the gridfield mailbox, group by conversation,
and ingest each conversation in parallel for lead detection and CRM creation.

## Step 1 — Fetch unprocessed emails

Use the Gmail MCP tool to search for the last 30 emails that do **not** have the
`aicheck` label:

```
search_gmail_messages: "in:inbox -label:aicheck newer_than:30d"
```

If zero results, report "No unprocessed emails found" and stop.

## Step 2 — Group by conversation

Use a **quick subagent** (model: haiku) to group the fetched emails by conversation.
Pass it the list of emails (message IDs, subjects, senders, dates) and ask it to return
a JSON array of conversation groups:

```json
[
  {
    "conversation_id": "thread_or_subject_key",
    "subject": "representative subject line",
    "message_ids": ["msg_id_1", "msg_id_2"],
    "participants": ["sender1@example.com", "sender2@example.com"]
  }
]
```

Grouping rules:

- Emails sharing a Gmail thread ID belong to the same conversation
- If thread IDs are unavailable, group by normalized subject (strip Re:/Fwd:/AW:/WG: prefixes)
- Single emails are a conversation of one

## Step 3 — Filter out internal-only conversations

Skip conversations where **all** participants are internal gridfield team members:

- `@gridfield.co`

A conversation with at least one external participant should still be processed.
After filtering, label the skipped internal-only emails with `aicheck` so they are
not fetched again.

## Step 4 — Load skills into context

Before spawning subagents, read the following skill files so their full content can be
embedded in each subagent prompt (subagents cannot invoke slash commands):

- `plugins/leads-module/skills/lead-crm/SKILL.md` — CRM creation rules
- `plugins/leads-module/skills/file-to-plot/SKILL.md` — document filing rules

Use the Read tool to load both files. Store their contents as `{lead_crm_skill}` and
`{file_to_plot_skill}` for insertion into the subagent prompt below.

## Step 5 — Parallel ingestion

For each remaining conversation group, spawn a **subagent in parallel** with the
prompt below. Do NOT process conversations sequentially — launch all subagents at once.

### Subagent prompt template

```
You are processing an email conversation for the gridfield CRM lead pipeline.

Conversation subject: {subject}
Message IDs: {message_ids}

## Reference skills

### CRM rules (lead-crm)
{lead_crm_skill}

### Document filing rules (file-to-plot)
{file_to_plot_skill}

## Your task

1. Fetch the full content of each message using get_gmail_message_content or
   get_gmail_messages_content_batch.

2. Extract from the emails: sender name, sender email, organisation (from email
   domain or signature block), subject, body text, any mentioned addresses/locations/
   plot details, attachments list, phone numbers, titles/roles.

3. Classify the conversation using the rules below.

4. If it's a lead, create CRM records following the CRM rules above.

5. After processing (created, updated, or skipped), label ALL message IDs with
   "aicheck" using modify_gmail_message_labels.


**IS a lead if it mentions:**
- A specific site, plot, or land parcel (address, location, or area description)
- Grid connection capacity or substation proximity
- Available land with potential for data center / industrial use
- An offer or introduction from a broker, landowner, or developer

**NOT a lead if:**
- Newsletter, marketing email, or automated notification
- Existing business relationship with no new site
- Purely administrative (invoices, scheduling, signatures)
- No identifiable site or plot being offered/discussed

If uncertain, classify as "Not a lead". Do not create speculative leads.

## If it's a new lead — CRM creation

Follow the CRM rules above for all CRM operations. Key points:

**NEVER set from email:** Power Feasibility, Tier, Assigned To — leave empty.

**Plot creation threshold:** only create a plot if a specific site/location is mentioned.
Do NOT create a plot for vague "we have land available" emails with no details.
In that case, still create Organisation and Contact if identifiable, and add a Note
that a vague land offer was received.

## If it's an existing lead update

Search Notion for the matching plot. Update its Notes field:
"YYYY-MM-DD — Email from [sender]: [1-line summary]"
Set Last Updated to today.

## Attachments

Download and analyse attachments. If those files are relevant sources of useful
information, process them following the document filing rules above.

## Return format

Result: created | updated | skipped
Reason: [why this decision was made]
Plot: [plot name if created/updated, or "n/a"]
Contact: [contact name if created, or "existing: [name]", or "n/a"]
Organisation: [org name if created, or "existing: [name]", or "n/a"]
Messages processed: [count]
Attachments noted: [list or "none"]
```

## Step 6 — Summary report

After all subagents complete, compile and present a summary table:

```
| # | Subject                          | Result                        |
|---|----------------------------------|-------------------------------|
| 1 | RE: Grundstück Frankfurt Höchst  | Created: Frankfurt — ...      |
| 2 | Meeting notes                    | Skipped: no lead content      |
| 3 | Fwd: Exposé Industriepark        | Created: Munich — ...         |
```

Report totals: leads created, leads updated, skipped, errors.
