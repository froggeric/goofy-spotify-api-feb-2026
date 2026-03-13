# Parameters

Description of parameters from the `config` file

## API
- `CLIENT_ID` and `CLIENT_SECRET` (string) - keys for accessing Spotify Web API. Created during [first installation](/install).

- `LASTFM_API_KEY` (string) - key for working with Last.fm API. Created [additionally](/tuning?id=Lastfm-setup).

- `MUSIXMATCH_API_KEY` (string) - key from the musixmatch service for the [detectLanguage](/reference/filter?id=detectlanguage) function. Created [additionally](/tuning?id=Musicmatch-setup).

## Listening History
- `ON_SPOTIFY_RECENT_TRACKS` (boolean) - when `true` Spotify listening history tracking. When `false` disabled.

- `ON_LASTFM_RECENT_TRACKS` (boolean) - when `true` Last.fm listening history tracking. When `false` disabled.

- `LASTFM_LOGIN` (string) - Last.fm user login whose history is collected. Used by default and in other module functions.

- `LASTFM_RANGE_RECENT_TRACKS` (number) - number of recent tracks viewed in Last.fm history over the past 15 minutes.

- `COUNT_RECENT_TRACKS` (number) - number of history tracks to save. Default is 20 thousand. In practice it works fine with 40 thousand. The limit is a file size of 50 MB.

## General
- `LOG_LEVEL` (string) - when `info` displays messages with information and errors from library functions. When `error` only error messages. When empty string disables messages. In the `config` parameters, the default value is set, acting on each run. In your code, you can change the log level for the current execution `Admin.setLogLevelOnce('value')`.

- `LOCALE` (string) - locale when requesting playlists. Affects how track titles are presented. [Known cases](https://github.com/Chimildic/goofy/discussions/79#discussioncomment-814744) where a Cyrillic artist was returned with a Latin equivalent. Default value is `RU`.

- `REQUESTS_IN_ROW` (number) - number of requests sent in parallel when possible. Default is 40. Affects data retrieval speed. For example, playlist track request. When getting a large number of error number `503` or having algorithms with a very large number of requests, it's recommended to lower this parameter value. Increasing is not recommended.

- `MIN_DICE_RATING` (number) - minimum coefficient value from 0.0 to 1.0 at which an element is considered the best match when importing, for example, tracks to Spotify. Default is _0.6005_.
If a found element has a lower value, it's discarded. When multiple elements satisfy the minimum value, the element with the highest of them is chosen.
