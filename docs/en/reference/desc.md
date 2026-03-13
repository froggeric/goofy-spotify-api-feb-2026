# Helper Descriptions

## Identifier

The tables below show how to get an identifier from a link or URI.

### Playlist {docsify-ignore}

| id or playlistId | URI | Link |
|-|-|-|
| 5ErHcGR1VdYQmsrd6vVeSV | spotify:playlist:**5ErHcGR1VdYQmsrd6vVeSV** | [open.spotify.com/playlist/**5ErHcGR1VdYQmsrd6vVeSV**?si=123](open.spotify.com/playlist/5ErHcGR1VdYQmsrd6vVeSV) |
| 4vTwFTW4DytSY1N62itnwz | spotify:playlist:**4vTwFTW4DytSY1N62itnwz** | [open.spotify.com/playlist/**4vTwFTW4DytSY1N62itnwz**?si=123](open.spotify.com/playlist/4vTwFTW4DytSY1N62itnwz) |

### User {docsify-ignore}

For old accounts equals the login. For new accounts a sequence of letters and numbers.

| userId | URI | Link |
|-|-|-|
| glennpmcdonald | spotify:user:**glennpmcdonald** | [open.spotify.com/user/**glennpmcdonald**](open.spotify.com/user/glennpmcdonald) |
| ldxdnznzgvvftcpw09kwqm151 | spotify:user:**ldxdnznzgvvftcpw09kwqm151** | [open.spotify.com/user/**ldxdnznzgvvftcpw09kwqm151**](open.spotify.com/user/ldxdnznzgvvftcpw09kwqm151) |

## Object Parameter Descriptions

The table describes the main keys of Spotify objects in a free translation. The original can be read [here](https://developer.spotify.com/documentation/web-api/reference/tracks/get-audio-features/).

| Key | Range | Description |
|-|-|-|
| `popularity` | 0 - 100 |Popularity of a track, artist, or album. Those closer to 100 are more popular.</br> <ul><li>Track. Calculated based on the total number of plays and how recent they are. A track with many recent plays will be more popular than a track with many old plays. The value may lag by several days, meaning it doesn't update in real time.</li> <li>Artist and album. Calculated based on track popularity.</li></ul> 
| `duration_ms` | 0 - 0+ | Track duration in milliseconds ([calculator](https://www.google.ru/search?ie=UTF-8&q=%D0%BC%D0%B8%D0%BD%D1%83%D1%82%D1%8B%20%D0%B2%20%D0%BC%D0%B8%D0%BB%D0%BB%D0%B8%D1%81%D0%B5%D0%BA%D1%83%D0%BD%D0%B4%D1%8B%20%D0%BA%D0%B0%D0%BB%D1%8C%D0%BA%D1%83%D0%BB%D1%8F%D1%82%D0%BE%D1%80)). Useful for removing tracks with short duration by setting a minimum value. Or conversely with long duration.|
| `explicit` | boolean | Presence or absence of explicit lyrics. In the case of the [rangeTracks](/reference/filter?id=rangetracks) function, the value `false` will remove tracks with explicit lyrics. The value `true` or absence of this key will keep all tracks.
| `added_at` | string | Date the track was added to the playlist as a string. Example usage in the [loved and forgotten](/template?id=Loved-and-Forgotten) template.
| `genres` and `ban_genres` | array | Artist or album genres. Tests show that albums always have an empty list. In the case of the [rangeTracks](/reference/filter?id=rangetracks) function, only tracks that have at least one genre from the specified `genres` array and none from the `ban_genres` array will be selected.
| `release_date` | dates | Period in which the album of the considered track was released in date format ([format described here](/reference/filter?id=rangedateabs)). For example, between 2018 and 2020: `{ min: new Date('2018'), max: new Date('2020') }`

## Track Features {docsify-ignore}
| Key | Range | Description |
|-|-|-|
| `acousticness` | 0.0 - 1.0 | Confidence interval estimating whether the track is acoustic. A value of 1.0 indicates high confidence in this. ![Acousticness value distribution](../img/acousticness.png)
| `danceability` | 0.0 - 1.0 | Estimates how suitable a track is for dancing based on tempo, rhythm stability, beats, and overall pattern indicators. Tracks closer to 0.0 are less danceable and those closer to 1.0 are more danceable ![Danceability value distribution](../img/danceability.png)
| `energy` | 0.0 - 1.0 | Measure of intensity and activity of a track. Typically, energetic tracks feel fast, loud, and noisy. For example, death metal tracks. Calculation is based on dynamic range, loudness, timbre, onset rate, and overall entropy. Tracks closer to 0.0 are less energetic and those closer to 1.0 are more energetic ![Energy value distribution](../img/energy.png)
| `instrumentalness` | 0.0 - 1.0 | Estimate of vocal presence. For example, rap or spoken track clearly has vocals. The closer the value to 1.0, the more likely the track contains no vocals. A value above 0.5 is interpreted as an instrumental track, but probability is higher when approaching unity. ![Instrumentalness value distribution](../img/instrumentalness.png)
| `liveness` | 0.0 - 1.0 | Estimate of audience presence in the track recording or live track. Values above 0.8 reflect high probability of this. ![Liveness value distribution](../img/liveness.png)
| `loudness` | -60 to 0 | Overall loudness in decibels. The loudness value is averaged across the entire track. Useful when comparing relative loudness of tracks. Typically ranges from -60 to 0 dB. ![Loudness value distribution](../img/loudness.png)
| `speechiness` | 0.0 - 1.0 | Estimate of the amount of spoken words in a track. A value close to 1.0 characterizes the track as a talk show, podcast, or audiobook. Tracks with a value above 0.66 are probably made entirely of spoken words. From 0.33 to 0.66 may contain both speech and music. Below 0.33 for music and tracks without speech. ![Speechiness value distribution](../img/speechiness.png)
| `valence` | 0.0 - 1.0 | Measure of track positivity. A high value indicates a happier, more cheerful mood. A low value is characteristic of tracks with sad, depressive mood. ![Valence value distribution](../img/valence.png)
| `tempo` | 30 - 210 | Overall tempo of the track calculated in beats per minute (BPM). ![Tempo value distribution](../img/tempo.png)
| `key` | 0+ | Overall key of the track. Values are chosen based on [Pitch Class](https://en.wikipedia.org/wiki/Pitch_class). That is 0 = C, 1 = C♯/D♭, 2 = D and so on. If key is not set, value is -1.
| `mode` | 0 or 1 | Modality of the track. Major = 1, minor = 0.
| `time_signature` | 1+ | Overall estimate of the track's time signature - conventionally a notation for determining the number of beats in each measure.

## Genres for Recommendation Selection

This list is only needed for [getRecomTracks](/reference/source?id=getrecomtracks). In [rangeTracks](/reference/filter?id=rangetracks) you can use [this list](http://everynoise.com/everynoise1d.cgi?scope=all).

```
a: acoustic, afrobeat, alt-rock, alternative, ambient, anime, 
b: black-metal, bluegrass, blues, bossanova, brazil, breakbeat, british, 
c: cantopop, chicago-house, children, chill, classical, club, comedy, country, 
d: dance, dancehall, death-metal, deep-house, detroit-techno, disco, disney, drum-and-bass, dub, dubstep, 
e: edm, electro, electronic, emo, 
f: folk, forro, french, funk, 
g: garage, german, gospel, goth, grindcore, groove, grunge, guitar, 
h: happy, hard-rock, hardcore, hardstyle, heavy-metal, hip-hop, holidays, honky-tonk, house, 
i: idm, indian, indie, indie-pop, industrial, iranian, 
j: j-dance, j-idol, j-pop, j-rock, jazz, 
k: k-pop, kids, 
l: latin, latino, 
m: malay, mandopop, metal, metal-misc, metalcore, minimal-techno, movies, mpb, 
n: new-age, new-release, 
o: opera, 
p: pagode, party, philippines-opm, piano, pop, pop-film, post-dubstep, power-pop, progressive-house, psych-rock, punk, punk-rock, 
r: r-n-b, rainy-day, reggae, reggaeton, road-trip, rock, rock-n-roll, rockabilly, romance, 
s: sad, salsa, samba, sertanejo, show-tunes, singer-songwriter, ska, sleep, songwriter, soul, soundtracks,
spanish, study, summer, swedish, synth-pop, 
t: tango, techno, trance, trip-hop, turkish, 
w: work-out, world-music
```

## Playlist Categories

To get the list of available categories for a country, run the following code. Results in logs.
```js
let listCategory = Source.getListCategory({ limit: 50, country: 'US' });
console.log(listCategory.map(c => '\n' + c.name + '\n' + c.id).join('\n'));
```
