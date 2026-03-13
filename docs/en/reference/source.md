# Source

Methods for fetching Spotify elements.

### craftTracks

Returns an array of tracks obtained from [getRecomTracks](/reference/source?id=getrecomtracks) for each group of five elements from the source tracks. Duplicate source tracks are ignored, and duplicates in recommended tracks are removed. The limit of five elements is dictated by the Spotify API for the recommendations function. You can partially influence the formed groups of five elements. Apply one of the `Order` sortings before calling the function.

Arguments
- (array) `tracks` - tracks for which to get recommendations. When `key` is `seed_artists`, an array of artists is acceptable.
- (object) `params` - additional parameters.

Parameter description
- (string) `key` - determines the key for recommendations. Allowed: `seed_tracks` and `seed_artists`. Default is `seed_tracks`.
- (object) `query` - optional parameter, all keys from [getRecomTracks](/reference/source?id=getrecomtracks) are available, except the one specified in `key`.

?> In `query` you can specify two of: `seed_tracks`, `seed_artists`, `seed_genres`. The third is selected based on `key`. This way, you can set a static track/artist/genre (up to 4 values total). The remaining free slots will be filled based on `key`.

Example 1 - Get recommendations for all liked tracks by their artists
```js
let tracks = Source.getSavedTracks();
let recomTracks = Source.craftTracks(tracks, {
    key: 'seed_artists',
    query: {
        limit: 20, // default and maximum 100
        min_energy: 0.4,
        min_popularity: 60,
        // target_popularity: 60,
    }
});
```

Example 2 - Recommendations with a static genre and track specified. The remaining 3 slots are filled with `seed_artists`.
```js
let recomTracks = Source.craftTracks(tracks, {
    key: 'seed_artists',
    query: {
        seed_genres: 'indie',
        seed_tracks: '6FZDfxM3a3UCqtzo5pxSLZ'
    }
});
```

Example 3 - You can specify only an array of tracks. Then recommendations will be by the `seed_tracks` key.
```js
let tracks = Source.getSavedTracks();
let recomTracks = Source.craftTracks(tracks);
```

### getAlbumsTracks

Returns an array of tracks from all albums.

Arguments
- (array) `albums` - list of albums
- (number) `limit` - if specified, tracks are randomly selected up to the specified quantity in each album separately.

Example 1 - Get tracks from top-10 Lastfm albums
```js
let albums = Lastfm.getTopAlbums({ user: 'login', limit: 10 });
let tracks = Source.getAlbumsTracks(albums);
```

### getAlbumTracks

Returns an array of tracks from the specified album.

Arguments
- (object) `album` - a single album object
- (number) `limit` - if specified, tracks are randomly selected up to the specified quantity.

Example 1 - Get tracks from the first album in an array
```js
let albums = Source.getArtistsAlbums(artists, {
    groups: 'album',
});
let albumTracks = Source.getAlbumTracks(albums[0]);
```

Example 2 - Get tracks from all albums
```js
let albums = Source.getArtistsAlbums(artists, {
    groups: 'album',
});
let tracks = [];
albums.forEach((album) => Combiner.push(tracks, Source.getAlbumTracks(album)));
```

### getArtists

Returns an array of artists according to the specified `paramsArtist`.

Arguments
- (object) `paramsArtist` - list of artist selection criteria. The object corresponds to the description from [getArtistsTracks](/reference/source?id=getartiststracks) in the artist section.

Example 1 - Get an array of followed artists
```js
let artists = Source.getArtists({
    followed_include: true,
});
```

### getArtistsAlbums

Returns an array with all albums of the specified artists.

Arguments
- (array) `artists` - array of artists
- (object) `paramsAlbum` - list of album selection criteria. The object corresponds to the description from [getArtistsTracks](/reference/source?id=getartiststracks) in the album section.

Example 1 - Get an array of singles from one artist
```js
let artist = Source.getArtists({
    followed_include: false,
    include: [ 
        { id: 'abc', name: 'Avril' }, 
    ],
});
let albums = Source.getArtistsAlbums(artist, {
    groups: 'single',
    // isFlat: false, // group by artist
});
```

### getArtistsTopTracks

Returns an artist's top tracks as an array. Up to 10 tracks per artist.

Arguments
- (array) `artists` - array of artists. Only `id` is significant.
- (boolean) `isFlat` - if `false`, the result is grouped by artists. If `true`, all tracks are in one array. Default is `true`.

