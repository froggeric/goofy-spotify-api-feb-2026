# Search

Search methods. Used by other functions. Will only be needed for individual solutions.

### find*

Returns an array with search results by keyword for types: playlist, track, album, artist.

Arguments
- (array) `keywords` - list of keywords, strings only. Each will have its own array in results.
- (number) `requestCount` - number of requests per **one** keyword. One request returns 50 objects if they exist. Maximum 40 requests. Default is 1.

Example 1 - Find 100 playlists by word `rain`

?> The best way would be the [mineTracks](/reference/source?id=minetracks) function. Direct use of the Search module is needed for solutions that are not implemented by default. For example, when [importing tracks from FM radio](https://github.com/Chimildic/goofy/discussions/35).

```js
let keywords = ['rain'];
let playlists = Search.findPlaylists(keywords, 2);
```

### getNoFound

Returns an array with keywords and search type for which no results were found during current script execution.

No arguments.

```js
let noFound = Search.getNoFound();
// structure: { type: '', keyword: '', item: {} }
```

### multisearch*

Returns the best match by track/artist/album keyword

Arguments
- (array) `items` - items to iterate
- (function) `parseNameMethod` - callback called for each item. Should return a string that is the keyword for search.

Example 1 - When the array of items has simple text
```js
let keywords = ['skillet', 'skydive'];
let artists = Search.multisearchArtists(keywords, (i) => i);
```

Example 2 - When the array of items has complex structure
```js
let tracks = Search.multisearchTracks(items, (item) => {
    return `${item.artist} ${item.title}`.formatName();
});
```
