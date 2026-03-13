# RecentTracks

Methods for working with listening history.

| Method | Result Type | Brief Description |
|--------|-------------|-------------------|
| [appendTracks](/reference/filter?id=appendtracks) | - | Add array of tracks to listening history file. |
| [compress](/reference/filter?id=compress) | - | Remove insignificant track data in cumulative listening history files. |
| [get](/reference/filter?id=get) | Array | Get listening history tracks. |

## appendTracks

Add array of tracks to listening history file. Playlist addition date `added_at` becomes listening date `played_at`. If no date exists, the function execution date is set. Sorted by listening date from newest to oldest.

!> Note the 60 thousand track limit for listening history. The limit can be increased in the `config` file.

### Arguments :id=appendtracks-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `filename` | String | Listening history filename: `SpotifyRecentTracks` or `LastfmRecentTracks`. |
| `tracks` | Array | Tracks to add. |

### Return :id=appendtracks-return {docsify-ignore}

No return value.

### Examples :id=appendtracks-examples {docsify-ignore}

1. Add all favorite tracks to listening history

```js
let tracks = Source.getSavedTracks();
RecentTracks.appendTracks('SpotifyRecentTracks', tracks);
```

## compress

Remove insignificant track data in cumulative listening history files depending on [parameters](/config). A file copy is created first.

?> Used for compatibility with previous library versions. One execution is enough to compress listening history files. New history tracks are compressed automatically.

### Arguments :id=compress-arguments {docsify-ignore}

No arguments.

### Return :id=compress-return {docsify-ignore}

No return value.

## get

Get listening history tracks depending on [parameters](/config). Sorted by listening date from newest to oldest.

### Arguments :id=get-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `limit` | Number | If specified, limit the number of tracks returned. If not, all available. |
| `isDedup` | Boolean | If `true` removes duplicates, if `false` leaves as is. Default is `true` |

#### Listening History Parameters

| Enabled Parameter | Returned Array |
|-|-|
| `ON_SPOTIFY_RECENT_TRACKS` | Spotify listening history only. File `SpotifyRecentTracks`. Duplicates are not removed. |
| `ON_LASTFM_RECENT_TRACKS` | Lastfm listening history only. File `LastfmRecentTracks`. Duplicates are not removed. |
| `ON_SPOTIFY_RECENT_TRACKS` and `ON_LASTFM_RECENT_TRACKS` | Combined sources with duplicate removal. File `BothRecentTracks`. |

### Return :id=get-return {docsify-ignore}

`tracks` (array) - listening history tracks.

### Examples :id=get-examples {docsify-ignore}

1. Get an array of listening history tracks. Track source depends on parameters.

```js
let tracks = RecentTracks.get();
```

2. Get 100 listening history tracks.

```js
let tracks = RecentTracks.get(100);
```
