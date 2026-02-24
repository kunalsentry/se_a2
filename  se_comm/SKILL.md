| name | description | 
| --- | --- |
| se-comm | Consolidate all communications from multiple sources, analyze & extract all immediate tasks or notatble events for sales engineers. |


# Sales Engineer Communication
Consolidate all communications from a given time period across multiple sources relating to existing & new sales opportunities with prospect customers. Extract immediate tasks for sales engineer relating customer queries/issues, identify any potential updates to the sales opportunity & highlight notable information. 

## Why this skill exists
Sales Engineer work across multiple surfaces in multiple tools. Consolidating and tracking all updates is tedious - the context is scattered. This skill does the gathering & synthesis to give the Sales Engineer a clear understanding of any communication or sales opportunity updates. It can also assist in update the relevant sources of truth ( but only if user explicitly agrees ) . 


## Inputs

The user may specify a single or multiple customers to filter communication for, if none is provided then no customer filtering should be applied.

A time period may also be specified, if not the default is the past 24 hours. 

**IMPORTANT** : DO NOT make any modifications to any data sources through Steps 1-4. Afterwards the user may prompt for changes, but never modify unless explicitly prompted by the user. 

## Step 1 : Retrieve all communication

Search the below sources in parallel for all communication & information within the time period. 

If a customer(s) is specified, filter for activity relating to that customer. 

**IMPORTANT** : Be conservative with filtering, only exclude if you are 99% sure it isn't relevant to the user's query. Otherwise, include it. 


### Slack 

Search for all public/private messages & notifications received in the time period. 

**IMPPORTANT** : Filter out any activity that clearly isn't related to a customer's interest in a technical product or a sales opportunity. If unsure, do not include activity but indicate to the user after immediate skill execution. They can query/prompt for additional context if they desire.

For each message, read the tread or any other linked message/thread to get the full context. 

If notified on any activity in the time period(replies, creation, added, mentions, ect. ), query for any relevant information to get the full context.

**IMPORTANT** : Pay special attention to any activity in external customer channels ( channel name format `ext-<customer name>-...` ) or internal customer channels (channel name format `opp-<customer name>-...`/`proj-<customer name>-...`). Activity here might be the most relevant to ongoing sales opportunities. 



### Gmail 

Retrieve all emails received & sent in the time period. 

Filter out any emails that are personal to the user and do not relate to sales opportunities or customers interest in techincal products. 

**IMPORTANT NOTE** : If unsure if email should be, don't include but indicate to the user after immediate execute of skill. They can query/prompt for additional context if they desire.

There may be automated emails that are not direct interations with customers, but from tools relating to meetings and sales opportunity management like Gong, Salesforce, and support tickets. 



## Step 2 : Retrieve existing sales opportunities

If not previously specified, ask the user for their currently active sales opportunities. This will usually be a  Notion page containing a database where each entry will be an active/previous sales opportunity. 

**IMPORTANT** : Enumerate ALL opportunities from the database — do not use a single semantic search call, as it will silently drop records.

Use `notion-search` with cursor-based pagination, iterating until no `next_cursor` is returned, to retrieve every page in the database: ``` notion-search: query = "*", data_source_url = <collection url>, cursor = <next_cursor until exhausted> ``` 

**IMPORTANT** : Filter the full result set for opportunities that are active, meaning NOT in terminal states (WIN / Lost / Transferred) 

Per opportunity, make **both** of the following calls before proceeding to anything else: 
``` 
notion-fetch: page_id = <opportunity page id>, include_discussions: true notion-get-comments: page_id = <opportunity page id> 
``` 
Note each opportunity's hallmark technical features, as well as customer information listed on the associated entry's page & comments. Analyze any & all of updates made.

Correlate the existing sales opportunity information with the activity information retrieved in Step 1. 

**IMPORTANT** : There may be activity that doesn't have an existing sales opportunity. This is fine, in fact this may warrant the creation of a new sales opportunity in the provided database. DO NOT modify that page. The user may ask for assistance in creating a new sales opportunity after this skill runs. 

## Step 3 : Hex (for usage data)


If available, check trial usage data for any customers that has communication/activity in Step 1 & 2. 

**IMPORTANT** : Only query for customers that indicating existing sentry usage from the information gathered in the previous steps. 

Even if there's no usage yet, that itself is a signal worth noting (e.g. "trial hasn't started yet").


## Step 4: Analyze & Extract for the User

Consolidate the activity per customer/sales opportunity. Extract relevant information & tasks the user should know. 

The following are notable: 

 - Technical questions or blockers, especially ones the user is explicitly responsible for 
 - Explicit action items for the user and whether they've been completed or are ongoing
 - Questions from the customer, and if they've been answered
 - Legal/security/procurement updates
 - Automated messages from other tools ( like Gong, Salesforce, Momentum ) relating to sales opportunities & meetings. 
 - Scheduling requests, or any other signals of engagement 
 - **ONLY FOR EXTERNAL CUSTOMER ACTIVITY** : Any negative tone and responsiveness of messages that may increase risk of successful sales opportunity.

Synthesize findings into a structured update for the user. The format should be:

```
<customer> : <New/SALES_OPPORTUNITY_STAGE, depends on if there is existing sales opportunity>

TLDR : <bullet list of either the user owes (only if any exists) or things to know ,each 10 words max and 3 bullets max total>

Todo : <more detailed list bullet list of what we the user owes (only if any exists), 7 bullets max>

Things to know : <more detailed bullet list of what we the user should know about this customer/activity(only if any exists),7 bullets max>

```


### Writing guidelines

- The reader (a sales engineer) wants to know "what do I need to know or do" from reading the TLDR
- Flag Risks, anything that could jepordize the customer's engagement from a technical standpoint.

## Step 4: Output findings

Output the findings.

**IMPORTANT** : You can add or modify to the sales opportunity database referenced in Step 2 , but explicitly ask and don't make any changes without user first confirming the specific changes.

 
