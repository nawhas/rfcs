# Audio Player: Shuffle Queue
**Authors:** [@szainmehdi](https://github.com/szainmehdi) [@shea786](https://github.com/shea786)  
**Type:** Feature

## Overview
In the audio player, users should be able to shuffle the tracks in the queue.

## Why are we making this change?
A common feature of any audio player is shuffling tracks. Users expect this functionality from Nawhas.com.

## Requirements
- The audio player should have a shuffle button.
- When shuffle is turned on, the audio player queue should be randomly re-ordered.
- When shuffle is turned off, the original order of the queue should be restored, but the current track should keep playing.
- Clicking the next button in the audio player should go to the next track whether shuffled or not.
- If shuffle is turned on, clicking the back button in the audio player should play the previous track in the shuffled queue. 
- The shuffled queue should start at the current track that is playing.

## Detailed Eng Design

### API
No API changes are necessary. 

### Frontend
We currently store upcoming tracks in a Vuex store. 
https://github.com/nawhas/nawhas/blob/e2ad6a3414eed217ab7f2e32918daa1cb480d3f1/web/src/store/modules/player.ts#L19-L24

#### Vuex Store
Some changes will need to be made to the `player` Vuex module.

#####  State

- Add a new `shuffledQueue` property
  - This will be contain an array of `QueuedTrack` same as the `queue` property.

##### Mutations

- Add `TOGGLE_SHUFFLE`
  - If `isShuffled` is true then do the following
      - Generate a random array from the current `queue`.
      - Then move the current track that is playing to the top of the list. 
      - Commit this new array to the store as `shuffledQueue`

  - If `isShuffled` is false then do the following
      - Set `shuffledQueue` to `null`

- Modify `REMOVE_TRACK`
  - Remove the track from the `shuffledQueue` queue if applicable, as well as the `queue`

- Modify `STOP`
  - Make the `shuffledQueue` to an empty array.

##### Getters
- Add `isShuffled` getter
  - If the length of `shuffledQueue` is greater than 0 then return `true`
  - If the length of `shuffledQueue` is equal to `0` then return `false`

- Add `queue` getter
  - If `isShuffled` is true then return `shuffledQueue` state
  - If `isShuffled` is false then return `queue` state

- Modify `track`
  - Make the `track` getter utalize the new `queue` getter to automatically get the queue depending on if its shuffled or not

#### AudioPlayer Component

- Add a new shuffle icon-button. 
  - Utilize the [`shuffle`](https://material.io/resources/icons/?search=shuffle&style=baseline) icon from Material Design.
  - This would be the same size as the current queue icon. 
  - This will be situated on the left hand side of the back play buttons
  - When the button is clicked, trigger `TOGGLE_SHUFFLE` mutation on the `player` store

- Change the `queue` computed method to get the `queue` getter from the `player` store

####  QueueList Component
If shuffle is on then render the 'shuffledQueue' array. If shuffle is off then render the 'queue' array

### Migration Path
No changes to migration

### Deployment Strategy

## Mockups
<!-- If there are any -->


## Test Cases

### Turning shuffle off should restore order of queue.
 - Tracks A, **B**, C, D, E, and F are on the queue.
 - Track B is playing.
 - Shuffle is turned on.
 - Shuffled Queue is now [**B**, E, C, A, F, D]
 - User skips to next track.
 - Track E is now playing.
 - Shuffle is turned off.
 - Queue is now [A, B, C, D, **E**, F]
 - User skips to previous track.
    
**Expectation:** Track D is now playing

## Open Questions
- Should the shuffle icon be situated next to the current queue and expand icons or should it be situated to the left of the play and back buttons like most audio players?
  - As answered by @szainmehdi , the shuffle button will be on the left of the play back button
