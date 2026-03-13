# Order

Methods for sorting.

| Method | Result Type | Brief Description |
|--------|-------------|-------------------|
| [reverse](/reference/order?id=reverse) | - | Sort in reverse order. |
| [separateArtists](/reference/order?id=separateartists) | - | Separate tracks of the same artist from each other. |
| [separateYears](/reference/order?id=separateyears) | Object | Distribute tracks by release year. |
| [shuffle](/reference/order?id=shuffle) | - | Shuffle array elements randomly. |
| [sort](/reference/order?id=sort) | - | Sort array by metadata. |

## reverse

Sort in reverse order.

### Arguments :id=reverse-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `array` | Array | Array to sort. |

### Return :id=reverse-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=reverse-examples {docsify-ignore}

1. Reverse sorting for an array of numbers.

```js
let array = [1, 2, 3, 4, 5, 6];

Order.reverse(array);
// result 6, 5, 4, 3, 2, 1

Order.reverse(array);
// result 1, 2, 3, 4, 5, 6
```

2. Reverse sorting for an array of tracks.

```js
let tracks = Source.getTracks(playlistArray);
Order.reverse(tracks);
```

## separateArtists

Separate tracks of the same artist from each other. Tracks that couldn't be placed will be excluded.

### Arguments :id=separateartists-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `tracks` | Array | Tracks to sort. |
| `space` | Number | Minimum spacing. |
| `isRandom` | Boolean | If `true` performs random sorting of the original array, which affects the order when separating artists. If `false` without random sorting. Then the result will be the same with the same input tracks. Default is `false`. |

### Return :id=separateartists-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=separateartists-examples {docsify-ignore}

1. Separation example

```js
let array = ['cat', 'cat', 'dog', 'lion']
Order.separateArtists(array, 1, false);
// result cat, dog, cat, lion

array = ['cat', 'cat', 'dog', 'lion']
Order.separateArtists(array, 1, false);
// repeated call, same result: cat, dog, cat, lion

array = ['cat', 'cat', 'dog', 'lion']
Order.separateArtists(array, 1, true);
// repeated call with random sorting: cat, lion, dog, cat
```

2. Separate the same artist by at least two others.

```js
let tracks = Source.getTracks(playlistArray);
Order.separateArtists(tracks, 2);
```

## separateYears

Distribute tracks by release year. Tracks are not sorted by date.

### Arguments :id=separateyears-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `tracks` | Array | Tracks to sort. |

### Return :id=separateyears-return {docsify-ignore}

`tracksByYear` (object) - track arrays distributed by years.

### Examples :id=separateyears-examples {docsify-ignore}

1. Get tracks released only in 2020

```js
let tracks2020 = Order.separateYears(tracks)['2020'];
```

2. An error may occur when there are no tracks of the specified year among the tracks. Choose one of: use [pickYear](/reference/selector?id=pickyear), substitute with an empty array, check with a condition.

```js
// Substitute with empty array if no tracks of specified year
let tracks2020 = Order.separateYears(tracks)['2020'] || [];

// Check through condition
let tracksByYear = Order.separateYears(tracks);
if (typeof tracksByYear['2020'] != 'undefined'){
    // tracks exist
} else {
    // no tracks
}
```

## shuffle

Shuffle array elements randomly.

### Arguments :id=shuffle-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `array` | Array | Array to sort. |
| `factor` | Number | Randomness factor. Default is 1. The closer to 0, the fewer swaps and closer to original position. |

### Return :id=shuffle-return {docsify-ignore}

Modifies the input array. Returns the same array.

### Examples :id=shuffle-examples {docsify-ignore}

1. Random sorting on an array of numbers with different factors. Run the `testShuffleWithFactor` function several times with different `factor` values for clarity.

```js
function testShuffleWithFactor() {
    let array = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
    console.log('0.0:', Order.shuffle(array, 0.0)); // no changes: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9
    console.log('0.3:', Order.shuffle(array, 0.3)); // close swaps: 0, 1, 3, 2, 6, 5, 4, 7, 8, 9
    console.log('0.6:', Order.shuffle(array, 0.6)); // result: 3, 0, 8, 1, 2, 5, 4, 7, 6, 9
    console.log('1.0:', Order.shuffle(array)) ; // default 1.0, result: 6, 0, 9, 5, 7, 4, 8, 1, 2, 3
}
```

1. Random sorting on an array of tracks.

```js
let tracks = Source.getTracks(playlistArray);
Order.shuffle(tracks);
```

## sort

Sort array by metadata.

!> The function makes additional requests. To reduce the number of requests, use it after maximum reduction of the track array by other methods. More details in [rangeTracks](/reference/filter?id=rangetracks).

?> If multiple track sources are mixed, not all have the specified key. For example, `played_at` only exists in listening history.

### Arguments :id=sort-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `source` | Array | Array to sort. |
| `pathKey` | String | Sorting key. |
| `direction` | String | Sorting direction: _asc_ ascending, _desc_ descending. Default is _asc_. |

#### Sorting Key

The key has format `category.data`. [Metadata description](/reference/desc?id=Object-parameter-descriptions).

| Category | Data |
|-|-|
| meta | name, popularity, duration_ms, explicit, added_at, played_at |
| features | acousticness, danceability, energy, instrumentalness, liveness, loudness, speechiness, valence, tempo, key, mode, time_signature, duration_ms |
| artist | popularity, followers, name |
| album | popularity, name, release_date |

?> When using the `features` category, the overall flow of any tracks becomes more pleasant.

### Return :id=sort-return {docsify-ignore}

No return value. Modifies the input array.

### Examples :id=sort-examples {docsify-ignore}

1. Sort by descending artist popularity.

```js
Order.sort(tracks, 'artist.popularity', 'desc');
```

2. Sort by ascending energy.

```js
Order.sort(tracks, 'features.energy', 'asc');
```
