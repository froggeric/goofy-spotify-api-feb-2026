# Additional Information

# Parameters

Description of parameters from the `config` file

## API
- `CLIENT_ID` and `CLIENT_SECRET` (string) - keys for accessing Spotify Web API. Created during [initial installation](/install).

- `LASTFM_API_KEY` (string) - key for working with Last.fm API. Created [additionally](/tuning?id=Setting-up-lastfm).

- `MUSIXMATCH_API_KEY` (string) - key from musixmatch service for the [detectLanguage](/reference/filter?id=detectlanguage) function. Created [additionally](/tuning?id=Setting-up-musixmatch).

## Listening History
- `ON_SPOTIFY_RECENT_TRACKS` (boolean) - when `true`, Spotify listening history tracking is enabled. When `false`, it is disabled.

- `ON_LASTFM_RECENT_TRACKS` (boolean) - when `true`, Last.fm listening history tracking is enabled. When `false`, it is disabled.

- `LASTFM_LOGIN` (string) - Last.fm username whose history is being collected. Used by default in other module functions as well.

- `LASTFM_RANGE_RECENT_TRACKS` (number) - the number of recent tracks reviewed in Last.fm history over the past 15 minutes.

- `COUNT_RECENT_TRACKS` (number) - the number of history tracks to save. Default is 60 thousand. In practice, it works with larger values as well. The limit is a file size of 50 MB.

## General
- `LOG_LEVEL` (string) - when `info`, messages with information and errors from library functions are displayed. When `error`, only error messages. When empty string, messages are disabled. In `config` parameters, the default value is set, which applies to each run. In your code, you can change the log level for the current execution using `Admin.setLogLevelOnce('value')`.

