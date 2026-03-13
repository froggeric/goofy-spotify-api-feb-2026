# Filter

Methods for filtering elements.

| Method | Result Type | Brief Description |
|-------|-------------|-------------------|
| [dedupArtists](/reference/filter?id=dedupartists) | - | Remove duplicate artists. |
| [dedupTracks](/reference/filter?id=deduptracks) | - | Remove duplicate tracks. |
| [getDateRel](/reference/filter?id=getdaterel) | Date | Calculate date by day offset relative to today. |
| [detectLanguage](/reference/filter?id=detectlanguage) | - | Determine the main language of track lyrics by text. |
| [getLastOutRange](/reference/filter?id=getlastoutrange) | Array | Get tracks that did not pass the last [rangeTracks](/reference/filter?id=rangetracks) filter. |
| [match](/reference/filter?id=match) | - | Remove tracks that do not match the regular expression. |
| [matchExcept](/reference/filter?id=matchexcept) | - | Wrapper for [match](/reference/filter?id=match) with inversion. |
| [matchExceptMix](/reference/filter?id=matchexceptmix) | - | Remove tracks containing the words _mix_ and _club_. |
| [matchExceptRu](/reference/filter?id=matchexceptru) | - | Remove tracks containing Cyrillic characters in the title. |
| [matchLatinOnly](/reference/filter?id=matchlatinonly) | - | Remove all tracks except those containing Latin characters in the title. |
| [matchOriginalOnly](/reference/filter?id=matchoriginalonly) | - | Remove non-original track versions. |
| [rangeDateAbs](/reference/filter?id=rangedateabs) | - | Select elements within a period by absolute addition or listening dates. |
| [rangeDateRel](/reference/filter?id=rangedaterel) | - | Select elements within a period by relative addition or listening dates. |
| [rangeTracks](/reference/filter?id=rangetracks) | - | Select tracks within metadata ranges. |
| [removeArtists](/reference/filter?id=removeartists) | - | Exclude artists from an array. |
| [removeTracks](/reference/filter?id=removetracks) | - | Exclude tracks from an array. |
| [removeUnavailable](/reference/filter?id=removeunavailable) | - | Exclude tracks that are not available for playback. |
| [replaceWithSimilar](/reference/filter?id=replacewithsimilar) | - | Replace tracks with similar ones. |

## dedupArtists

Remove duplicate artists by _id_. Only one element per artist remains.

### Arguments :id=dedupartists-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `items` | Array | Tracks or artists from which to remove duplicate artists. |

### Return :id=dedupartists-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=dedupartists-examples {docsify-ignore}

1. Remove duplicate artists from tracks.

```js
let tracks = Source.getTracks(playlistArray);
Filter.dedupArtists(tracks);
```

2. Remove duplicate artists from an artist array.

```js
let relatedArtists = Source.getRelatedArtists(artists);
Filter.dedupArtists(relatedArtists);
```

## dedupTracks

Remove duplicate tracks by _id_ and _name_.