Example 1 - `isFlat = true`
```js
let tracks = Source.getArtistsTopTracks(artists);
tracks[0]; // first track of first artist
tracks[10]; // first track of second artist, if first has 10 tracks
```

Example 2 - `isFlat = false`
```js
let tracks = Source.getArtistsTopTracks(artists, false);
tracks[0][0]; // first track of first artist
tracks[1][0]; // first track of second artist
```

### getArtistsTracks

Returns an array of artist tracks according to the specified `params`.

!> Many albums will be included in the selection. Especially with a large number of followed artists (100+). To reduce execution time, use filters for artists and albums. You can specify a random selection of N elements.

?> There is a bug in the Spotify API. Despite explicitly specifying the country from the token, some albums may be duplicated (for different countries). Use duplicate removal for tracks.

Arguments
- (object) `params` - list of criteria for selecting artists and their tracks

| Key | Type | Description |
|-|-|-|
| isFlat | boolean | When `false`, the result is grouped by artists. Default is `true`. |
| followed_include | boolean | When `true`, includes followed artists. When `false`, artists are taken only from `include` |
| include | array | Selection of artists by `id` to get albums. The `name` key is for convenience and optional.  |
| exclude | array | Selection of artists by `id` to exclude artists from the selection. Use in combination with `followed_include` |
| popularity | object | Artist popularity range |
| followers | object | Artist follower count range |
| genres | array | List of genres. If at least one is present, the artist passes the filter.  |
| ban_genres | array | List of genres to block. If at least one is present, the artist is removed from the selection. |
| groups | string | Album type. Allowed: `album`, `single`, `appears_on`, `compilation` |
| release_date | object | Album release date. Relative period with `sinceDays` and `beforeDays`. Absolute period with `startDate` and `endDate` |
| _limit | number | If specified, a random number of specified elements is selected (artist, album, track) |

Example of `params` object with all keys
```js
{
    isFlat: true,
    artist: {
        followed_include: true,
        popularity: { min: 0, max: 100 },
        followers: { min: 0, max: 100000 },
        artist_limit: 10,
        genres: ['indie'],
        ban_genres: ['rap', 'pop'],
        include: [
            { id: '', name: '' }, 
            { id: '', name: '' },
        ],
        exclude:  [
            { id: '', name: '' }, 
            { id: '', name: '' },
        ],
    },
    album: {
        groups: 'album,single',
        release_date: { sinceDays: 6, beforeDays: 0 },
        // release_date: { startDate: new Date('2020.11.30'), endDate: new Date('2020.12.30') },
        album_limit: 10,
        track_limit: 1,
    }
}
```

Example 1 - Get tracks from singles of followed artists released in the last week including today. Exclude several artists.
```js
let tracks = Source.getArtistsTracks({
    artist: {
        followed_include: true,
        exclude:  [
            { id: 'abc1', name: '' }, 
            { id: 'abc2', name: '' },
        ],
    },
    album: {
        groups: 'single',
        release_date: { sinceDays: 7, beforeDays: 0 },
    },
});
```

Example 2 - Get tracks from albums and singles from the last week of ten randomly selected followed artists. Artists with no more than 10 thousand followers. Only one track per album.
```js
let tracks = Source.getArtistsTracks({
    artist: {
        followed_include: true,
        artist_limit: 10,
        followers: { min: 0, max: 10000 },
    },
    album: {
        groups: 'album,single',
        track_limit: 1,
        release_date: { sinceDays: 7, beforeDays: 0 },
    },
});
```

Example 3 - Get tracks from albums and singles of specified artists
```js
let tracks = Source.getArtistsTracks({
    artist: {
        followed_include: false,
        include:  [
            { id: 'abc1', name: '' }, 
            { id: 'abc2', name: '' },
        ],
    },
    album: {
        groups: 'album,single',
    },
});
```

### getCategoryTracks

Returns an array of tracks from playlists in the specified category. Playlists are sorted by popularity. [List of categories](/reference/desc?id=Playlist-Categories).

Arguments
- (string) `category_id` - category name.
- (object) `params` - additional parameters.

`params` description
- (number) `limit` - limit the number of selected playlists. Maximum 50, default 20.
- (number) `offset` - skip the specified number of tracks. Default 0.
- (string) `country` - country name to view category playlists. For example, `RU` or `AU`.

