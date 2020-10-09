# Redirects
**Authors:** [@szainmehdi](https://github.com/szainmehdi)  
**Type:** Feature

## Overview
Provide a way for the frontend to implement 301 Permanent Redirects when slugs change for reciters, albums, and tracks. Additionally, attempt to import old URLs from the original nawhas.com and preserve those with redirects as much as possible.

## Why are we making this change?
<!-- Explain the motivation for this change -->
<!-- 
Example: 
To achieve the greater effort of allowing public edit access on Nawhas.com, enabling moderators to log in is a prerequisite. This change will also lay the foundation for the overall authentication system and enable Contributor registration and logins in the future.
-->

When a reciter's name changes, the slug is updated, and all previous links (nested under `/reciters/{slug}`) are invalidated and begin to 404. This is terrible for user experience, and terrible for SEO. Similar situations arise when changing the `year` of an album, or the title of a track.

To preserve old links, we need to implement a 404 handler on the frontend, and a way for the frontend to determine what the new link should be.

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
When certain events are fired, like `ReciterNameChanged`, `TrackTitleChanged`, or `AlbumYearChanged`, we need to write records to the database to support redirects.

#### Database Changes

#####  `reciter_aliases`

| field      | type   | notes                |
| ---------- | ------ | -------------------- |
| id         | uuid   | PK                   |
| reciter_id | uuid   | FK for `reciters.id` |
| alias      | string | An old slug          |

##### `album_aliases`

| field      | type   | notes                |
| ---------- | ------ | -------------------- |
| id         | uuid   | PK                   |
| reciter_id | uuid   | FK for `reciters.id` |
| album_id   | uuid   | FK for `albums.id`   |
| alias      | string | An old slug          |

##### `track_aliases`

| field      | type   | notes                |
| ---------- | ------ | -------------------- |
| id         | uuid   | PK                   |
| reciter_id | uuid   | FK for `reciters.id` |
| album_id   | uuid   | FK for `albums.id`   |
| track_id   | uuid   | FK for `tracks.id`   |
| alias      | string | An old slug          |

#### Endpoint

```
POST /v1/redirect
{
  "parameters": {
    [key: string]: string;
  }
}

(example)
POST /v1/redirect
{
  "parameters": {
    "reciter": "the-tejani-brothers",
    "album": "2018"
  }
}

Responses:
 - 404: no redirect found
 - 200: redirect found

200 OK
{
  "to": "/reciters/tejani-brothers/albums/2018-19",
}
```

### Frontend
<!-- Describe any changes necessary to the Vue app to make this feature possible. -->

### Deployment Strategy
<!-- Describe rollout strategy. -->

## Mockups
<!-- Attach relevant mockups here. Links to Figma are also appropriate. -->
