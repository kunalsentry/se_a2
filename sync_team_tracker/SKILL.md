---
name: sync-team-tracker
description: Sync existing sales opportunities in personal tracker with the team tracker
---

# Sync Team Tracker with Personal Tracker
Handles syncing all new creations & updates in an sale engineer's personal sales opportunity tracker with the team tracker. Outline's guidelines on how to summarize & push updates. 

## Why this is Important
Some sales engineers track all of their existing/previous opportunities in personal Notion databases, seperate from the team-level tracker that is referenced by managers & executives. Keeping both in sync is tedious & difficult. This skill automates that process, modifying the team tracker with updates made in the personal tracker. 

## When to Invoke
Only invoke this skill if user explicitly requrests syncing their current opportunities with team tracker, or updating the team tracker.

**IMPORTANT** : The usual team sync where the tracker is parsed through occurs on Fridays. This skill should usually run on Thursday or Friday (morning), if the user requests on any other day pause execution and ask them to confirm. 

## Before Starting

### Parse personal tracker
If not previously done, parse the user's personal opportunity tracker for all sales opportunities that have had some update in the past week. 

**IMPORTANT** : The update can be either on the entry(property, page contents) or a status update(comment). 

Per opportunity in any tracker, make **both** of the following calls before proceeding to anything else: 

``` 
notion-fetch: page_id = <opportunity page id>, include_discussions: true notion-get-comments: page_id = <opportunity page id> 
``` 


Note the schema of the personal tracker.

### Parse team tracker

**IMPORTANT** : Only a single notion page with a database should be referenced. If not provided, ask the user for the link to the team POC tracker Notion database. 

If not previously done, parse the team's opportunity tracker _only_ for opportunities that are owned by the user. 

**IMPORTANT** : Only reference team opportunity entries that the user is listed as a sales engineer for. 

Note the schema of the team tracker. 

## Comparing data & Generating Changes 

**IMPORTANT** : In this stage DO NOT make any changes to either the personal or team tracker. Only reference, then output findings. 

Per opportunity in the personal tracker, compare against the opportunities parsed from the team tracker. Identify the deltas, and what changes should be made in the team tracker to reflect the updates documented in the personal tracker. 

### New Opportunity Creation
A new entry might need to be created and populated with information, if the personal tracker has an opportunity that currently isn't being tracked in the team tracker. 

### Modifying Existing Opportunity

Changes that can occur to existing opportunity entries: 

#### Properties
Properties can be modified if required. 

**IMPORTANT** : There isn't a 1:1 mapping in properties between the personal & team tracker. Air on the side of caution when reccomending modifications, if not 99% confident in change don't generate 

#### Page
The page per opportunity on the team tracker should be _exactly_ the same as the related opportunity on the personal tracker. 

#### Comments
**IMPORTANT** : This is the highest priority & most referenced element of the team tracker. 

Reference the status updates in the personal tracker, and find the delta in information/activity documented with the team tracker updates. 

**IMPORTANT** : Only _new_ information/activity/developments should be appended with each new status check, no duplicate information. If the existing status checks already list out the information generated, pause further execution for that sales opp & indicate so to the user. 

All Status updates should follow this format: 

```
<date> — Last <N> <days/weeks>

TLDR: <15 words max, Be direct: What notable updates have occured in this opportunity?> 

Sentiment: <✅/‼️> <7 words max. Be direct: is this trending well or not?>

<optional> Risks: <what could derail or delay this sales opportunity?>

<Reverse-chronological timeline of events (most recent first), each starting with the date, each max 7 words:>

```

## Output Changes
Output the changes to be made to the team tracker.

**IMPORTANT**: DO NOT make any changes yet. Ask user if you should proceed, or if any changes should be made. 

