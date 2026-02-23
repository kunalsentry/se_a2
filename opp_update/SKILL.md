| name | description | 
| --- | --- |
| opp-status-update | Generates an update to an active customer sales opportunity. Focus on what the most recent customer interaction & internal conversation has been since the last update posted in the opportunity. Unless specified, run on all opportunities that have some activity in the last month. |


# Opportunity Status Update
Generate an update for a single/multiple active sales opportunity by searching connected data sources, then posting it as a comment on a notion page. 

## Why this skill exists
Sales Engineers need to keep their managers informed on sales opportunity progress, but assembling these updates is tedious - the context is scattered across Slack channels, email threads, Notion pages, and usage dashboards. This skill does the agathering and synthesis so the user can focus on reviewing and acting on the information. 

## Inputs

The user may specify a single/multiple customer or time period. If not specified, perform for all opportunities with activity in the last month. 

## Step 1 : Retrieve Recent Sales Opportunity Data per Customer 

If not previously specified, ask the user for the Notion page containing a database of all their active opportunities. 

**Enumerate ALL opportunities from the database — do not use a single semantic search call, as it will silently drop records.** 

Use `notion-search` with cursor-based pagination, iterating until no `next_cursor` is returned, to retrieve every page in the database: ``` notion-search: query = "*", data_source_url = <collection url>, cursor = <next_cursor until exhausted> ``` 

Filter the full result set for opportunities that are: 
- Not in terminal states (WIN / Lost / Transferred) 
- Have a `Last edited time` within the specified time period (default: last 30 days) 

Per opportunity, make **both** of the following calls before proceeding to anything else: 
``` 
notion-fetch: page_id = <opportunity page id>, include_discussions: true notion-get-comments: page_id = <opportunity page id> 
``` 
Note each opportunity's hallmark technical features & all of the updates made. Specifically note the date & content of the last update made (LUM) — this will be relevant when querying customer/internal communications.

## Step 2 : Search connected data sources

Search these sources in parallel for activity within the time period. Filter for any new information/communication not previously noted in the opportunity's updates. The goal is to build a timeline of what happened — meetings, messages, decisions, blockers - since the LUM .

### Notion (already fetched)

Review the page content from Step 1 for existing context. 

### Slack 

Search for the account in Slack. There are usually two relevant channels:

- **Internal opp channel** (e.g. `#opp-<customer>-enterprise`) — internal discussions
  about strategy, pricing, legal, next steps
- **External shared channel** (e.g. `#ext-<customer>-sentry`) — direct communication with the customer

For each channel found:
1. Read recent messages to get the full picture. 
2. Read threads on important messages — thread replies often contain the most important updates (e.g. legal responses, technical answers, action item completions)

```
slack_search_public_and_private: query = "<account> after:<start_date>"
slack_read_channel: channel_id = <each relevant channel>
slack_read_thread: channel_id + message_ts for messages with replies
```

Pay special attention to:
- Action items and whether they've been completed
- Questions from the customer (especially unanswered ones)
- Legal/security/procurement updates
- Technical questions or blockers
- Scheduling requests (signal of engagement)
- Tone and responsiveness of customer messages
- Any internal slack threads not external customer facing but relevant 

### Gmail and Google Drive (if relevant signals appear)

Search if Slack conversations reference email threads or shared documents. These often contain call summaries, legal documents, or requirements docs.

```
search_gmail_messages: q = "<account name>"
google_drive_search: api_query = "fullText contains '<account name>'"
```

### Hex (for usage data)

If available, check trial usage data. Even if there's no usage yet, that itself is a
signal worth noting (e.g. "trial hasn't started yet").

## Step 3: Build the status update

Synthesize findings into a structured update. The format should be:

```
<date> — Engagement & Sentiment (last <N> <days/weeks>)

TLDR sentiment: <✅/‼️> <15 words max. Be direct: is this trending well or not?>

<Reverse-chronological timeline of key events (most recent first), each starting with the date.
Focus on engagement signals, each max 15 words:>
- Who initiated contact and how quickly people responded
- Meetings that happened and key outcomes
- Questions asked (especially technical ones — shows real evaluation)
- Legal/procurement progress
- Action items completed or still open

Open items (<customer> side): <bullet list of what the customer owes, each 10 words max>

Open items (Sentry side): <bullet list of what we owe,each 10 words max>

Risks: <what could derail or delay this deal>
```

### Writing guidelines

- Lead with the overall sentiment — the reader (a manager) wants to know "should I
  be worried?" within the first sentence
- Be specific with dates and names — "Al met with legal on Feb 17" not "legal is
  being reviewed"
- Call out unanswered questions explicitly — these are action items hiding in plain
  sight
- Note engagement velocity — how quickly the customer responds is one of the
  strongest signals of deal health
- Flag timeline risks — if there's a close date or renewal deadline, do the math on
  whether the POC timeline fits

## Step 4: Output 

Output each opportunities status update for the user to view/modify/ask followup questions about. 

DO NOT POST OR MODIFY ANY EXISTING DOCUMENT unless the user explicitly agrees to. Air on the side of caution and ask before making any changes(s). 



