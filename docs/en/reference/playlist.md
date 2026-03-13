# Playlist

Methods for creating and managing playlists.

### getDescription

Returns a string like: `Artist 1, Artist 2... and more`.

Arguments
- (array) `tracks` - tracks from which artists are randomly selected.
- (number) `limit` - number of randomly selected artists. Default is 5.

Example 1 - Create a playlist with description
```js
let tracks = Source.getTracks(playlistArray);
Playlist.saveWithReplace({
    id: 'abcd',
    name: 'Big Daily Mix',
    tracks: tracks,
    description: Playlist.getDescription(tracks),
});
```

### removeTracks

Removes tracks from a playlist.

Arguments
- (string) `id` - playlist id.
- (array) `tracks` - array of tracks.

Example 1 - Remove likes from playlist
```js
let savedTracks = Source.getSavedTracks();
Playlist.removeTracks('id', savedTracks);
```

### saveAsNew

Creates a playlist. A new one each time. Returns the `id` of the created playlist.

Arguments
- (object) `data` - data for creating the playlist.

Data format for creating a playlist
- (string) `name` - playlist name, required.
- (array) `tracks` - array of tracks, required.
- (string) `description` - playlist description. Up to 300 characters.
- (boolean) `public` - if `false` the playlist will be private. Default is `true`.
- (string) `sourceCover` - direct link to cover (up to 256 KB). If specified, `randomCover` is ignored.
- (string) `randomCover` - add a random cover with value `once`. Without use, standard Spotify mosaic.

Example 1 - Create a public playlist with favorite tracks without description with random cover
```js
let tracks = Source.getSavedTracks();
Playlist.saveAsNew({
  name: 'Copy of Favorite Tracks',
  tracks: tracks,
  randomCover: 'once',
  // sourceCover: tracks[0].album.images[0].url,
});
```

Example 2 - Create a private playlist with recent listening history and description without cover.
```js
let tracks = RecentTracks.get(200);
Playlist.saveAsNew({
  name: 'Listening History',
  description: '200 recently listened tracks'
  public: false,
  tracks: tracks,
});
```

### saveWithAppend

Adds tracks to existing ones in the playlist. Updates other data (name, description). If the playlist doesn't exist yet, creates a new one. Returns the `id` of the playlist to which tracks were added.

Arguments
- (object) `data` - playlist data. Playlist data format according to [saveWithReplace](/reference/playlist?id=savewithreplace) description.
- (string) `position` - where to add tracks: beginning `begin` or end `end`. Default is `end`.

Example 1 - Add tracks to the beginning of a playlist.
```js
let tracks = Source.getTracks(playlistArray);
Playlist.saveWithAppend({
    id: 'fewf4t34tfwf4',
    name: 'Daily Mix',
    tracks: tracks
});
```

Example 2 - Add tracks to the end of a playlist, update name and description.
```js
let tracks = Source.getTracks(playlistArray);
Playlist.saveWithAppend({
    id: 'fewf4t34tfwf4',
    name: 'New Name',
    description: 'New description',
    tracks: tracks,
});
```

!> If you update the playlist name without specifying `id`, a new playlist will be created. Because the search won't find a playlist with the new name.

### saveWithReplace

Replaces playlist tracks. Updates other data (name, description). If the playlist doesn't exist yet, creates a new one. Returns the `id` of the playlist to which tracks were added.

Arguments
- (object) `data` - playlist data.

Playlist data format
- (string) `id` - [playlist identifier](#identifier).
- (string) `name` - playlist name, required.
- (array) `tracks` - array of tracks, required.
- (string) `description` - playlist description. Up to 300 characters.
- (boolean) `public` - if `false` the playlist will be private. Default is `true`.
- (string) `sourceCover` - direct link to cover (up to 256 KB). If specified, `randomCover` is ignored.
- (string) `randomCover` - if `once` will add a random cover. With `update` updates the cover each time. Without use, standard Spotify mosaic.

?> It's recommended to always specify `id`. If `id` is not specified, search by name. If such playlist doesn't exist, a new one is created.

Example 1 - Update playlist content and cover
```js
let tracks = Source.getTracks(playlistArray);
Playlist.saveWithReplace({
    id: 'fewf4t34tfwf4',
    name: 'Daily Mix',
    description: 'Playlist description',
    tracks: tracks,
    randomCover: 'update',
    // sourceCover: tracks[0].album.images[0].url,
});
```

Example 2 - Update playlist content from example 1. Search by name.
```js
let tracks = RecentTracks.get();
Playlist.saveWithReplace({
    name: 'History',
    description: 'New playlist description',
    tracks: tracks,
    randomCover: 'update',
});
```

### saveWithUpdate

Updates a playlist or creates a new one. Tracks that are in the array but not in the playlist - are added. Tracks that are not in the array but are in the playlist - are removed. What exists in both - is preserved. Track addition date is not affected. Cannot apply sorting where old and new tracks are mixed (acts like `saveWithAppend`). Returns the `id` of the playlist to which tracks were added.

Arguments
- (object) `data` - playlist data, corresponds to [saveWithReplace](/reference/playlist?id=savewithreplace).

Additional mode for `data`
- (string) `position` - where to add tracks: beginning `begin` or end `end`. Default is `end`.
