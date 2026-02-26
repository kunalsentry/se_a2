---
name: opp-management
description: Manage access of master database housing all sales opportunities for Sales Engineers. Guildelines on access & moidification functionality.
---

# Sentry Sales Opportunity Management
Assists in accessing central database of Sentry sales opportunities specified by the user, who is a Sales Engineer. Standardizes creation of opportunity, both in when it's warranted & formats to adhere to. 

## Why this is important
Sentry Sales Engineers have their own unique way to manage new, active & completed sales opportunities. As automated tooling increased, this skill should the sole interacting layer between these workflows and the source of truth the users rely on. 

## When to invoke 
- Query : When a single/multiple sales opportunity(s)'s core information is required
- Create : When a new sales opportunity needs to be created
- Update : When there's an update in the sales opportunity that needs to be documented

## Before Starting

### Sales Opportunity Source of Truth
Ensure that the user has specified where their sales opportunities are tracked. You can assume the source will be a single notion document with a database, where every entry is a sales opportunity. 

**IMPORTANT** : If the structure of the data is unclear, ask the user followups to get clarity. 

**IMPORTANT** : For notion databases, enumerate ALL opportunities from the database — do not use a single semantic search call, as it will silently drop records.

Use `notion-search` with cursor-based pagination, iterating until no `next_cursor` is returned, to retrieve every page in the database: ``` notion-search: query = "*", data_source_url = <collection url>, cursor = <next_cursor until exhausted> ``` 

Per opportunity, make **both** of the following calls before proceeding to anything else: 

``` 
notion-fetch: page_id = <opportunity page id>, include_discussions: true notion-get-comments: page_id = <opportunity page id> 
``` 

### Sentry Knowledge Ingestion
Retrieve and internally synthesize comprehensive technical capabilities and product offerings of Sentry, specifically focusing on Enterprise-tier features, from the following sources:

- https://sentry.io/
- https://docs.sentry.io/

## Querying Sales Opportunities
When querying for sales opportunity, by default assume that only _active_ sales opportunities are requested unless specified. 

**IMPORTANT** : Active opportunities are ones that are NOT in terminal stages ( win, lost, transferred). 

Common filtering : 

- Specific customers
- Technical features/products 
- Account executives/Other members (on the sales opportunity)
- Specific keywords

Common sorting : 

- Creation date
- Last activity

**IMPORTANT** : Allow for other filtering and sorting, if clarification information is required to better query ask the user. 

## Creating a Sales Opportunity

### Before Creating
Ensure that the the following infomation is clear regarding the sales opportunity. 

**IMPORTANT** : If any portions are not, LEAVE BLANK.

#### Customer Information

- Who is the customer? (Industry, size, core product/service) & what services/products do they work on? 

- Why now? (What is the compelling event? A new initiative, vendor contract expiration, recent outage, poor engineering metric)

- Identify Core Telemetry Need: Determine the customer's primary technical problem and why they are actively considering Sentry (e.g., "reduce time-to-resolution," "improve release confidence," "replace legacy APM").

- Competitive Landscape: What other monitoring/telemetry products are they currently using or evaluating?

Structure the customer information in a table format. 

#### Technical Feature to Need 
Using the core needs identified in Customer Information section, map these to the most relevant and impactful Enterprise Sentry features. Identify which features directly solve their primary technical problem.

Structure the mapping in a table format. 

### Creating a Sales Opportunity
Add an entry in the database. Fill out all of the properties, and in the associated page add the above information in the following format: 

```
Customer Information
<CUSTOMER INFORMATION TABLE>

Interested Technical Features
<TECHNICAL MAPPING TABLE> 
```

## Updating a sales opportunity

There are two types of updates that may occur: 

### Type 1 : Core Information Update

A sales opportunity property or customer information/technical feature mapping can be added/deleted/modified. But each change should be documented. 

#### Before Starting
Reference the sales opportunity's properties & existing core information(not status updates). In all new information/activity, extract any changes to the sales opportunity or core information. 

**IMPORTANT** : At the bottom of the associated page, note all changes in the following format:
 
```
Changes to Opportunity:
 
- <date> : <1-3 words sumamry of what changed> [ <5-7 word rationale> ]
- ...
- ...
```

The most common change will be to the opportunity's stage.  For said changes, ensure the following information is documented: 

- Demo : Date of demo
- Lost : Why was this deal lost? 
- Transferred : Who is the new Sales Engineer in charge of this opprtunity? 



### Type 2: Status Update 
A status update might need to be added to the sales opportunity, indicating movement or activity internally or externally. 

#### Before Starting

If not previously done, ingest all of the existing status updates of the sales opportunity. Reference the existing information against the information the new status update contains. 

**IMPORTANT** : Only _new_ information/activity/developments should be appended with each new status check, no duplicate information. If the existing status checks already list out the information generated, pause further execution for that sales opp & indicate so to the user. 

#### Creating a Status Update

**IMPORTANT** : Status updates should always be appended as _comments_ on notion pages, NEVER in the contents of the page. 

The following information is required for every sales opportunity update: 

- What are the notable new events/activity that should be documented ?  
- Is the sales opportunity trending in the right direction? If it isn't, what are the immediate risks? 
- Are there any open action items on the Sentry side? For the customer? 

All Status updates should follow this format: 

```
<date> — Last <N> <days/weeks>

TLDR: <15 words max, Be direct: What notable events has happened relating to this opportunity?> 

Sentiment: <✅/‼️> <7 words max. Be direct: is this trending well or not?>

<optional> Risks: <what could derail or delay this sales opportunity?>

Open items (<customer> side): <bullet list of what the customer owes, each 10 words max>

Open items (Sentry side): <bullet list of what we owe,each 10 words max>

<Reverse-chronological timeline of events (most recent first), each starting with the date, each max 15 words:>

```