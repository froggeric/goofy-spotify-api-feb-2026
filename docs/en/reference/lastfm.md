# Lastfm

Methods for interacting with Last.fm.

!> Track equivalents are found via Spotify search for the best match. If there is no match, the track is ignored. One Last.fm track equals one search request. Be mindful of the [limits](/en/details?id=Limits) on the number of requests per day and execution time.

| Method | Result Type | Brief Description |
|-------|--------------|------------------|
| [convertToSpotify](/en/reference/lastfm?id=converttospotify) | Array | Find Spotify items using data from Last.fm. |
| [getAlbumsByTag](/en/reference/lastfm?id=getalbumsbytag) | Array | Get albums by tag. |
| [getArtistsByTag](/en/reference/lastfm?id=getartistsbytag) | Array | Get artists by tag. |
| [getCustomTop](/en/reference/lastfm?id=getcustomtop) | Array | Get a custom user top. |
| [getLibraryStation](/en/reference/lastfm?id=getlibrarystation) | Array | Get tracks from the _library_ radio station. |
| [getLovedTracks](/en/reference/lastfm?id=getlovedtracks) | Array | Get Last.fm _likes_. |
| [getMixStation](/en/reference/lastfm?id=getmixstation) | Array | Get tracks from the _mix_ radio station. |
| [getNeighboursStation](/en/reference/lastfm?id=getneighboursstation) | Array | Get tracks from the _neighbors_ radio station. |
| [getRecentTracks](/en/reference/lastfm?id=getrecenttracks) | Array | Get a user's Last.fm listening history.  |
| [getRecomStation](/en/reference/lastfm?id=getrecomstation) | Array | Get tracks from the _recommendations_ radio station. |
| [getSimilarArtists](/en/reference/lastfm?id=getsimilarartists) | Array | Get similar artists. |
| [getSimilarTracks](/en/reference/lastfm?id=getsimilartracks) | Array | Get similar tracks. |
| [getTopAlbums](/en/reference/lastfm?id=gettopalbums) | Array | Get a user's top albums. |
| [getTopAlbumsByTag](/en/reference/lastfm?id=gettopalbumsbytag) | Array | Get top albums by tag. |
| [getTopArtists](/en/reference/lastfm?id=gettopartists) | Array | Get a user's top artists. |
| [getTopArtistsByTag](/en/reference/lastfm?id=getTopArtistsByTag) | Array | Get top artists by tag. |
| [getTopTracks](/en/reference/lastfm?id=gettoptracks) | Array | Get a user's top tracks. |
| [getTopTracksByTag](/en/reference/lastfm?id=gettoptracksbytag) | Array | Get top tracks by tag. |
| [getTracksByTag](/en/reference/lastfm?id=gettracksbytag) | Array | Get tracks by tag. |
| [rangeTags](/en/reference/lastfm?id=rangetags) | - | Filter tracks by tags. |
| [removeRecentArtists](/en/reference/lastfm?id=removerecentartists) | - | Remove artists based on Last.fm listening history data. |
| [removeRecentTracks](/en/reference/lastfm?id=removerecenttracks) | - | Remove tracks based on Last.fm listening history data. |

## convertToSpotify

Find Spotify items using data from Last.fm.

### Arguments :id=converttospotify-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `items` | String | Items in lastfm format. For example, obtained from [getCustomTop](/en/reference/lastfm?id=getcustomtop) with `isRawItems = true`. |
| `type` | String | Search type: `track`, `artist`, or `album`. Defaults to `track`. |

### Return :id=converttospotify-return {docsify-ignore}

`items` (array) - search result.

## getAlbumsByTag