### Arguments :id=deduptracks-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `tracks` | Array | Tracks from which to remove duplicates. |
| `offsetDurationMs` | Number | Deviation in milliseconds at which identically named tracks are considered the same. Default is 2000 milliseconds (2 seconds). [More details](https://github.com/Chimildic/goofy/discussions/116). |

### Return :id=deduptracks-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=deduptracks-examples {docsify-ignore}

1. Remove duplicates. Specifying the deviation is optional.

```js
let tracks = Source.getTracks(playlistArray);
Filter.dedupTracks(tracks);
```

## getDateRel

Calculate date by day offset relative to today.

### Arguments :id=getdaterel-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `days` | Number | Offset in days relative to today. |
| `bound` | String | Time boundary. With `startDay` it's 00:00, with `endDay` it's 23:59. If not specified, the value remains from the moment the method is executed. |

### Return :id=getdaterel-return {docsify-ignore}

`date` - the calculated date after the offset.

### Examples :id=getdaterel-examples {docsify-ignore}

Example in the [favorites and forgotten](/template?id=Любимо-и-забыто) template.

## detectLanguage

Determine the main language of track lyrics by text.

!> Requires setting the `MUSIXMATCH_API_KEY` parameter, [more details](/config). The service limits the number of requests per day. Use the function only after reducing the track array with other filters.

### Arguments :id=detectlanguage-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `tracks` | Array | Tracks for which to determine the main language of lyrics. |
| `params` | Object | Filter parameters. |

#### Filter Parameters :id=detectlanguage-params {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `isRemoveUnknown` | Boolean | Action when language is unknown (when not in the `musixmatch` database or it's instrumental/dance). When `true`, removes such tracks; when `false`, keeps them. Default is `false`. |
| `include` | Array | Languages to keep. |
| `exclude` | Array | Languages to remove. |

?> Two-letter language codes in lowercase are accepted. [List of languages](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes).

### Return :id=detectlanguage-return {docsify-ignore}

No return value. Modifies the input array when filter parameters are specified.

A `lyrics` object is added to tracks, containing the language code `lang` and a short text excerpt `text`.

### Examples :id=detectlanguage-examples {docsify-ignore}

1. Take the [Germany Top 50](https://open.spotify.com/playlist/37i9dQZEVXbJiZcmkrIHGU?si=33fdf90a2b854fc8) and keep only German tracks.

```js
let tracks = Source.getPlaylistTracks('', '37i9dQZEVXbJiZcmkrIHGU');
Filter.detectLanguage(tracks, {
  isRemoveUnknown: true,
  include: ['de'],
});
```

2. Similar situation, exclude Russian.

```js
let tracks = Source.getPlaylistTracks('', '37i9dQZEVXbL8l7ra5vVdB');
Filter.detectLanguage(tracks, {
  isRemoveUnknown: true,
  exclude: ['ru'],
});
```

3. Find out which languages are in the array.

```js
let tracks = Source.getPlaylistTracks('', '37i9dQZEVXbL8l7ra5vVdB');
Filter.detectLanguage(tracks, { isRemoveUnknown: true });
console.log(Array.from(new Set(tracks.map(t => t.lyrics.lang))).join('\n'));
```

?> For a set of tracks in different languages, the [mineTracks](/reference/source?id=minetracks) search function is suitable.

## getLastOutRange

Get tracks that did not pass the last [rangeTracks](/reference/filter?id=rangetracks) filter.

### Arguments :id=getlastoutrange-arguments {docsify-ignore}

No arguments.

### Return :id=getlastoutrange-return {docsify-ignore}

`outRangeTracks` (array) - tracks that did not pass the filter.

### Examples :id=getlastoutrange-examples {docsify-ignore}

1. Get tracks that did not pass the filter.

```js
let tracks = Source.getTracks(playlistArray);
Filter.rangeTracks(tracks, args);
let outRangeTracks = Filter.getLastOutRange();
```

## match

Remove tracks that do not match the regular expression for track name, album, and artist.

### Arguments :id=match-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `items` | Array | Tracks or artists to check against the regular expression. |
| `strRegex` | String | The regular expression value. |
| `invert` | Boolean | If `true`, inverts the result. Default is `false`. |

### Return :id=match-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=match-examples {docsify-ignore}

1. Remove tracks containing the words _cover_ or _live_ in their title.

```js
let tracks = Source.getTracks(playlistArray);
Filter.match(tracks, 'cover|live', true);
```

## matchExcept

Wrapper for [match](/reference/filter) with `invert = true`.

### Arguments :id=matchexcept-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `items` | Array | Tracks or artists to check against the regular expression. |
| `strRegex` | String | The regular expression value. |

### Return :id=matchexcept-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=matchexcept-examples {docsify-ignore}

1. Remove tracks containing the words _cover_ or _live_ in their title.

```js
let tracks = Source.getTracks(playlistArray);
Filter.matchExcept(tracks, 'cover|live');
```

## matchExceptMix

Remove tracks containing the words _mix_ and _club_. Wrapper for [matchExcept](/reference/filter?id=matchexcept) with argument `strRegex = 'mix|club'`.

### Arguments :id=matchexceptmix-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `tracks` | Array | Tracks to check against the regular expression. |

### Return :id=matchexceptmix-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=matchexceptmix-examples {docsify-ignore}

1. Remove tracks containing the words _mix_ or _club_ in their title.

```js
let tracks = Source.getTracks(playlistArray);
Filter.matchExceptMix(tracks);
```

## matchExceptRu

Remove tracks containing Cyrillic characters in the title. Wrapper for [matchExcept](/reference/filter?id=matchexcept) with argument `strRegex = '[а-яА-ЯёЁ]+'`.

?> For filtering by track language, use [detectLanguage](/reference/filter?id=detectlanguage).

### Arguments :id=matchexceptru-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `tracks` | Array | Tracks to check against the regular expression. |

### Return :id=matchexceptru-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=matchexceptru-examples {docsify-ignore}

1. Remove tracks with Cyrillic characters in the title.

```js
let tracks = Source.getTracks(playlistArray);
Filter.matchExceptRu(tracks);
```

## matchLatinOnly

Remove all tracks except those containing Latin characters in the title. Wrapper for [match](/reference/filter?id=match) with argument `strRegex = '^[a-zA-Z0-9 ]+$'`.

?> For filtering by track language, use [detectLanguage](/reference/filter?id=detectlanguage).

### Arguments :id=matchlatinonly-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `tracks` | Array | Tracks to check against the regular expression. |

### Return :id=matchlatinonly-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=matchlatinonly-examples {docsify-ignore}

1. Keep only tracks with Latin characters in the title.

```js
let tracks = Source.getTracks(playlistArray);
Filter.matchLatinOnly(tracks);
```

## matchOriginalOnly

Remove non-original track versions. Wrapper for [matchExcept](/reference/filter?id=matchexcept) with argument `strRegex = 'mix|club|radio|piano|acoustic|edit|live|version|cover|karaoke'`.

### Arguments :id=matchoriginalonly-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `tracks` | Array | Tracks to check against the regular expression. |

### Return :id=matchoriginalonly-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=matchoriginalonly-examples {docsify-ignore}

1. Keep only original track versions.

```js
let tracks = Source.getTracks(playlistArray);
Filter.matchOriginalOnly(tracks);
```

## rangeDateAbs

Select elements within a period by absolute dates. Addition or listening date is checked.

?> For filtering by album release date, use [rangeTracks](/reference/filter?id=rangetracks).

### Arguments :id=rangedateabs-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `items` | Array | Elements to check. |
| `startDate` | Date | Start of the absolute period. |
| `endDate` | Date | End of the absolute period. |

Date format `YYYY-MM-DDTHH:mm:ss.sss`, where:
- `YYYY-MM-DD` - year, month, day
- `T` - separator for specifying time. Include if adding time.
- `HH:mm:ss.sss` - hours, minutes, seconds, milliseconds

### Return :id=rangedateabs-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=rangedateabs-examples {docsify-ignore}

1. Tracks added between September 1st and 3rd.

```js
let tracks = Source.getTracks(playlistArray);
let startDate = new Date('2020-09-01');
let endDate = new Date('2020-09-03');
Filter.rangeDateAbs(tracks, startDate, endDate);
```

2. Tracks added from August 1st 15:00 to August 20th 10:00.

```js
let tracks = Source.getTracks(playlistArray);
let startDate = new Date('2020-08-01T15:00');
let endDate = new Date('2020-08-20T10:00');
Filter.rangeDateAbs(tracks, startDate, endDate);
```

3. Tracks added from September 1st to the current date and time.

```js
let tracks = Source.getTracks(playlistArray);
let startDate = new Date('2020-09-01');
let endDate = new Date();
Filter.rangeDateAbs(tracks, startDate, endDate);
```

## rangeDateRel

Select elements within a period by relative dates. Addition or listening date is checked.

!> If an element does not contain a date, it is set to 01.01.2000. This is possible, for example, if the track was added to Spotify very long ago, the source is [getTopTracks](/reference/source?id=gettoptracks), these are "My Daily Mix #N" playlists, or a number of other sources.

### Arguments :id=rangedaterel-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `items` | Array | Elements to check. |
| `sinceDays` | Number | Start of the relative period. |
| `beforeDays` | Number | End of the relative period. |

The diagram shows an example for `sinceDays = 7` and `beforeDays = 2`. That is, get elements added to the playlist from September 3rd 00:00 to September 8th 23:59 relative to today, September 10th.

![Example of using sinceDays and beforeDays](../img/DaysRel.png ':size=60%')

?> The mechanism for obtaining the artist tracking start date is described [on the forum](https://github.com/Chimildic/goofy/discussions/98)

### Return :id=rangedaterel-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=rangedaterel-examples {docsify-ignore}

1. Tracks added in the last 5 days and today.

```js
let tracks = Source.getTracks(playlistArray);
Filter.rangeDateRel(tracks, 5);
// equivalent to Filter.rangeDateRel(tracks, 5, 0);
```

2. Tracks from the last 7 days excluding today.

```js
let tracks = Source.getTracks(playlistArray);
Filter.rangeDateRel(tracks, 7, 1);
```

3. Tracks for a single day that was 14 days ago.

```js
let tracks = Source.getTracks(playlistArray);
Filter.rangeDateRel(tracks, 14, 14);
```

4. Tracks for today only.
```js
let tracks = Source.getTracks(playlistArray);
Filter.rangeDateRel(tracks);
// equivalent to Filter.rangeDateRel(tracks, 0, 0);
```

## rangeTracks

Select tracks within metadata ranges.

### Arguments :id=rangetracks-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `tracks` | Array | Tracks to check. |
| `params` | Object | Selection parameters. |

!> The function requests additional data. To reduce the number of requests, use it after reducing the track array by other means (e.g., [rangeDateRel](/reference/filter?id=rangedaterel), [match](/reference/filter?id=match), etc.). Retrieved data is cached for the **current** execution. Repeated calls to the function or [sort](/reference/order?id=sort) with the same categories do not send new requests.

#### Selection Parameters :id=rangetracks-params {docsify-ignore}

Below is an example of the `params` object with all valid check conditions. [Parameter description](/reference/desc?id=Описание-параметров-объектов).

```js
let params = {
    meta: {
        popularity: { min: 0, max: 100 },
        duration_ms: { min: 0, max: 10000 },
        explicit: false,
    },
    artist: {
        popularity: { min: 0, max: 100 },
        followers: { min: 0, max: 100000 },
        genres: ['indie'],
        ban_genres: ['rap', 'pop'],
        isRemoveUnknownGenre: false,
    },
    features: {
        acousticness: { min: 0.0, max: 1.0 },
        danceability: { min: 0.0, max: 1.0 },
        energy: { min: 0.0, max: 1.0 },
        instrumentalness: { min: 0.0, max: 1.0 },
        liveness: { min: 0.0, max: 1.0 },
        loudness: { min: -60, max: 0 },
        speechiness: { min: 0.0, max: 1.0 },
        valence: { min: 0.0, max: 1.0 },
        tempo: { min: 30, max: 210 },
        key: 0,
        mode: 0,
        time_signature: 1,

        // calculated https://github.com/Chimildic/goofy/discussions/87
        anger: { min: 0.0, max: 1.0 },
        happiness: { min: 0.0, max: 1.0 },
        sadness: { min: 0.0, max: 1.0 }, 
        
        // duplicates args.meta.duration_ms, one is enough (choice depends on category)
        duration_ms: { min: 0, max: 10000 },
    },
    album: {
        popularity: { min: 30, max: 70 },
        album_type: ['single', 'album'],
        release_date: { sinceDays: 6, beforeDays: 0 },
        // or release_date: { startDate: new Date('2020.11.30'), endDate: new Date('2020.12.30') },
    },
};
```

### Return :id=rangetracks-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=rangetracks-examples {docsify-ignore}

1. Remove rap genre tracks.

```js
let tracks = Source.getTracks(playlistArray);
Filter.rangeTracks(tracks, {
    artist: {
      ban_genres: ['rap'],
    }
});
```

2. Select tracks in indie and alternative genres.

```js
let tracks = Source.getTracks(playlistArray);
Filter.rangeTracks(tracks, {
    artist: {
        genres: ['indie', 'alternative'],
    },
});
```

3. Select low-popularity tracks from lesser-known artists.

```js
let tracks = Source.getTracks(playlistArray);
Filter.rangeTracks(tracks, {
    meta: {
      popularity: { min: 0, max: 49 },
    },
    artist: {
      followers: { min: 0, max: 9999 },
    },
});
```

## removeArtists

Exclude artists from an array. Match is determined by track artist _id_.

### Arguments :id=removeartists-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `original` | Array | Tracks or artists to check. |
| `removable` | Array | Tracks or artists to exclude. |
| `invert` | Boolean | Invert the result. If `true`, remove all except `removable`. Default is `false`. |
| `mode` | String | Artist selection mode. With `every`, checks all artists; with `first`, only the first (typically the main artist). Default is `every`. |

### Return :id=removeartists-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=removeartists-examples {docsify-ignore}

1. Get playlist tracks and exclude liked track artists.

```js
let sourceArray = Source.getTracks(playlistArray);
let removedArray = Source.getSavedTracks();
Filter.removeArtists(sourceArray, removedArray);
```

## removeTracks

Exclude tracks from an array. Match is determined by track _id_ and track name together with artist.

?> There is a possibility of encountering the [relink issue](https://github.com/Chimildic/goofy/discussions/99) when removing listening history.

### Arguments :id=removetracks-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `original` | Array | Tracks to check. |
| `removable` | Array | Tracks to exclude. |
| `invert` | Boolean | Invert the result. If `true`, remove all except `removable`. Default is `false`. |
| `mode` | String | Artist selection mode. With `every`, checks all artists; with `first`, only the first (typically the main artist). Default is `every`. |

### Return :id=removetracks-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=removetracks-examples {docsify-ignore}

1. Get playlist tracks and exclude liked tracks.

```js
let sourceArray = Source.getTracks(playlistArray);
let removedArray = Source.getSavedTracks();
Filter.removeTracks(sourceArray, removedArray);
```

## removeUnavailable

Exclude tracks that are not available for playback. Does not replace the original with an alternative. That is, there is no redirection to another track, see the [relink issue](https://github.com/Chimildic/goofy/discussions/99) for details. Makes additional requests (1 per 50 tracks) when a track is in an indeterminate state.

?> The filter can be applied to tracks from `Cache` that have passed the [compressTracks](/reference/cache?id=compresstracks) method. If the method was not applied, the state is determined by the value in the cached track.

### Arguments :id=removeunavailable-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `tracks` | Array | Tracks to check. |
| `market` | String | Country in which track availability is checked. Default is the account's country. |

### Return :id=removeunavailable-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=removeunavailable-examples {docsify-ignore}

1. Remove tracks unavailable in Russia from a playlist.

```js
let tracks = Source.getPlaylistTracks('', 'id');
Filter.removeUnavailable(tracks, 'RU');
```

## replaceWithSimilar

Replace tracks with similar ones. N random tracks from [getRecomTracks](/reference/source?id=getrecomtracks) results for each replacement. Recommendations are requested with `target_*` parameters from the original track. When there is no replacement, the track is removed.

### Arguments :id=replacewithsimilar-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `params` | Object | Replacement parameters. |

### Replacement Parameters :id=replacewithsimilar-params {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `origin` | Array | Tracks to check. |
| `replace` | Array | Tracks to replace. |
| `count` | Number | Number of recommended tracks per original. Default is 1, maximum is 100. Output count may be lower due to lack of recommendations or filtering. |
| `isRemoveOriginArtists` | Boolean | If `true`, removes artists found in `origin` from recommendations. If `false`, keeps them. Default is `false`. |

### Return :id=replacewithsimilar-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=replacewithsimilar-examples {docsify-ignore}

1. Replace recently played tracks and playlist likes with close alternatives.

```js
let tracks = Source.getPlaylistTracks('', 'id');
Filter.replaceWithSimilar({
    origin: tracks,
    replace: [RecentTracks.get(2000), Source.getSavedTracks()],
    count: 3
});
```

2. Get recommendations based on liked tracks without original artists.

```js
let tracks = Source.getSavedTracks();
Filter.replaceWithSimilar({
  origin: tracks,
  replace: tracks,
  isRemoveOriginArtists: true,
})
```