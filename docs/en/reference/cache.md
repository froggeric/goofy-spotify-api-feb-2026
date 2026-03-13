# Cache {docsify-ignore}

Methods for managing Google Drive data.

By default, without specifying a file extension, `json` is implied. When explicitly specified, the text format `txt` is supported.

| Method | Result Type | Brief Description |
|--------|-------------|-------------------|
| [append](/reference/cache?id=append) | Number | Append data to an array from a file. |
| [compressArtists](/reference/cache?id=compressartists) | - | Remove insignificant data about artists. |
| [compressTracks](/reference/cache?id=compresstracks) | - | Remove insignificant data about tracks. |
| [copy](/reference/cache?id=copy) | String | Create a copy of the file in the source file's folder. |
| [read](/reference/cache?id=read) | Array/Object/String | Read data from a file. |
| [remove](/reference/cache?id=remove) | - | Move file to Google Drive trash. |
| [rename](/reference/cache?id=rename) | - | Rename a file. |
| [write](/reference/cache?id=write) | - | Write data to a file. |

## append

Append data to an array from a file. Creates the file if it doesn't exist.

### Arguments :id=append-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `filepath` | String | [Path to file](/best-practices?id=File-path). |
| `content` | Array | Data to add. |
| `place` | String | Join location: `begin` - beginning, `end` - end. Default is `end`. |
| `limit` | Number | Limit the number of array elements after appending new data. Default is 400 thousand elements. |

### Return :id=append-return {docsify-ignore}

`contentLength` (number) - number of elements after addition.

### Examples :id=append-examples {docsify-ignore}

1. Append playlist tracks to the beginning of a file. Limit the array to 5 thousand tracks after appending.

```js
let tracks = Source.getPlaylistTracks('playlist name', 'id');
Cache.append('filename.json', tracks, 'begin', 5000);
```

2. Append playlist tracks to the end of a file.

```js
let tracks = Source.getPlaylistTracks('playlist name', 'id');
Cache.append('filename.json', tracks);
```

## compressArtists

Remove insignificant data about artists. Use before saving to a file to reduce its size.

### Arguments :id=compressartists-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `artists` | Array | Artists from which insignificant data should be removed. |

### Return :id=compressartists-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=compressartists-examples {docsify-ignore}

1. Reduce file size with an array of artists.

```js
let filename = 'artists.json';
let artists = Cache.read(filename);
Cache.compressArtists(artists);
Cache.write(filename, artists);
```

## compressTracks

Remove insignificant data about tracks. Use before saving to a file to reduce its size. Includes a call to [compressArtists](/reference/cache?id=compressartists)

### Arguments :id=compresstracks-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `tracks` | Array | Tracks from which insignificant data should be removed. |

### Return :id=compresstracks-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=compresstracks-examples {docsify-ignore}

1. Reduce file size with an array of tracks.

```js
let filename = 'tracks.json';
let tracks = Cache.read(filename);
Cache.compressTracks(tracks);
Cache.write(filename, tracks);
```

## copy

Create a copy of a file in the source file's folder.

### Arguments :id=copy-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `filepath` | String | [Path to file](/best-practices?id=File-path). |

### Return :id=copy-return {docsify-ignore}

`filecopypath` (string) - path to the created copy.

### Examples :id=copy-examples {docsify-ignore}

1. Create a copy of a file and read its data.

```js
let filename = 'tracks.json';
let filecopyname = Cache.copy(filename);
let tracks = Cache.read(filecopyname);
```

## read

Read data from a file.

### Arguments :id=read-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `filepath` | String | [Path to file](/best-practices?id=File-path). |

### Return :id=read-return {docsify-ignore}

`content` (array/object/string) - data from the file.

If the file doesn't exist, the extension in the `filepath` string is checked. If absent or equal to _json_ - returns an empty array. In other cases - an empty string.

### Examples :id=read-examples {docsify-ignore}

1. Read data from a file and add to a playlist.

```js
let tracks = Cache.read('tracks.json');
Playlist.saveAsNew({
    name: 'Tracks from File',
    tracks: tracks,
});
```

## remove

Move a file to Google Drive trash. Data from trash is deleted after 30 days.

### Arguments :id=remove-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `filepath` | String | [Path to file](/best-practices?id=File-path). |

### Return :id=remove-return {docsify-ignore}

No return value.

### Examples :id=remove-examples {docsify-ignore}

1. Move a file to trash

```js
Cache.remove('filepath.json');
```

## rename

Rename a file.

!> Do not use the names `SpotifyRecentTracks`, `LastfmRecentTracks`, `BothRecentTracks`. They are needed in the [listening history](/details?id=Listening-history) accumulation mechanism.

### Arguments :id=rename-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `filepath` | String | [Path to file](/best-practices?id=File-path). |
| `newFilename` | String | New file name (not path) |

### Return :id=rename-return {docsify-ignore}

No return value.

### Examples :id=rename-examples {docsify-ignore}

1. Rename a file.

```js
Cache.rename('filename.json', 'newname.json');
```

## write

Write data to a file. Creates the file if it doesn't exist. Overwrites the file if it exists.

### Arguments :id=write-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `filepath` | String | [Path to file](/best-practices?id=File-path). |
| `content`  | Array | Data to write.                                 |

### Return :id=write-return {docsify-ignore}

No return value.

### Examples :id=write-examples {docsify-ignore}

1. Write favorite tracks to a file.

```js
let tracks = Source.getSavedTracks();
Cache.write('liked.json', tracks);
```