Get albums by tag. Parses names from the tag page, [for example](https://www.last.fm/tag/indie/albums).

### Arguments :id=getalbumsbytag-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `tag` | String | Tag name. |
| `limit` | Number | Maximum number of albums. |

### Return :id=getalbumsbytag-return {docsify-ignore}

`albums` (array) - album search result in Spotify.

### Examples :id=getalbumsbytag-examples {docsify-ignore}

1. Get albums by tag.

```js
let albums = Lastfm.getAlbumsByTag('indie', 40);
```

## getArtistsByTag

Get artists by tag. Parses names from the tag page, [for example](https://www.last.fm/tag/pixie/artists).

### Arguments :id=getartistsbytag-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `tag` | String | Tag name. |
| `limit` | Number | Maximum number of artists. |

### Return :id=getartistsbytag-return {docsify-ignore}

`artists` (array) - artist search result in Spotify.

### Examples :id=getartistsbytag-examples {docsify-ignore}

1. Get artists by tag.

```js
let artists = Lastfm.getArtistsByTag('pixie', 40);
```

## getCustomTop

Get a custom user top.

?> There is a [detailed example](https://github.com/Chimildic/goofy/discussions/91) on the forum showing usage that allows you to get _recommendations from the past_

### Arguments :id=getcustomtop-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `params` | Object | Selection parameters. |

#### Selection Parameters :id=getcustomtop-params {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `user` | Object | Last.fm username. Uses the value from the `config file` by default. |
| `from` | Date/String/Number | Start of period. |
| `to` | Date/String/Number | End of period. |
| `type` | String | Type: `track`, `artist`, or `album`. Defaults to `track`. |
| `count` | Number | Number of items. Defaults to 40. |
| `offset` | Number | Skip first N items. Defaults to 0. |
| `minPlayed` | Number | Minimum play count inclusive. Defaults to 0. |
| `maxPlayed` | Number | Maximum play count inclusive. Defaults to 100 thousand. |
| `isRawItems` | Boolean | When not specified or `false`, searches for items by name in Spotify. If `true`, the result contains lastfm items. `count` and `offset` are ignored. May be useful for custom filtering. Then use the [convertToSpotify](/en/reference/lastfm?id=converttospotify) function. |

### Return :id=getcustomtop-return {docsify-ignore}

`items` (array) - selection result, sorted by play count (when `isRawItems = false`).

A `countPlayed` key with the play count value is added to the result objects.

### Examples :id=getcustomtop-examples {docsify-ignore}

1. Get top 40 tracks for 2015

```js
let topTracks = Lastfm.getCustomTop({
    from: '2015-01-01', // or new Date('2015-01-01'),
    to: '2015-12-31', // or new Date('2015-12-31').getTime(),
});
```

2. Get top 10 artists for the first half of 2014

```js
let topArtists = Lastfm.getCustomTop({
    type: 'artist',
    from: '2014-01-01',
    to: '2014-06-30',
    count: 10,
});
```

## getLibraryStation

Get tracks from the Last.fm _library_ radio station. Contains only scrobbled tracks. Note the warning from [getRecentTracks](/en/reference/lastfm?id=getrecenttracks).

### Arguments :id=getlibrarystation-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `user` | String | Last.fm username. Uses the value from the `config file` by default. |
| `countRequest` | Number | Number of requests to Last.fm. One request yields approximately 20 to 30 tracks. |

### Return :id=getlibrarystation-return {docsify-ignore}

`tracks` (array) - track search result by name in Spotify.

### Examples :id=getlibrarystation-examples {docsify-ignore}

1. Get tracks from the _library_ radio station.

```js
let tracks = Lastfm.getLibraryStation('login', 2);
```

## getLovedTracks

Get Last.fm _likes_. Note the warning from [getRecentTracks](/en/reference/lastfm?id=getrecenttracks). Includes the addition date, which can be used for date filtering.

### Arguments :id=getlovedtracks-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `user` | String | Last.fm username. Uses the value from the `config file` by default. |
| `limit` | Number | Limit the number of tracks. |

### Return :id=getlovedtracks-return {docsify-ignore}

`tracks` (array) - track search result by name in Spotify.

### Examples :id=getlovedtracks-examples {docsify-ignore}

1. Get Last.fm _likes_.

```js
let tracks = Lastfm.getLovedTracks('login', 200);
```

## getMixStation

Get tracks from the Last.fm _mix_ radio station. Contains scrobbled tracks and Last.fm recommendations. Note the warning from [getRecentTracks](/en/reference/lastfm?id=getrecenttracks).

### Arguments :id=getmixstation-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `user` | String | Last.fm username. Uses the value from the `config file` by default. |
| `countRequest` | Number | Number of requests to Last.fm. One request yields approximately 20 to 30 tracks. |

### Return :id=getmixstation-return {docsify-ignore}

`tracks` (array) - track search result by name in Spotify.

### Examples :id=getmixstation-examples {docsify-ignore}

1. Get tracks from the _mix_ radio station.

```js
let tracks = Lastfm.getMixStation('login', 2);
```

## getNeighboursStation

Get tracks from the Last.fm _neighbors_ radio station. Contains tracks listened to by Last.fm users with similar musical tastes. Note the warning from [getRecentTracks](/en/reference/lastfm?id=getrecenttracks).

### Arguments :id=getneighboursstation-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `user` | String | Last.fm username. Uses the value from the `config file` by default. |
| `countRequest` | Number | Number of requests to Last.fm. One request yields approximately 20 to 30 tracks. |

### Return :id=getneighboursstation-return {docsify-ignore}

`tracks` (array) - track search result by name in Spotify.

### Examples :id=getneighboursstation-examples {docsify-ignore}

1. Get tracks from the _neighbors_ radio station.

```js
let tracks = Lastfm.getNeighboursStation('login', 2);
```

## getRecentTracks

Get a user's Last.fm listening history. Privacy must be disabled in the account settings.

### Arguments :id=getrecenttracks-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `user` | String | Last.fm username. Uses the value from the `config file` by default. |
| `count` | Number | Limit the number of tracks. |

### Return :id=getrecenttracks-return {docsify-ignore}

`tracks` (array) - track search result by name in Spotify.

### Examples :id=getrecenttracks-examples {docsify-ignore}

1. Get 200 recently listened tracks.

```js
let tracks = Lastfm.getRecentTracks('login', 200);
```

## getRecomStation

Get tracks from the Last.fm _recommendations_ radio station. Note the warning from [getRecentTracks](/en/reference/lastfm?id=getrecenttracks).

### Arguments :id=getrecomstation-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `user` | String | Last.fm username. Uses the value from the `config file` by default. |
| `countRequest` | Number | Number of requests to Last.fm. One request yields approximately 20 to 30 tracks. |

### Return :id=getrecomstation-return {docsify-ignore}

`tracks` (array) - track search result by name in Spotify.

### Examples :id=getrecomstation-examples {docsify-ignore}

1. Get tracks from the _recommendations_ radio station.

```js
let tracks = Lastfm.getRecomStation('login', 2);
```

## getSimilarArtists

Get similar artists.

### Arguments :id=getsimilarartists-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `items` | Array | Tracks or artists. Only `name` is significant. |
| `match` | Number | Minimum artist similarity value in the range from _0.0_ to _1.0_.  |
| `limit` | Number | Number of similar artists per original artist. |
| `isFlat` | Number | If `false`, the result is grouped by artists. If `true`, all artists are in one array. Defaults to `true`. |

### Return :id=getsimilarartists-return {docsify-ignore}

`artists` (array) - artist search result in Spotify.

### Examples :id=getsimilarartists-examples {docsify-ignore}

1. Get artists similar to followed artists.

```js
let artists = Source.getArtists({ followed_include: true, });
let similarArtists = Lastfm.getSimilarArtists(artists, 0.65, 20);
```

## getSimilarTracks

Get similar tracks.

### Arguments :id=getsimilartracks-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `tracks` | Array | Tracks or artists. Only `name` is significant. |
| `match` | Number | Minimum track similarity value in the range from _0.0_ to _1.0_.  |
| `limit` | Number | Number of similar tracks per original track. |
| `isFlat` | Number | If `false`, the result is grouped by input tracks. If `true`, all tracks are in one array. Defaults to `true`. |

### Return :id=getsimilartracks-return {docsify-ignore}

`tracks` (array) - track search result in Spotify.

### Examples :id=getsimilartracks-examples {docsify-ignore}

1. Get tracks similar to playlist tracks.

```js
let playlistTracks = Source.getPlaylistTracks('name', 'id');
let similarTracks = Lastfm.getSimilarTracks(playlistTracks, 0.65, 30);
```

## getTopAlbums

Get a user's top albums.

### Arguments :id=gettopalbums-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `params` | Object | Selection parameters. Same as [getTopTracks](/en/reference/lastfm?id=gettoptracks) parameters. |

### Return :id=gettopalbums-return {docsify-ignore}

`albums` (array) - album search result in Spotify.

### Examples :id=gettopalbums-examples {docsify-ignore}

1. Get top 10 albums for six months.

```js
let albums = Lastfm.getTopAlbums({
  user: 'login',
  period: '6month',
  limit: 10
});
```

## getTopAlbumsByTag

Get top albums by tag.

### Arguments :id=gettopalbumsbytag-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `params` | Object | Selection parameters. |

#### Selection Parameters :id=gettopalbumsbytag-params {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `tag` | String | Tag name. |
| `limit` | Number | Number of albums per page. Defaults to 50. Can be more, but in this case Last.fm sometimes returns a different quantity. |
| `page` | Number | Page number, used for offsetting through results. Defaults to 1. |

### Return :id=gettopalbumsbytag-return {docsify-ignore}

`albums` (array) - album search result in Spotify.

### Examples :id=gettopalbumsbytag-examples {docsify-ignore}

1. Get albums 51-100 by rock tag

```js
let albums = Lastfm.getTopAlbumsByTag({
    tag: 'rock',
    page: 2,
})
```

## getTopArtists

Get a user's top artists.

### Arguments :id=gettopartists-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `params` | Object | Selection parameters. Same as [getTopTracks](/en/reference/lastfm?id=gettoptracks) parameters. |

### Return :id=gettopartists-return {docsify-ignore}

`artists` (array) - artist search result in Spotify.

### Examples :id=gettopartists-examples {docsify-ignore}

1. Get top 10 artists for six months.

```js
let artists = Lastfm.getTopArtists({
  user: 'login',
  period: '6month',
  limit: 10
});
```

## getTopArtistsByTag

Get top artists by tag.

### Arguments :id=gettopartistsbytag-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `params` | Object | Selection parameters. Same as [getTopAlbumsByTag](/en/reference/lastfm?id=gettopalbumsbytag) parameters. |

### Return :id=gettopartistsbytag-return {docsify-ignore}

`artists` (array) - artist search result in Spotify.

### Examples :id=gettopartistsbytag-examples {docsify-ignore}

1. Get the second ten artists from the indie top.

```js
let artists = Lastfm.getTopArtistsByTag({
    tag: 'indie',
    limit: 10,
    page: 2,
})
```

## getTopTracks

Get a user's top tracks.

### Arguments :id=gettoptracks-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `params` | Object | Selection parameters. |

#### Selection Parameters :id=gettoptracks-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `user` | String | Last.fm username. Uses the value from the `config file` by default. |
| `period` | String | One of the values: _overall_, _7day_, _1month_, _3month_, _6month_, _12month_. |
| `limit` | Number | Limit the number of tracks. |

### Return :id=gettoptracks-return {docsify-ignore}

`artists` (array) - artist search result in Spotify.

### Examples :id=gettoptracks-examples {docsify-ignore}

1. Get top 40 tracks for six months.

```js
let tracks = Lastfm.getTopTracks({
  user: 'your login',
  period: '6month',
  limit: 40
});
```

## getTopTracksByTag

Get top tracks by tag.

### Arguments :id=gettoptracksbytag-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `params` | Object | Selection parameters. Same as [getTopAlbumsByTag](/en/reference/lastfm?id=gettopalbumsbytag) parameters. |

### Return :id=gettoptracksbytag-return {docsify-ignore}

`tracks` (array) - track search result in Spotify.

### Examples :id=gettoptracksbytag-examples {docsify-ignore}

1. Get top 20 by pop tag.

```js
let tracks = Lastfm.getTopTracksByTag({
    tag: 'pop',
    limit: 20,
})
```

## getTracksByTag

Get tracks by tag. Parses names from the tag page, [for example](https://www.last.fm/ru/tag/vocal/tracks?page=1).

### Arguments :id=gettracksbytag-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `tag` | String | Tag name. |
| `limit` | Number | Maximum number of tracks. |

### Return :id=gettracksbytag-return {docsify-ignore}

`tracks` (array) - track search result in Spotify.

### Examples :id=gettracksbytag-examples {docsify-ignore}

1. Get tracks by tag.

```js
let tracks = Lastfm.getTracksByTag('vocal', 40);
```

2. Get tracks by multiple tags.

```js
let tracks = ['rock', 'indie', 'pixie'].reduce((tracks, tag) => 
    Combiner.push(tracks, Lastfm.getTracksByTag(tag, 100)), []);
```

## rangeTags

Filter tracks by tags.

### Arguments :id=rangetags-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `spotifyTracks` | Array | Spotify tracks to check. |
| `params` | Object | Selection parameters. |

### Selection Parameters :id=rangetags-params {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `include` | Array | Tag objects. If present on a track (at least one), it is kept. |
| `exclude` | Array | Tag objects. If present on a track (at least one), it is removed. |
| `isRemoveUnknown` | Boolean | When `true`, tracks without tags are removed. When `false`, they remain. Defaults to `false`. |

### Return :id=rangetags-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=rangetags-examples {docsify-ignore}

1. Keep only rock from recent liked tracks, excluding indie. Since tags are assigned by users, they have a popularity rating (up to 100). `minCount` - minimum value inclusive. If lower, the track is removed despite having the tag.

```js
let tracks = Source.getSavedTracks(20);
Lastfm.rangeTags(tracks, {
  isRemoveUnknown: true,
  include: [
    { name: 'rock', minCount: 10 },
  ],
  exclude: [
    { name: 'indie', minCount: 10 },
  ]
});
```

2. Add tags to tracks, remove unknown.

```js
let tracks = Source.getSavedTracks(20);
Lastfm.rangeTags(tracks, {
  isRemoveUnknown: true,
});
```

## removeRecentArtists

Remove artists based on Last.fm listening history data.

### Arguments :id=removerecentartists-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `original` | Array | Tracks from which to remove items. |
| `user` | String | Last.fm username. Uses the value from the `config file` by default. |
| `count` | Number | Number of tracks from listening history. Defaults to 600. |
| `mode` | String | Artist selection mode. With `every`, checks each artist; with `first`, only the first one (typically the main artist). Defaults to `every`. |

### Return :id=removerecentartists-return {docsify-ignore}

No return value. Modifies the input array.

## removeRecentTracks

Remove tracks based on Last.fm listening history data.

### Arguments :id=removerecenttracks-arguments {docsify-ignore}

| Name | Type | Description |
|-----|-----|----------|
| `original` | Array | Tracks from which to remove items. |
| `user` | String | Last.fm username. Uses the value from the `config file` by default. |
| `count` | Number | Number of tracks from listening history. Defaults to 600. |
| `mode` | String | Artist selection mode. With `every`, checks each artist; with `first`, only the first one (typically the main artist). Defaults to `every`. |

### Return :id=removerecenttracks-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=removerecenttracks-examples {docsify-ignore}

1. Create a playlist with liked tracks that haven't been listened to in the last thousand scrobbles.

```js
let savedTracks = Source.getSavedTracks();
Lastfm.removeRecentTracks(savedTracks, 'login', 1000)
Playlist.saveAsNew({
  name: 'Haven\'t listened in a while',
  tracks: savedTracks,
});
```