# Patching Lyrics
**Authors:** [@szainmehdi](https://github.com/szainmehdi)  
**Type:** Feature

## Overview
This change will allow tracking changes to lyrics in a much more robust way, giving us the tools to handle content editing in a much more sophisticated way and paving the way for advanced moderation features. Read on for more.

### Definitions
<!-- 
Define terms used in this document, if not obvious. 
Remove this section if unnecessary. 
-->

| Term            | Definition                                                        |
| --------------- | ----------------------------------------------------------------- |
| Lyrics Document | refers to the JSON or Plain Text content of the lyrics model      |
| Lyrics Entity   | refers to the entity and corresponding database schema for Lyrics |

## Why are we making this change?

Let's take a moment to highlight each of the potential problems this feature may help solve.

### Conflicting Changes

Currently, if multiple users edit lyrics for the same track simultaneously, we have a major problem: whoever submits their changes **last** wins. The previous changes are simply ignored/overwritten. The timeline below might help illustrate the issue.

![Conflicting Changes Timeline](./why-1-conflicting-changes.jpg)

As you can see above, **Blue** could have spent 15 minutes tediously writing up the lyrics for the track. **Orange** on the other hand just made a simple correction. If the timing is right, the change made by **Orange** can completely blow away the changes made by **Blue**.

A couple notes: we can prevent this from happening without major changes. We simply need to make sure that the version of tracks a user is editing is still the active version when we're saving the new content, otherwise, we reject the changes.

That leads to another issue though. Let's reverse the roles in the timeline and make it so that **Orange** makes massive changes and **Blue** makes a tiny spelling correction. If we implemented a "reject-if-out-of-date" policy, we'd end up throwing away all of **Orange**'s work because **Blue** got through first. This is unideal.

Instead of a "reject-if-out-of-date" policy, we could _allow  conflicting changes_ but then log/alert moderators to review situations of the sort. After all, we do have a historical log of every single _version_ of the lyrics. But this approach also leads to some undesirable situations. Human intervention is required whenever this situation arises, and worse, there's no easy way to merge the changes together. (Suppose **Orange** fixed 100 misspellings, while **Blue** added timestamps to the entire track. How do we preserve both contributions?)

### Moderating Changes and Reverting A Change
With the system we currently have in place, here's what a sequence of changes would look like. 

```
Version 1:
 Moula x3
 Roze alt mujshe
 sawaal ye kaya gaya
 ...
   
Version 2:
 Maula x2
 Roze alt mujshe
 sawaal ye kya gaya
 ...
  
Version 3:
 Moula x2
 Roze alast mujhse
 sawaal ye kiya gaya
 ...
```

Of course, this is a super contrived example, but this highlights a few potentially serious problems.

1. Even though this example uses only 3 lines, it's not immediately obvious **what exactly changed** in each revision.
2. If you _just_ wanted to revert the changes made in Version 2, you can't easily do so.

Imagine a scenario where a moderator reviews the changes made to a track once a week. If there have been 10 different sets of changes, how do you effectively moderate this? How can you revert a change in Version 3 if the document is now at Version 10?

## The Solution

In theory, the solution is simple: **diffs and patches**. This will help us identify what changed easily and allow moderaters to be more effective, potentially solve the conflicting changes issue, and potentially allow reverting previous changes easily.

Here's how the proposed feature will work at a high level.

On the frontend in the editor, instead of sending a full JSON object payload to edit lyrics, we'll send a `JSON Patch` object, which describes just the changeset.

An example: 

```json
[
  {
    "op": "replace",
    "path": "/data/0/timestamp",
    "value": 0
  },
  {
    "op": "replace",
    "path": "/data/0/lines/0/text",
    "value": "Maula"
  },
  {
    "op": "replace",
    "path": "/data/0/lines/0/repeat",
    "value": 3
  },
  {
    "op": "replace",
    "path": "/data/1/timestamp",
    "value": 14.0007
  },
  {
    "op": "replace",
    "path": "/data/2/timestamp",
    "value": 18.0579
  },
  {
    "op": "replace",
    "path": "/data/2/lines/0/text",
    "value": "Kya tum ko hai qubool Ali-un-Wali-ullah"
  },
  {
    "op": "replace",
    "path": "/data/3/timestamp",
    "value": 23.5676
  },
  {
    "op": "replace",
    "path": "/data/4/timestamp",
    "value": 27.5854
  },
  {
    "op": "replace",
    "path": "/data/4/lines/0/text",
    "value": "Honto(n) se mere bas yehi aati thi ik sada"
  }
]
```

This sort of payload, along with the ID (essentially, the "version" number) of the lyrics entity, will be sent to the API for processing. 

- If there's no version mismatch, the patch will be applied, a new Lyrics entity generated, and the patch stored on the lyrics entity.
- If there IS a version mismatch, we'll try our best to apply the patch. If not possible, we'll return an error to the frontend and attempt to create a conflict resolution system of some sort. 🤔
- If the patch is reversible, we're store the reverse patch as well.

## Requirements
<!-- Provide a list of acceptance criteria. Example:
- Engineers can provision a Moderator account.
- Moderators can log in to Nawhas.com with an email address and password.
- Moderators can log out of Nawhas.com to end their session.
- The frontend application can determine if a User is logged in.
- The frontend application can determine if a User is a Moderator.
-->

## Detailed Engineering Design

### API
<!-- Describe any changes necessary to the API to make this feature possible. -->
Library: https://github.com/swaggest/json-diff

### Frontend
<!-- Describe any changes necessary to the Vue app to make this feature possible. -->
Library: https://github.com/cujojs/jiff



### Deployment Strategy
<!-- Describe rollout strategy. -->

## Mockups
<!-- Attach relevant mockups here. Links to Figma are also appropriate. -->