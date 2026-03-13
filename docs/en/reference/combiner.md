# Combiner {docsify-ignore}

Methods for combining elements.

| Method | Result Type | Brief Description |
|--------|-------------|-------------------|
| [alternate](/reference/combiner?id=alternate) | Array | Alternate array elements. One step - one array element. |
| [mixin](/reference/combiner?id=mixin) | Array | Alternate elements of two arrays. One step - one or more elements from one array. |
| [mixinMulti](/reference/combiner?id=mixinmulti) | Array | Alternate elements of unlimited number of arrays. One step - one or more elements from one array. |
| [push](/reference/combiner?id=push) | Array | Add to the end of the first array elements of the second array and so on. |

## alternate

Alternate array elements. One step - one array element.

### Arguments :id=alternate-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `bound` | String | Alternation boundary. With `min` alternation ends when one of the sources is exhausted. With `max` alternation continues as long as there are untouched elements. |
| ``...arrays`` | Arrays | Sources of elements for alternation. |

### Return :id=alternate-return {docsify-ignore}

`resultArray` (array) - new array in which source elements alternate.

### Examples :id=alternate-examples {docsify-ignore}

1. Alternate elements of three arrays.

```js
let firstArray = [1, 3, 5];
let secondeArray = [2, 4, 6, 8, 10];
let thirdArray = [100, 200, 300];
let resultArray = Combiner.alternate('max', firstArray, secondeArray, thirdArray);
// result 1, 2, 100, 3, 4, 200, 5, 6, 300, 8, 10
```

2. Alternate top plays for the month and favorite tracks.

```js
let topTracks = Source.getTopTracks('short'); // let's say, 50 tracks
let savedTracks = Source.getSavedTracks(20); // let's say, 20 tracks
let resultArray = Combiner.alternate('min', topTracks, savedTracks);
// result contains 40 tracks
```

## mixin

Alternate elements of two arrays. One step - one or more elements from one array. Includes a call to [mixinMulti](/reference/combiner?id=mixinmulti).

### Arguments :id=mixin-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `xArray` | Array | First source. |
| `yArray` | Array | Second source. |
| `xRow` | Number | Number of consecutive elements from first source. |
| `yRow` | Number | Number of consecutive elements from second source. |
| `toLimitOn` | Boolean | Elements alternate as long as the proportion can be maintained. If `true` extra elements are not included in result. If `false` they are added at the end of result. Default is `false`. |

### Return :id=mixin-return {docsify-ignore}

`resultArray` (array) - new array in which elements of two sources alternate in specified proportion.

### Examples :id=mixin-examples {docsify-ignore}

1. Alternate playlist tracks and favorite tracks in 5 to 1 ratio. Discard extras.

```js
let tracks = Source.getTracks(playlistArray);
let savedTracks = Source.getSavedTracks();
let resultArray = Combiner.mixin(tracks, savedTracks, 5, 1, true);
```

## mixinMulti

Alternate elements of unlimited number of arrays. One step - one or more elements from one array.

### Arguments :id=mixinmulti-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `params` | Object | Alternation parameters. |

#### Alternation Parameters :id=mixinmulti-params {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `source` | Array | Array of source arrays. |
| `inRow` | Array | Elements that set the number of consecutive elements for each source. |
| `toLimitOn` | Boolean | Elements alternate as long as the proportion can be maintained. If `true` extra elements are not included in result. If `false` they are added at the end of result. Default is `false`. With `toLimitOn = true`, the first iteration checks the number of elements. If there are fewer elements than specified by the ratio, an empty array is returned. |

### Return :id=mixinmulti-return {docsify-ignore}

`resultArray` (array) - new array in which source elements alternate in specified proportion.

### Examples :id=mixinmulti-examples {docsify-ignore}

1. Alternate elements in 1:1:1 ratio. Keep all elements.

```js
let x = [1, 2, 3, 4, 5];
let y = [10, 20, 30, 40];
let z = [100, 200, 300];
let result = Combiner.mixinMulti({
    source: [x, y, z],
    inRow: [1, 1, 1],
});
// 1, 10, 100, 2, 20, 200, 3, 30, 300, 4, 40, 5
```

2. Alternate elements in 2:4:2 ratio as long as the sequence can be maintained.

```js
let x = [1, 2, 3, 4, 5];
let y = [10, 20, 30, 40];
let z = [100, 200, 300];
let result = Combiner.mixinMulti({
    toLimitOn: true,
    source: [x, y, z],
    inRow: [2, 4, 2],
});
// 1, 2, 10, 20, 30, 40, 100, 200
```

3. Alternate recommendations, favorite tracks, and listening history in 4:1:1 ratio as long as the sequence can be maintained.

```js
let recom = Source.getRecomTracks();
let saved = Source.getSavedTracks();
let recent = RecentTracks.get();
let tracks = Combiner.mixinMulti({
    toLimitOn: true,
    source: [recom, saved, recent],
    inRow: [4, 1, 1],
});
```

## push

Add to the end of the first array elements of the second array and so on.

### Arguments :id=push-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `sourceArray` | Array | First array. Elements from subsequent arrays are added to it. |
| `...additionalArray` | Arrays | Subsequent arrays to add to the previous one. |

### Return :id=push-return {docsify-ignore}

`sourceArray` (array) - original first array after adding elements from other arrays.

### Examples :id=push-examples {docsify-ignore}

1. Add elements of second array to the end of first array.

```js
let firstArray = Source.getTracks(playlistArray); // let's say, 20 tracks
let secondeArray = Source.getSavedTracks(); // let's say, 40 tracks
Combiner.push(firstArray, secondeArray);
// now firstArray has 60 tracks
```

2. Add elements of two other arrays to the first array.

```js
let firstArray = Source.getTracks(playlistArray); // let's say, 25 tracks
let secondeArray = Source.getSavedTracks(); // let's say, 100 tracks
let thirdArray = Source.getPlaylistTracks(); // let's say, 20 tracks
Combiner.push(firstArray, secondeArray, thirdArray);
// now firstArray has 145 tracks
```
