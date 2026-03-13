# Player

Methods for controlling the player.

| Method | Return Type | Brief Description |
|-------|-------------|-------------------|
| [addToQueue](/en/reference/player?id=addtoqueue) | - | Add tracks to the playback queue. |
| [getAvailableDevices](/en/reference/player?id=getavailabledevices) | Array | Get list of available devices. |
| [getPlayback](/en/reference/player?id=getplayback) | Object | Get player data, including the currently playing track. |
| [next](/en/reference/player?id=next) | - | Skip to the next track in the queue. |
| [pause](/en/reference/player?id=pause) | - | Pause playback on the current player. |
| [previous](/en/reference/player?id=previous) | - | Go to the previous track in the queue. |
| [resume](/en/reference/player?id=resume) | - | Resume playback of the current queue or create a new queue. |
| [setRepeatMode](/en/reference/player?id=setrepeatmode) | - | Set repeat mode. |
| [toggleShuffle](/en/reference/player?id=toggleshuffle) | - | Toggle queue shuffle mode. |
| [transferPlayback](/en/reference/player?id=transferplayback) | - | Transfer current playback to another device. |

## addToQueue

Add tracks to the playback queue. Equivalent to _play next_ in the Spotify interface.

### Arguments :id=addtoqueue-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `items` | Array/Object | Array of tracks or a track object to add to the queue. Only the _id_ is relevant. |
| `deviceId` | String | Device identifier. Optional when playback is active. |

### Return :id=addtoqueue-return {docsify-ignore}

No return value.

### Examples :id=addtoqueue-examples {docsify-ignore}

1. Play next the most recently added liked track.

```js
let tracks = Source.getSavedTracks(1);
Player.addToQueue(tracks[0]);
```

## getAvailableDevices

Get the list of available devices (currently connected to Spotify). Use to get the device _id_. The value from `getPlayback` becomes empty fairly quickly when paused.

### Arguments :id=getavailabledevices-arguments {docsify-ignore}

No arguments.

### Return :id=getavailabledevices-return {docsify-ignore}

`devices` (array) - available devices. [Array example](https://developer.spotify.com/documentation/web-api/reference/#endpoint-get-a-users-available-devices).

### Examples :id=getavailabledevices-examples {docsify-ignore}

1. Select a device by type. `Smartphone` for phone, `Computer` for PC.

```js
let device = Player.getAvailableDevices().find(d => d.type == 'Smartphone');
// device.id
```

## getPlayback

Get player data, including the currently playing track. Becomes empty fairly quickly when paused.

### Arguments :id=getplayback-arguments {docsify-ignore}

No arguments.

### Return :id=getplayback-return {docsify-ignore}

`playback` (object) - player data. [Object example](https://developer.spotify.com/documentation/web-api/reference/#endpoint-get-information-about-the-users-current-playback).

### Examples :id=getplayback-examples {docsify-ignore}

[Usage example](https://github.com/Chimildic/goofy/discussions/102)

## next

Skip to the next track in the queue.

### Arguments :id=next-arguments {docsify-ignore}

No arguments.

### Return :id=next-return {docsify-ignore}

No return value.

## pause

Pause playback on the current player.

### Arguments :id=pause-arguments {docsify-ignore}

No arguments.

### Return :id=pause-return {docsify-ignore}

No return value.

## previous

Go to the previous track in the queue.

### Arguments :id=previous-arguments {docsify-ignore}

No arguments.

### Return :id=previous-return {docsify-ignore}

No return value.

## resume

Resume playback of the current queue or create a new queue.

### Arguments :id=resume-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `params` | Object | Queue parameters. |

#### Queue Parameters :id=resume-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `deviceId` | String | Device identifier. Optional when playback is active. |
| `context_uri` | String | Play by URI, for example a playlist or album. |
| `tracks` | Array | Create a new queue with tracks. Use either `context_uri` or `tracks`. |
| `position_ms` | Number | Set track progress in milliseconds. |
| `offset` | Number | Set the active track in the queue `{ "position": 5 }`. Zero-based index. |

### Return :id=resume-return {docsify-ignore}

No return value.

### Examples :id=resume-examples {docsify-ignore}

1. Resume playback after pause.

```js
Player.pause();
Utilities.sleep(5000);
Player.resume();
```

2. Create a queue from liked tracks

```js
let tracks = Source.getSavedTracks();
Player.resume({
    tracks: tracks
});
```

3. Play a playlist by URI

```js
let playlistId = '37i9dQZF1DWYmDNATMglFU';
Player.resume({
    context_uri: `spotify:playlist:${playlistId}`,
});
```

## setRepeatMode

Set repeat mode.

### Arguments :id=setrepeatmode-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `state` | String | When `track` repeats the current track, when `context` repeats the current queue, when `off` it's disabled. |
| `deviceId` | String | Device identifier. Optional when playback is active. |

### Return :id=setrepeatmode-return {docsify-ignore}

No return value.

## toggleShuffle

Toggle queue shuffle mode.

### Arguments :id=toggleshuffle-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `state` | String | When `true` enables shuffle, when `false` disables it. |
| `deviceId` | String | Device identifier. Optional when playback is active. |

### Return :id=toggleshuffle-return {docsify-ignore}

No return value.

## transferPlayback

Transfer current playback to another device (i.e., the queue and playing track, [getPlayback](/en/reference/player?id=getplayback)).

### Arguments :id=transferplayback-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `deviceId` | String | _id_ of the new device. Available values can be obtained, for example, via [getAvailableDevices](/en/reference/player?id=getavailabledevices). |
| `isPlay` | Boolean | When `true` playback will start on the new device. When not specified or `false` the state will remain the same as on the previous device. |

### Return :id=transferplayback-return {docsify-ignore}

No return value.

### Examples :id=transferplayback-examples {docsify-ignore}

[Usage example](https://github.com/Chimildic/goofy/discussions/126)