- `LOCALE` (string) - locale when requesting playlists. Affects how track titles are presented. [There are known cases](https://github.com/Chimildic/goofy/discussions/79#discussioncomment-814744) where an artist in Cyrillic was returned with a Latin equivalent. Default value is `RU`.

- `REQUESTS_IN_ROW` (number) - the number of requests sent in parallel when possible. Default is 20. Affects data retrieval speed. For example, playlist track requests. Increasing is not recommended. Spotify has started giving shadow bans for a day or more (i.e., stops responding to requests). If you continue to receive bans, reduce the value to 10 or optimize the code to reduce the number of requests from the source.

- `MIN_DICE_RATING` (number) - minimum coefficient value from 0.0 to 1.0, at which an element is considered the best match when importing, for example, tracks to Spotify. Default is _0.6005_.
If the found element has a lower value, it is discarded. When multiple elements satisfy the minimum value, the element with the highest value is selected.

# Identifier

The tables below show how to get an identifier from a link or URI.

## Playlist {docsify-ignore}

| id or playlistId | URI | Link |
|-|-|-|
| 5ErHcGR1VdYQmsrd6vVeSV | spotify:playlist:**5ErHcGR1VdYQmsrd6vVeSV** | [open.spotify.com/playlist/**5ErHcGR1VdYQmsrd6vVeSV**?si=123](open.spotify.com/playlist/5ErHcGR1VdYQmsrd6vVeSV) |
| 4vTwFTW4DytSY1N62itnwz | spotify:playlist:**4vTwFTW4DytSY1N62itnwz** | [open.spotify.com/playlist/**4vTwFTW4DytSY1N62itnwz**?si=123](open.spotify.com/playlist/4vTwFTW4DytSY1N62itnwz) |

## User {docsify-ignore}

For old accounts, it equals the username. For new accounts, it's a sequence of letters and numbers.

| userId | URI | Link |
|-|-|-|
| glennpmcdonald | spotify:user:**glennpmcdonald** | [open.spotify.com/user/**glennpmcdonald**](open.spotify.com/user/glennpmcdonald) |
| ldxdnznzgvvftcpw09kwqm151 | spotify:user:**ldxdnznzgvvftcpw09kwqm151** | [open.spotify.com/user/**ldxdnznzgvvftcpw09kwqm151**](open.spotify.com/user/ldxdnznzgvvftcpw09kwqm151) |

# Object Parameter Descriptions

The table describes the main keys of Spotify objects in a free translation. The original can be read [here](https://developer.spotify.com/documentation/web-api/reference/tracks/get-audio-features/).

| Key | Range | Description |
|-|-|-|
| `popularity` | 0 - 100 |Popularity of a track, artist, or album. Those closer to 100 are more popular.</br> <ul><li>Track. Calculated based on the total number of plays and how recent they are. A track with more recent plays will be more popular than a track with more older plays. The value may lag by several days, meaning it is not updated in real time.</li> <li>Artist and album. Calculated based on the popularity of tracks.</li></ul>
| `duration_ms` | 0 - 0+ | Track duration in milliseconds ([calculator](https://www.google.com/search?q=minutes+to+milliseconds+calculator)). Useful for removing tracks with short duration by setting a minimum value. Or conversely, tracks with long duration.|
| `explicit` | boolean | Presence or absence of explicit lyrics. In the case of the [rangeTracks](/reference/filter?id=rangetracks) function, a value of `false` will remove tracks with explicit lyrics. A value of `true` or absence of this key will keep all tracks.
| `added_at` | string | Date when a track was added to a playlist in string format. Example usage in the [favorites and forgotten](/template?id=Favorites-and-forgotten) template.
| `genres` and `ban_genres` | array | Genres of an artist or album. Tests show that albums always have an empty list. In the case of the [rangeTracks](/reference/filter?id=rangetracks) function, only those tracks that have at least one genre from the specified `genres` array and none from the `ban_genres` array will be selected.
| `isRemoveUnknownGenre` | boolean | When `true`, removes artists with an empty genre list (common for lesser-known ones). When `false`, keeps them. Default is `true`. |
| `release_date` | dates | Period in which the album of the considered track was released in date format ([format described here](/reference/filter?id=rangedateabs)). For example, between 2018 and 2020: `{ min: new Date('2018'), max: new Date('2020') }`

## Track Features {docsify-ignore}
| Key | Range | Description |
|-|-|-|
| `acousticness` | 0.0 - 1.0 | Confidence interval estimating whether a track is acoustic. A value of 1.0 indicates high confidence in this. ![Acousticness value distribution](/img/acousticness.png)
| `danceability` | 0.0 - 1.0 | Evaluates how suitable a track is for dancing based on its tempo, rhythm stability, beats, and overall pattern of indicators. Tracks closer to 0.0 are less danceable, and those closer to 1.0 are more danceable. ![Danceability value distribution](/img/danceability.png)
| `energy` | 0.0 - 1.0 | Assessment of the intensity and activity of a track. Typically, energetic tracks feel fast, loud, and noisy. For example, death metal genre tracks. The calculation is based on dynamic range, loudness, timbre, onset rate, and overall entropy. Tracks closer to 0.0 are less energetic, and those closer to 1.0 are more energetic. ![Energy value distribution](/img/energy.png)
| `instrumentalness` | 0.0 - 1.0 | Assessment of vocal presence. For example, a rap or spoken track clearly has vocals. The closer the value is to 1.0, the more likely the track contains no vocals. A value above 0.5 is interpreted as an instrumental track, but the probability is higher when approaching one. ![Instrumentalness value distribution](/img/instrumentalness.png)
| `liveness` | 0.0 - 1.0 | Assessment of audience presence in the track recording or live track. Values above 0.8 reflect a high probability of this. ![Liveness value distribution](/img/liveness.png)
| `loudness` | -60 to 0 | Overall loudness in decibels. The loudness value is averaged across the entire track. Useful for comparing the relative loudness of tracks. Typically, the range is from -60 to 0 dB. ![Loudness value distribution](/img/loudness.png)
| `speechiness` | 0.0 - 1.0 | Assessment of the number of spoken words in a track. A value close to 1.0 characterizes the track as a talk show, podcast, or audiobook. Tracks with a value above 0.66 are probably entirely composed of speech. From 0.33 to 0.66 may contain both speech and music. Below 0.33 for music and tracks without speech. ![Speechiness value distribution](/img/speechiness.png)
| `valence` | 0.0 - 1.0 | Assessment of track positivity. A high value indicates a happier, more cheerful mood. A low value is characteristic of tracks with a sad, depressive mood. ![Valence value distribution](/img/valence.png)
| `tempo` | 30 - 210 | Overall tempo of the track calculated in beats per minute (BPM). ![Tempo value distribution](/img/tempo.png)
| `key` | 0+ | Overall key of the track. Values are selected based on [Pitch Class](https://en.wikipedia.org/wiki/Pitch_class). That is, 0 = C, 1 = C♯/D♭, 2 = D, and so on. If the key is not set, the value is -1.
| `mode` | 0 or 1 | Modality of the track. Major = 1, minor = 0.
| `time_signature` | 1+ | Overall estimate of the track's time signature - conventionally, a notation for determining the number of beats in each measure.

# Genres for Recommendation Selection

This list is only needed for [getRecomTracks](/reference/source?id=getrecomtracks). In [rangeTracks](/reference/filter?id=rangetracks), you can use [this list](http://everynoise.com/everynoise1d.cgi?scope=all).

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

# Playlist Categories

To get a list of available categories for a country, run the following code. Results appear in logs.
```js
let listCategory = Source.getListCategory({ limit: 50, country: 'US' });
console.log(listCategory.map(c => '\n' + c.name + '\n' + c.id).join('\n'));
```