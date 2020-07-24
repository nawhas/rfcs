# Event Sourcing
**Authors:** [@szainmehdi](https://github.com/szainmehdi)  
**Type:** Feature

## Overview
Event sourcing is a different way of thinking about the application domain. It primarily involves storing _events_ that occurred, instead of modifying rows in the database directly. This allows a strong audit trail by design, and makes traditional _models_ operate more like read-only views that can be reconstructed at any point to serve the needs of the application.

### Definitions
<!-- 
Define terms used in this document, if not obvious. 
Remove this section if unnecessary. 
-->

| Term        | Definition                                            |
| ----------- | ----------------------------------------------------- |
|             |                                                       |

## Why are we making this change?
<!-- Explain the motivation for this change -->
<!-- 
Example: 
To achieve the greater effort of allowing public edit access on Nawhas.com, enabling moderators to log in is a prerequisite. This change will also lay the foundation for the overall authentication system and enable Contributor registration and logins in the future.
-->

## Requirements
<!-- Provide a list of acceptance criteria. Example:
- Engineers can provision a Moderator account.
- Moderators can log in to Nawhas.com with an email address and password.
- Moderators can log out of Nawhas.com to end their session.
- The frontend application can determine if a User is logged in.
- The frontend application can determine if a User is a Moderator.
-->

## Events

### Reciters
| Event                 | Description                         |
| --------------------- | ----------------------------------- |
| ReciterCreated        |                                     |
| ReciterModified       |                                     |
| ReciterDeleted        |                                     |
| ReciterAvatarUploaded | Specifically handles avatar uploads |

### Albums
| Event                | Description                          |
| -------------------- | ------------------------------------ |
| AlbumCreated         |                                      |
| AlbumModified        |                                      |
| AlbumDeleted         |                                      |
| AlbumArtworkUploaded | Specifically handles artwork uploads |

## Detailed Engineering Design

### API

#### Event
The event class will consist of:
```
id: uuid
name: string (event name)
payload: json
recorded_at: timestamp
```

#### ProjectedEvent
For each _version_ of our database, we'll track which events have been projected already. This should help alleviate some of the concerns with replaying events on each deploy and knowing which events have been processed by which projectors.

```
id: uuid
event_id: uuid
projector: string
projected_at: timestamp
```

#### Projectors
We'll have projectors that handle every change event and update the read-only database. The obvious ones that update our traditional, relational database tables are not going to be covered in depth, but for example:

```
class RecitersProjector
{
  public function onReciterCreated(ReciterCreated $event): void
  {
    $reciter = new Reciter($event->attributes);
    $reciter->save();
  }
}
```

##### Revisions
In addition, we'll track revisions when any relevant event occurs.

A revision essentially contains a JSON-serialized version of the entity, with a version number.

```
id: uuid
entity: string
version: int
content: json
created_at: timestamp
```

```
class RevisionsProjector 
{
  public function onRevisionableEvent(Revisionable $event)
  {
    // Create a revision for the entity.
  }  
}
```

### Database
Events will be written to and read from a separate database. This will allow us to potentially spin up new databases on every deploy and replay events.

All events will be stored in a `source` database connection.
On a deploy, we'll:
- Create a new database with called `nawhas_v` suffixed by the version tag of the deploy.
  - e.x. version 3.1.2 will be deployed with `nawhas_v3_1_2` 
- Migrate all tables
- Replay all events into the new database
- Run a sanity-check verification script
- Go live with that pod.

This will allow us to rebuild data dynamically as needed for each deploy.

#### Concerns & Open Questions
- What happens with events that occur during the time the old version of the app is still writing events but the new version of the app is reading events?

### Frontend
<!-- Describe any changes necessary to the Vue app to make this feature possible. -->

### Deployment Strategy
<!-- Describe rollout strategy. -->

## Mockups
<!-- Attach relevant mockups here. Links to Figma are also appropriate. -->