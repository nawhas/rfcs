# Thoughts on Auditing

**Authors:** [@szainmehdi](https://github.com/szainmehdi)  
**Type:** Feature

## Overview
The goal of Nawhas.com is to allow everyone in the community to contribute to the site, uploading new content and modifying existing content. This comes with a lot of challenges in preventing abuse, allowing rollbacks of bad changes, and keeping data integrity.

There are a number of approaches to handle auditing. This document aims to discuss the pros, cons, challenges, and solutions offered by each of the approaches we've thought of.

## Scope and Requirements
First, let's discuss the requirements and scope of auditing in the context of Nawhas.com

### R1 :: View All Changes in Reverse Chronological Order
As a moderator of Nawhas.com, I need to see what has changed recently so that I can take appropriate actions against bad or abusive changes.

This information should be visible in a Moderator Dashboard in the form of a Revision History.

Each change should include:
- The entity ID
- The entity type
- The change type
- The content of changes applied

### R2 :: Track Which User Made Changes
As a moderator of Nawhas.com, I need to see who made a given change to an entity so that, in the future with the right tooling, I can take appropriate actions against that user.

Actions (out of scope) against a user might include:
- Placing contributions in a moderation queue
- Quarantining a user's future contributions (shadow-ban)
- Banning the user.

### R3 :: Restoring State
As a moderator of Nawhas.com, I need to be able to restore an entity to a previous state in order to rollback bad changes.

Restoring state might include:
- Ability to restore a single entity to a previous revision.
- Ability to revert a single change to an entity (description, for example)
- Ability to restore an entity and all related entities to a previous state.

### R4 :: Entity-Scoped Change History
As a moderator of Nawhas.com, I need to be able to see every change done to a given entity, including information about who changed it, and when, so that I can ensure no data is being lost.

I should be able to pull a list of changes with all relevant information for a particular entity, for example, a specific reciter. 

## Approaches

### Audit Records
With a minimal amount of changes to our application, we've been able to come up with a pretty strong auditing system. https://github.com/nawhas/nawhas/pull/203

This approach has allowed us to, for the most part, keep an audit log of what changes were made to the data, and who made them. 

This approach already fulfills much of **R1** and **R2**, and can support **R4** with relative ease.

#### High Level Overview:
- Save changes to the database using Doctrine.
- Manually fire domain events when a change is made.
- `Auditor` class subscribes to these domain events and writes an `AuditRecord`
  - `AuditRecord` contains 
    - JSON encoded values of what the entity looked like before and after the change
    - User ID
    - Timestamps
  - Only certain fields are tracked, defined by the model.

#### Limitations
- Doctrine's `getOriginalEntityData()` is unreliable and difficult to work with.
  - This function, which we're relying upon for grabbing the old values of a changed entity, does not work well for any Entity property that is not a primitive (i.e. string or int, etc.)
- This approach will likely never evolve to support a "moderation queue" or a "shadow-ban" approach to moderation (**R2**). 
  - By design, this only works for changes that have been made _already_. Coming up with a solution for a _moderator queue_ with this approach would be near impossible.
- Restoring changes would be a bit of a manual process, but can be done. 
  - Since we're keeping track of changes, and we have an "old" value and a "new" value, we can, with relative ease, roll back a certain change. 
  - We can't, however, easily roll back to a particular state. This is a solvable problem with better use of serializers, reflection, or better events, but is more complicated than it seems.

### Event Sourcing
Event sourcing is a different way of thinking about the application domain. It primarily involves storing _events_ that occurred, instead of modifying rows in the database directly. The current state of the database is derived from the events and can be rebuilt at any time. This allows a strong audit trail by design, and makes traditional _models_ operate more like read-only views that can be reconstructed at any point to serve the needs of the application.

#### High Level Overview:
- Fire domain events, such as `ReciterCreated` or `ReciterAvatarUploaded` whenever a change occurs.
- Event is stored in the database with the relevant information
- `Projector` classes respond to events and modify the read-only tables.
- Application uses read-only tables to power API / frontend.
- Every single change to the database is tracked as an event.

With clever design, this approach can be extended to:
- Build a revisions table of an entity or an aggregate root (or both) to restore a model to a previous state at any time.
- Push changes into a moderation queue that don't affect live data until approved.
- Shadow ban users by pretending that certain changes have been applied to live data depending on the logged in user
- Build powerful new ways of using the data in the system

### Eloquent with Laravel Auditing
http://www.laravel-auditing.com/

## Glossary

### moderation queue
This technique involves allowing a contributor to make a change to a given entity, and hold that change in a queue requiring a moderator to approve it before that change is published to the website for all to see.

For example:
1. Live Site: `Reciter (id: 1, name: Zain)`
2. Change 1: `User A` edits Reciter's name to "Zain Mehdi"
3. Live Site: `Reciter (id: 1, name: Zain)`
4. `Moderator Z` approves Change 1
5. Live Site: `Reciter (id: 1, name: Zain Mehdi)`




```
