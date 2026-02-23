| name | description | 
| --- | --- |
| se-todos | Extract all communications from multiple sources and summarize action items for sales engineer. |


# Sales Engineer Todo
Consolidate all recent communications from multiple data sources relating to existing/new sales opportunities with prospect customers. Extract immediate tasks for sales engineer relating customer queries/issues or sales opportunity updates, and summarize key information relating to customer/sales opportunity. 

## Why this skill exists
Sales Engineer work across multiple surfaces in multiple mediums. Consolidating and tracking all updates is tedious - the context is scattered & so is the tooling. This skill does the gathering and synthesis to give the Sales Engineer a clear understanding of any recent communication or sales opportunity updates, and can assist in update the relevant sources of truth ( but only if user explicitly agrees ) . 


## Inputs

The user may specify a single/multiple customer or time period. If not specified, retrieve all communication/information from the past day. 

## Step 1 : Retrieve existing sales opportunities

If not previously specified, ask the user for the Notion page containing a database of all their opportunities. 

**Enumerate ALL opportunities from the database — do not use a single semantic search call, as it will silently drop records.** 

Use `notion-search` with cursor-based pagination, iterating until no `next_cursor` is returned, to retrieve every page in the database: ``` notion-search: query = "*", data_source_url = <collection url>, cursor = <next_cursor until exhausted> ``` 

Filter the full result set for opportunities that are: 
- Not in terminal states (WIN / Lost / Transferred) 

Per opportunity, make **both** of the following calls before proceeding to anything else: 
``` 
notion-fetch: page_id = <opportunity page id>, include_discussions: true notion-get-comments: page_id = <opportunity page id> 
``` 
Note each opportunity's hallmark technical features & all of the updates made. Specifically note the date & content of the last update made (LUM) — this will be relevant when querying customer/internal communications.

## Step 2 : Search connected data sources

Search these sources in parallel for all activity within the time period. Only extract communication/information relating to sales opportunities and directly involving/relevant to the user invoking the skill. 

### Slack 

Search for all messages & notifications received in the time period. For each message read the tread to get the full picture. 

Use the following tools: 
```
slack_search_public_and_private: 
slack_read_channel: 
slack_read_thread: 
```

Pay special attention to:
- Action items and whether they've been completed
- Questions from the customer (especially unanswered ones)
- Legal/security/procurement updates
- Technical questions or blockers
- Scheduling requests (signal of engagement)
- Tone and responsiveness of customer messages
- Any internal slack threads not external customer facing but relevant 

### Gmail 

Retrieve all emails received in the time period. Focus on emails with direct communication with customers, as well as automated ones sent by other tooling relating to customer meetings ( Gong ) , technical questions ( support ), or the opportunity itself ( Salesforce opportunity updates). 

```
search_gmail_messages: q = "<account name>"
google_drive_search: api_query = "fullText contains '<account name>'"
```

### Hex (for usage data)

If available, check trial usage data for all customers that have communications indicating existing sentry usage. Even if there's no usage yet, that itself is a signal worth noting (e.g. "trial hasn't started yet").

## Step 3: Build the todos

Synthesize findings into a structured update for the user. The format should be:

```
Relevant communication in last <N> <days/weeks>

<customer> : <New/Existing>

<UPDATE STAGE : _ , only if indicated through tooling > 

Communication <Reverse-chronological timeline of key events>: 
- <Who initiated> Questions asked (especially technical ones shows real evaluation) / Action items completed or still open / Information shared <link> 

Todo : <bullet list of what we the user owes,each 10 words max>

```


### Writing guidelines

- Lead with the overall sentiment — the reader (a manager) wants to know "should I be worried?" within the first sentence
- Be specific with dates and names — "Al met with legal on Feb 17" not "legal is being reviewed"
- Call out unanswered questions explicitly — these are action items hiding in plain sight
- Note engagement velocity — how quickly the customer responds is one of the strongest signals of deal health
- Flag Risks — if there's a close date or renewal deadline, do the math on whether the POC timeline fits

## Step 4: Output 

Output all of the todos. 

You can add or modify the database of the active opps, but explicitly ask and don't make any changes without user first confirming. 
