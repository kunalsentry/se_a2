---
name: se-comm
description: Consolidate all communications from multiple sources, analyze & extract all immediate tasks or notatble events for sales engineers. |
---

# Sales Engineer Communication
Consolidate all communications from a given time period across multiple sources relating to existing & new sales opportunities with prospect customers. Extract immediate tasks for sales engineer relating customer queries/issues, identify any potential updates to the sales opportunity & highlight notable information. 

## Why this skill exists
Sales Engineer work across multiple surfaces in multiple tools. Consolidating and tracking all updates is tedious - the context is scattered. This skill does the gathering & synthesis to give the Sales Engineer a clear understanding of any communication or sales opportunity updates. It can also invoke changes to the sales opportunity source of truth ( but only if user explicitly agrees ).

## When to invoke 
Any query relating to action items/tasks, sales opportunity activity, recent updates

## Inputs

The user may specify a single or multiple customers to filter communication for, if none is provided then no customer filtering should be applied.

A time period may also be specified, if not the default is the past 24 hours. 

**IMPORTANT** : This skill directly DOES NOT make any modifications to any data sources. Afterwards the user may prompt for changes or invoke other skills, but never modify unless explicitly prompted by the user. 

## Step 1 : Retrieve all Communication

Search the below sources in parallel for all communication & information within the time period. 

If a customer(s) is specified, filter for activity relating to that customer. 

**IMPORTANT** : Be conservative with filtering, only exclude if you are 99% sure it isn't relevant to the user's query. Otherwise, include it. 


### Slack 

Search for all public/private messages & notifications received in the time period. 

**IMPORTANT** : Filter out any activity that clearly isn't related to a customer's interest in a technical product or a sales opportunity. If unsure, do not include activity but indicate to the user after immediate skill execution. They can query/prompt for additional context if they desire.

For each message, read the tread or any other linked message/thread to get the full context. 

If notified on any activity in the time period(replies, creation, added, mentions, ect. ), query for any relevant information to get the full context.

**IMPORTANT** : Pay special attention to any activity in external customer channels ( channel name format `ext-<customer name>-...` ) or internal customer channels (channel name format `opp-<customer name>-...`/`proj-<customer name>-...`). Activity here might be the most relevant to ongoing sales opportunities. 


### Gmail 

Retrieve all emails received & sent in the time period. 

Filter out any emails that are personal to the user and do not relate to sales opportunities or customers interest in techincal products. 

**IMPORTANT NOTE** : If unsure if email should be, don't include but indicate to the user after immediate execute of skill. They can query/prompt for additional context if they desire.

There may be automated emails that are not direct interations with customers, but from tools relating to meetings and sales opportunity management like Gong, Salesforce, and support tickets. 


## Step 2 : Retrieve existing sales opportunities

Retrieve all active sales opportunities of this user. First search through skills for ones that can assist, if none are present then query the user. 

Correlate the existing sales opportunity information with the activity information retrieved in Step 1. 

**IMPORTANT** : There may be activity that doesn't have an existing sales opportunity. This is fine, in fact this may warrant the creation of a new sales opportunity in the provided database. DO NOT modify that page. The user may ask for assistance in creating a new sales opportunity after this skill runs. 

## Step 3 : Additional Sources Querying

### Hex (for usage data)

If available, check trial usage data for any customers that has communication/activity in Step 1 & 2. 

**IMPORTANT** : Only query for customers that indicating existing sentry usage from the information gathered in the previous steps. 

Even if there's no usage yet, that itself is a signal worth noting (e.g. "trial hasn't started yet").

### Gong ( for meeting/call information)
A local Gong MCP server _might_ be running that can be used to query call transcripts/outlines/ect. 

If available, and if any previously gathered notifications/activity/information indicating a meeting relating to a sales opportunity has occured, query Gong for any information that could be relevant for a sales engineer. For example: technical requirements/interested features, customer questions/blockers, customer sentiment, ect.) 

## Step 4: Analyze & Extract for the User

Consolidate the activity per customer/sales opportunity. Extract relevant information & tasks the user should know. 

The following are high priority: 

 - Technical questions or blockers that are unanswered/unresolved
 - Explicit action items for the user that have NOT been completed
 - Scheduling requests, or any other signals of engagement 
 - **ONLY FOR EXTERNAL CUSTOMER ACTIVITY** : Any negative tone and responsiveness of messages that may increase risk of successful sales opportunity.

The following are notable but of lesser priority: 
 - Technical questions/blockers and action items the user is responsible for that HAVE been answered
 - Legal/security/procurement updates
 - Automated messages from other tools ( like Gong, Salesforce, Momentum ) relating to sales opportunities & meetings. 


Synthesize findings into a structured update for the user. The format should be:

```
<customer> : <New/SALES_OPPORTUNITY_STAGE, depends on if there is existing sales opportunity>

TLDR : <bullet list of HIGH PRIORITY tasks the user owes (only if any exists) or things to know ,each 10 words max >

Other Things : <detailed list bullet list of LESSER PRIORITY information & tasks relating to the user 7 bullets max>


```


### Writing guidelines

- The reader (a sales engineer) wants to know "what do I need to know or do" from reading the TLDR
- Flag Risks, anything that could jepordize the customer's engagement from a technical standpoint.

## Step 5: Output findings to Slack Canvas

Output the findings both to the user in chat AND to a private Slack canvas for persistent tracking.

### Canvas workflow

1. **Search for existing canvas** : Search Slack for a canvas titled `"SE Action Items"` created by the current user (use `type:canvases creator:@me` file search).
2. **If an existing canvas is found** : Read the canvas content. Note any items that have already been checked off — these should be preserved as-is in the new canvas (do not uncheck them).
3. **Create/replace the canvas** : Create a new Slack canvas titled `"SE Action Items"`. The canvas API does not support editing, so each run creates a fresh canvas that supersedes the previous one.

### Canvas content format

Structure the canvas as follows:

```
# SE Action Items
*Last updated: <current date & time>*

## <Customer Name> — <New/SALES_OPPORTUNITY_STAGE>

### Action Items
- [ ] <HIGH PRIORITY action item 1>
- [ ] <HIGH PRIORITY action item 2>
...

### Other Notable
- <LESSER PRIORITY item 1>
- <LESSER PRIORITY item 2>
...

(repeat per customer)

---
## Completed (from previous run)
- [x] <any previously checked-off items carried over>
```

**IMPORTANT formatting rules:**
- Every HIGH PRIORITY action item from the TLDR MUST be a checkbox (`- [ ]`)
- Lesser priority / informational items are regular bullet points (no checkbox)
- If a previous canvas existed, any items marked `- [x]` (checked) should be preserved in a "Completed" section at the bottom
- New items that duplicate a completed item should NOT be re-added as unchecked
- Keep each checkbox item concise (under 15 words) but specific enough to act on

### After canvas creation

- Share the canvas link with the user in the chat output
- DM the user a link to the channel
- Also display the structured per-customer summary (from Step 3's format) directly in chat for immediate visibility
 
