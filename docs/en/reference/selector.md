# Selector

Methods for selection and branching.

### isDayOfWeek

Returns a boolean value: `true` if today is the day of the week `strDay`, and `false` if not.

Arguments
- (string) `strDay` - day of the week.
- (string) `locale` - locale for the day of the week. Defaults to `en-US`, for which valid values are: `sunday`, `monday`, `tuesday`, `wednesday`, `thursday`, `friday`, `saturday`.

Usage example
```js
if (Selector.isDayOfWeek('friday')){
    // today is Friday
} else {
    // another day of the week
}
```

### isDayOfWeekRu

Returns a boolean value: `true` if today is the day of the week `strDay`, and `false` if not. The day of the week value is in Cyrillic.

Arguments
- (string) `strDay` - day of the week. Valid values: `понедельник`, `вторник`, `среда`, `четверг`, `пятница`, `суббота`, `воскресенье`.

Usage example
```js
if (Selector.isDayOfWeekRu('понедельник')){
    // today is Monday
} else if (Selector.isDayOfWeekRu('среда')) {
    // today is Wednesday
} else {
    // another day of the week
}
```

### isWeekend

Returns a boolean value: `true` if today is Saturday or Friday, and `false` if not.

No arguments.

Usage example
```js
if (Selector.isWeekend()){
    // today is a weekend
} else {
   // weekday
}
```

### keepAllExceptFirst / sliceAllExceptFirst

Modifies / returns an array consisting of all elements of array `array` except the first `skipCount` elements.

> Difference between `keep*` and `slice*` functions:
> 
> - `keep*` modifies the contents of the original array,
> - `slice*` returns a new array without modifying the original.

Arguments
- (array) `array` - array from which elements are taken.
- (number) `skipCount` - number of elements to skip.

Example 1 - Get all tracks except the first 10.
```js
let tracks = Source.getTracks(playlistArray);
tracks = Selector.sliceAllExceptFirst(tracks, 10);
```

### keepAllExceptLast / sliceAllExceptLast

Modifies / returns an array consisting of all elements of array `array` except the last `skipCount` elements.

Arguments
- (array) `array` - array from which elements are taken.
- (number) `skipCount` - number of elements to skip.

Example 1 - Get all tracks except the last 10.
```js
let tracks = Source.getTracks(playlistArray);
tracks = Selector.sliceAllExceptLast(tracks, 10);
```

### keepFirst / sliceFirst

Modifies / returns an array consisting of the first `count` elements of array `array`.

Arguments
- (array) `array` - array from which elements are taken.
- (number) `count` - number of elements.

Example 1 - Get the first 100 tracks.
```js
let tracks = Source.getTracks(playlistArray);
tracks = Selector.sliceFirst(tracks, 100);
```

### keepLast / sliceLast

Modifies / returns an array consisting of the last `count` elements of array `array`.

Arguments
- (array) `array` - array from which elements are taken.
- (number) `count` - number of elements.

Example 1 - Get the last 100 tracks.
```js
let tracks = Source.getTracks(playlistArray);
tracks = Selector.sliceLast(tracks, 100);
```

### keepNoLongerThan / sliceNoLongerThan

Modifies / returns an array of tracks with a total duration of no more than `minutes` minutes.

Arguments
- (array) `tracks` - source array of tracks.
- (number) `minutes` - number of minutes.

Example 1 - Get tracks with a total duration of no more than 60 minutes.
```js
let tracks = Source.getTracks(playlistArray);
tracks = Selector.sliceNoLongerThan(tracks, 60);
```

Example 2 - To calculate the duration of tracks from an array, use one of the following options
```js
let tracks = Source.getPlaylistTracks('', '37i9dQZF1DX5PcuIKocvtW');
let duration_ms = tracks.reduce((d, t) => d + t.duration_ms, 0); // milliseconds
let duration_s = tracks.reduce((d, t) => d + t.duration_ms, 0) / 1000; // seconds
let duration_min = tracks.reduce((d, t) => d + t.duration_ms, 0) / 1000 / 60; // minutes
let duration_h = tracks.reduce((d, t) => d + t.duration_ms, 0) / 1000 / 60 / 60; // hours
```

### keepRandom / sliceRandom

Modifies / returns an array consisting of randomly selected elements from the source array.

Arguments
- (array) `array` - array from which elements are taken.
- (number) `count` - number of randomly selected elements.

Example 1 - Get 20 random tracks.
```js
let tracks = Source.getTracks(playlistArray);
tracks = Selector.sliceRandom(tracks, 20);
```

### pickYear

Returns an array of tracks released in the specified year. If no such tracks exist, the nearest year is selected.

Arguments
- (array) `tracks` - tracks to select from.
- (string) `year` - release year.
- (number) `offset` - acceptable offset for the nearest year. Defaults to 5.

Example 1 - Select liked tracks released in 2020
```js
let tracks = Selector.pickYear(savedTracks, '2020');
```

### sliceCopy

Returns a new array that is a copy of the source array.

?> Use copying when you need to perform different actions on the same source in one script. This will speed up execution and avoid sending the same requests twice.

Arguments
- (array) `array` - source array to copy.

Example 1 - Create a copy of an array.
```js
let tracks = Source.getTracks(playlistArray);
let tracksCopy = Selector.sliceCopy(tracks);
```