Example 1 - Get tracks from the second ten playlists in the "focus" category from Australia.
```js
let tracks = Source.getCategoryTracks('focus', { limit: 10, offset: 10, country: 'AU' });
```

Example 2 - Get tracks from 20 playlists in the party category.
```js
let tracks = Source.getCategoryTracks('party');
```

### getFollowedTracks

Returns an array of tracks from followed playlists and/or personal playlists of a specified user.

?> If you need to perform different actions on the source, create a copy of the array using [sliceCopy](/reference/selector?id=slicecopy) instead of making new requests to Spotify via getFollowedTracks.

Arguments
- (object) `params` - playlist selection arguments.

Key description
- (string) `type` - type of playlists to select. Default is `followed`.
- (string) `userId` - [user identifier](#identifier). If not specified, the `userId` of the authorized user is set, i.e., yours.
- (number) `limit` - if used, playlists are selected randomly.
- (array) `exclude` - list of playlists to exclude. Only `id` is significant. The `name` value is optional, only needed to understand which playlist it is. A comment can suffice.
- (boolean) `isFlat` - if `false`, the result is grouped by artists. If `true`, all tracks are in one array (each contains an `origin` key with playlist data). Default is `true`.

|type|Selection|
|-|-|
| owned | Only personal playlists |
| followed | Only followed playlists |
| all | All playlists |

Full `params` object
```js
{
    type: 'followed',
    userId: 'abc',
    limit: 2,
    exclude: [
        { name: 'playlist 1', id: 'abc1' },
        { id: 'abc2' }, // playlist 2
    ],
}
```

Example 1 - Get tracks only from my followed playlists.
```js
// All default values, arguments not specified
let tracks = Source.getFollowedTracks();

// Same thing with explicit playlist type specification
let tracks = Source.getFollowedTracks({
    type: 'followed',
});
```

Example 2 - Get tracks from only two randomly selected personal playlists of user `example`, excluding several playlists by their id. 
```js
let tracks = Source.getFollowedTracks({
    type: 'owned',
    userId: 'example',
    limit: 2, 
    exclude: [
        { id: 'abc1' }, // playlist 1
        { id: 'abc2' }, // playlist 2
    ],
});
```

!> You should avoid users with too many playlists. For example, `glennpmcdonald` with almost 5 thousand playlists. The limitation is due to the execution quota in Apps Script. There won't be enough time to get that volume of tracks. See [limitations description](/details?id=Limitations) for details.


### getListCategory

Returns an array of categories available for [getCategoryTracks](/reference/source?id=getcategorytracks).

Arguments
- (object) `params` - category selection parameters.

`params` description
- (number) `limit` - limit the number of selected categories. Maximum 50, default 20.
- (number) `offset` - skip the specified number of categories. Default 0. Used for getting categories after 50+.
- (string) `country` - country name to view categories. For example, `RU` or `AU`. If not specified, globally available ones. But availability errors are possible. To avoid errors, specify the same `country` for the category list and playlist requests. 

Example 1 - Get tracks from 10 playlists from a random category
```js
let listCategory = Source.getListCategory({ limit: 50, country: 'RU' });
let category = Selector.sliceRandom(listCategory, 1);
let tracks = Source.getCategoryTracks(category[0].id, { limit: 10, country: 'RU' });
```

### getPlaylistTracks

Returns an array of tracks from a single playlist. Similar to [getTracks](/reference/source?id=gettracks) with one playlist.

Arguments
- (string) `name` - playlist name.
- (string) `id` - [playlist identification number](/reference/desc?id=Playlist).
- (string) `user` - [user identification number](/reference/desc?id=User). Default is yours.
- (number) `count` - number of tracks to select.
- (boolean) `inRow` - selection mode. If the key is missing or `true`, select the first `count` elements, otherwise random selection

Example 1 - Get tracks from a single playlist
```js
let tracks = Source.getPlaylistTracks('Blocked tracks', 'abcdef');
```

Example 2 - Random selection of 10 tracks
```js
// playlist name and user can be left empty if the playlist id is known
let tracks = Source.getPlaylistTracks('', 'id', '', 10, false);
```

### getRecomArtists

Returns an array of recommended artists based on specified parameters. This function is intended to replace [getRelatedArtists](/reference/source?id=getrelatedartists) due to errors from Spotify's updated policy. Internally uses [getRecomTracks](/reference/source?id=getrecomtracks) with artist seed (artists are extracted from recommended tracks).

Arguments
- (array) `artists` - list of artists to get similar ones for. Only `id` is significant.
- (object) `queryObj` - parameters for recommendation selection. Corresponds to [getRecomTracks](/reference/source?id=getrecomtracks).
- (boolean) `isFlat` - if `false`, the result is grouped by artists. If `true`, all artists are in one array. Default is `true`.

Example 1 - Replacement for `getRelatedArtists`

```js
let followedArtists = Source.getArtists({ followed_include: true })
Selector.keepFirst(followedArtists, 5)

// Before: let recomArtists = Source.getRelatedArtists(followedArtists)
let recomArtists = Source.getRecomArtists(followedArtists) 
```

Example 2 - Recommendations with mixed seed
```js
let followedArtists = Source.getArtists({ followed_include: true })
Selector.keepFirst(followedArtists, 5)

let recomArtists = Source.getRecomArtists(followedArtists, {
  isFullArtist: false, // artist from track has shortened data, if you need popularity, genre, follower count, specify true
  limit: 10, // default 50, maximum 100
  seed_tracks: '3nzL5CIQiCEt6jRt1AlQ9d', // you can embed an additional track seed so Spotify gives a recommendation close to both the track and the artist
  min_valence: 0.65, // track features are also supported, but have less significance since the result takes the artist, not a specific track
}) 
```

### getRecomTracks

Returns an array of recommended tracks based on specified parameters (up to 100 tracks). For new or lesser-known artists/tracks, there may not be enough accumulated data to generate recommendations. 

Arguments
- (object) `queryObj` - parameters for recommendation selection.

Valid parameters
- limit - number of tracks. Maximum 100.
- seed_* - up to **5 values** in any combination:
  - seed_artists - [artist identifiers](/reference/desc?id=Identifier), separated by comma.
  - seed_tracks - [track identifiers](/reference/desc?id=Identifier), separated by comma.
  - seed_genres - genres, separated by comma. See allowed values [here](/reference/desc?id=Genres-for-Recommendation-Selection).
- max_* - maximum value of one of the [track features](/reference/desc?id=Track-Features).
- min_* - minimum value of one of the [track features](/reference/desc?id=Track-Features).
- target_* - target value of one of the [track features](/reference/desc?id=Track-Features). The closest values are selected.

?> Additionally, the `populatiry` key is available in `features`. For example, `target_popularity`. This is hidden in the Spotify API documentation.

!> When specifying a specific genre in `seed_genres`, tracks of that genre may not necessarily be returned. Such a seed is a starting point for recommendations.

Example object with parameters
```js
let queryObj = {
    seed_artists: '',
    seed_genres: '',
    seed_tracks: '',
    max_*: 0,
    min_*: 0,
    target_*: 0,
};
```

Example 1 - Get recommendations by indie and alternative genres with positive mood:
```js
let tracks = Source.getRecomTracks({
    seed_genres: 'indie,alternative',
    min_valence: 0.65,
});
```

Example 2 - Get recommendations in rock and electronic genres based on 3 random liked artists (up to 5 values).
```js
let savedTracks = Source.getSavedTracks();
Selector.keepRandom(savedTracks, 3);

let artistIds = savedTracks.map(track => track.artists[0].id);

let tracks = Source.getRecomTracks({
    seed_artists: artistIds.join(','),
    seed_genres: 'rock,electronic'
});
```

Example 3 - Substituting artist genres. In all `craftTracks` requests, randomly selected genres will be used once. On the next run, different ones.
```js
let artists = Source.getArtists({ followed_include: false });
let genres = Array.from(new Set(artists.reduce((genres, artist) => {
  return Combiner.push(genres, artist.genres);
}, [])));
let recomTracks = Source.craftTracks(artists, {
  key: 'seed_artists',
  query: {
    seed_genres: Selector.sliceRandom(genres, 3).join(','),
    min_popularity: 20,
  }
});
```

### getRelatedArtists

Returns an array of similar artists according to Spotify data.

Arguments
- (array) `artists` - list of artists to get similar ones for. Only `id` is significant.
- (boolean) `isFlat` - if `false`, the result is grouped by artists. If `true`, all artists are in one array. Default is `true`.

Example 1 - `isFlat = true`
```js
let relatedArtists = Source.getRelatedArtists(artists);
relatedArtists[0]; // first artist
relatedArtists[10]; // 11th artist
```

Example 2 - `isFlat = false`
```js
let relatedArtists = Source.getRelatedArtists(artists, false);
relatedArtists[0][0]; // first artist, similar to the first from source
relatedArtists[1][0]; // first artist, similar to the second from source
```

### getRecentReleasesByArtists

Returns recent releases within the specified artists and time period.

Arguments
- (object) `params` - selection parameters
  - (array) `artists` - artists of the searched albums. Only `id` is significant for each element.
  - (object) `date` - relative or absolute time period (`sinceDays` and `beforeDays` or `startDate` and `endDate`).
  - (array) `type` - allowed album type (`single`, `album`).
  - (boolean) `isFlat` - if `false`, the result is grouped by artists. If `true`, all tracks are in one array. Default is `true`.

Example 1 - Releases from followed artists in the last week
```js
let weekReleases = Source.getRecentReleasesByArtists({
  artists: Source.getArtists({ followed_include: true }),
  date: { sinceDays: 7, beforeDays: 0 },
  type: ['album', 'single'],
});
// When albums and singles are selected - duplicate tracks may appear, especially with a large date range
Filter.dedupTracks(weekReleases)
```

### getSavedAlbums

Returns an array of saved albums, each containing up to 50 tracks. Albums are sorted from recent to old by save date.

No arguments.

Example 1 - Get tracks from the last saved album
```js
let albums = Source.getSavedAlbums();
let tracks = albums[0].tracks.items;
```

### getSavedAlbumTracks

Returns an array of tracks from all saved albums. You can specify random album selection.

Arguments:
- (number) `limit` - if used, albums are randomly selected up to the specified value.

Example 1 - Get tracks from three random albums
```js
let tracks = Source.getSavedAlbumTracks(3);
```

Example 2 - Get tracks from all saved albums
```js
let tracks = Source.getSavedAlbumTracks();
```

### getSavedTracks

Returns an array of liked tracks. Sorted from new to old. If likes were added by automatic means, the date is the same. Therefore, the final array sorting may differ from the Spotify interface.

?> To reduce the number of requests, use `Cache.read('SavedTracks.json')`. The cache is updated daily.

Argument
- (number) `limit` - if specified, limits the number of tracks and requests to get them. When not specified, all are returned.

?> If you have many liked tracks and need to perform different actions on them in a script, create a copy of the array using [sliceCopy](/reference/selector?id=slicecopy) instead of making new requests to Spotify.

Example 1 - Get an array of liked tracks.
```js
let tracks = Source.getSavedTracks();
```

Example 2 - Get the last 5 likes.
```js
let tracks = Source.getSavedTracks(5);
```

### getTopArtists

Returns top artists for the selected period. Up to 98 artists. 

Arguments
- (string) `timeRange` - period. Default is `medium`. Possible values are given in [getTopTracks](/reference/source?id=gettoptracks).

Example 1 - Get top tracks from top 10 artists
```js
let artists = Source.getTopArtists('long');
Selector.keepFirst(artists, 10);
let tracks = Source.getArtistsTopTracks(artists);
```

### getTopTracks

Returns an array of tracks with top plays for the selected period. Up to 98 tracks.

Arguments
- (string) `timeRange` - period. Default is `medium`.

|timeRange|Period|
|-|-|
| short | Approximately the last month |
| medium | Approximately the last 6 months |
| long | Over several years |

!> Such tracks do not contain information about the date added. When using [rangeDateRel](/reference/filter?id=rangedaterel) or [rangeDateAbs](/reference/filter?id=rangedateabs), they are assigned the date 01.01.2000

Example 1 - Get top tracks for the last month.
```js
let tracks = Source.getTopTracks('short');
```

Example 2 - Get top tracks over several years.
```js
let tracks = Source.getTopTracks('long');
```

### getTracks

Returns an array of tracks from one or more playlists.

Arguments
- (array) `playlistArray` - one or more playlists. 

Format of *one* playlist
- `id` - [playlist identification number](/reference/desc?id=Playlist).
- `userId` - [user identification number](/reference/desc?id=User).
- `name` - playlist name.
- `count` - number of tracks to select.
- `inRow` - selection mode. If the key is missing or `true`, select the first `count` elements, otherwise random selection

| id | name | userId | Action |
|:-:|:-:|:-:|:-|
| ✓ | ☓ | ☓ | Take playlist with specified id |
| ☓ | ✓ | ☓ | Search playlist by name among yours |
| ☓ | ✓ | ✓ | Search playlist by name from user |

?> It is recommended to always specify `id` and `name`. This is the fastest and most convenient way.

!> If `name` is specified without `id` and there are multiple playlists with that name, tracks from the first one found will be returned. When a playlist is not found, an empty array is returned.

Example 1 - Get tracks from two playlists by `id`. The `name` value is optional. Specified for convenience.
```js
let tracks = Source.getTracks([
  { name: 'Top Hits', id: '37i9dQZF1DX12G1GAEuIuj' },
  { name: 'Cardio', id: '37i9dQZF1DWSJHnPb1f0X3' },
]);
```

Example 2 - Get tracks from personal playlists The Best and Soundtracks.
```js
let tracks = Source.getTracks([
  { name: 'The Best' },
  { name: 'Soundtracks' },
]);
```

Example 3 - Get tracks from a playlist named mint from user spotify.
```js
let tracks = Source.getTracks([
  { name: 'mint', userId: 'spotify' },
]);
```

### getTracksRandom

Returns an array of tracks from one or more playlists. Playlists are selected randomly. 

Arguments
- (array) `playlistArray` - one or more playlists. Same as [getTracks](/reference/source?id=gettracks).
- (number) `countPlaylist` - number of randomly selected playlists. Default is one.

Example 1 - Get tracks from one randomly selected playlist out of three.
```js
let tracks = Source.getTracksRandom([
  { name: 'Top Hits', id: '37i9dQZF1DX12G1GAEuIuj' },
  { name: 'Cardio', id: '37i9dQZF1DWSJHnPb1f0X3' },
  { name: 'Dark Side', id: '37i9dQZF1DX73pG7P0YcKJ' },
]);
```

Example 2 - Get tracks from two randomly selected playlists out of three.
```js
let playlistArray = [
  { name: 'Top Hits', id: '37i9dQZF1DX12G1GAEuIuj' },
  { name: 'Cardio', id: '37i9dQZF1DWSJHnPb1f0X3' },
  { name: 'Dark Side', id: '37i9dQZF1DX73pG7P0YcKJ' },
];
let tracks = Source.getTracksRandom(playlistArray, 2);
```

### mineTracks

Returns an array of tracks found when searching for playlists, albums, or tracks by keywords. Duplicates are removed from the result.

Arguments
- (object) `params` - search parameters.

`params` description
- (string) `type` - search type. Allowed: `playlist`, `album`, `track`. Default is `playlist`. When `track`, you can use [advanced search](https://support.spotify.com/by-ru/article/search/).
- (array) `keyword` - list of keywords for searching elements.
- (number) `requestCount` - number of requests per keyword. One request returns 50 elements, if available. Maximum 40 requests. Default is one.
- (number) `itemCount` - number of elements to select from all found per keyword. Default is three.
- (number) `skipCount` - number of elements to skip from the beginning per keyword. Default is zero. 
- (boolean) `inRow` - if not specified or `false`, elements are selected randomly. If `true`, the first `N` elements are taken (by `itemCount` value).
- (number) `popularity` - minimum track popularity value. Default is zero.
- (object) `followers` - playlist follower count range (inclusive boundaries). Filter before `itemCount` selection. Use only with small `requestCount` when `type = playlist`. 

!> You must maintain a balance of values in `params`. Several large values can take a long time to execute and make many requests. Find acceptable combinations in practice.

> You can display the number of requests made. Add this line at the end of the function: 
> `console.log('Number of requests', CustomUrlFetchApp.getCountRequest());`

Example 1 - Select 5 random playlists for each keyword with track popularity from 70. With limited playlist follower count.
```js
let tracks = Source.mineTracks({
    keyword: ['synth', 'synthpop', 'rock'],
    followers: { min: 2, max: 1000 },
    itemCount: 5,
    requestCount: 3,
    popularity: 70,
});
```

Example 2 - Select 10 first playlists by keyword with any track popularity
```js
let tracks = Source.mineTracks({
    keyword: ['indie'],
    itemCount: 10,
    inRow: true,
});
```

Example 3 - Select tracks from random albums
```js
let tracks = Source.mineTracks({
    type: 'album',
    keyword: ['winter', 'night'],
});
```

Example 4 - Select tracks in the indie genre from 2020
```js
let tracks = Source.mineTracks({
    type: 'track',
    keyword: ['genre:indie + year:2020'],
});
```