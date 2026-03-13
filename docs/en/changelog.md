# Changelog

The current library version is reflected in the `VERSION` constant in the `library` file.

To add your own functions or override existing ones, use the [instructions](https://github.com/Chimildic/goofy/discussions/18).

[Copy the updated code](https://script.google.com/d/1DnC4H7yjqPV2unMZ_nmB-1bDSJT9wQUJ7Wq-ijF4Nc7Fl3qnbT0FkPSr/edit?usp=sharing).

### Version 2.0.4
- Perform the [migration](/migrate2.md)
- Added function [getRecomArtists](/reference/source?id=getrecomartists) as a replacement for errors from [getRelatedArtists](/reference/source?id=getrelatedartists) after Spotify's updated policy

## Version 1.8.5
- Removed `EveryNoise` module. The site stopped providing releases directly in the HTML page. Now the page is populated dynamically, which is impossible to parse within Apps Script.
- Removed function `Source.getReleasesByArtists`.
- Added function [Source.getRecentReleasesByArtists](/reference/source?id=getrecentreleasesbyartists). The algorithm for obtaining recent releases has been completely redesigned.
- Added a [template for obtaining recent releases](/template?id=Новые-релизы-по-частям) when there is a very large number of artists.

## Version 1.8.4
- Added `factor` argument to [Order.shuffle](/reference/order?id=shuffle) function.
- Added interruption when saving tracks to a non-existent playlist (when id is strictly hardcoded in the arguments of `Playlist.save*` function).
- Added description of actions for `Access not granted or expired` error.
- The value of `REQUESTS_IN_ROW` parameter is forcibly reduced to 20 if the current value is 40. Spotify started frequently giving shadow bans for a day or more when this value is large.
- Functions `SpotifyRequest.get*` filter out null elements to avoid script interruption due to unexpected Spotify responses.
- Fix for Library module functions. Spotify changed the request format.
- Fix for playlist track backup in case of unexpected error. The file name will not contain dates. Google Drive stores file versions.

## Version 1.8.3
- [12.09.22] Fixes for rarely encountered bugs

## Version 1.8.2

- [#175](https://github.com/Chimildic/goofy/discussions/175) Added error handling when sending tracks to a playlist. Changed the key indicating the place of adding tracks: instead of boolean `toEnd` now string `position`. By default tracks are added to the end of the playlist.
- [#187](https://github.com/Chimildic/goofy/discussions/187) Re-read file on unknown Google Drive error
- When searching for the best match, transliteration from Cyrillic to Latin is applied.
- Fix for error where tracks from `getSavedAlbumTracks` function were not written to cache.
- Fix for error where import was interrupted due to null search result from Spotify.
- Fix: first run of Clerk task occurred at any time, ignoring the target value.

## Version 1.8.0

- Changed algorithm for finding the best match when importing to Spotify. This will reduce the number of "wrong" tracks when the searched item is not in the Spotify database. You can influence the comparison accuracy by changing the `MIN_DICE_RATING` parameter in the _config_ file.
- Increased limits in various places.
- Country for `removeUnavailable` is taken from account data by default.
- Removed function `sendMusicRequest`.

## Version 1.7.0

- Add the [Cheerio](https://github.com/Chimildic/goofy/discussions/91#discussioncomment-1931923) library.
- [#153](https://github.com/Chimildic/goofy/issues/153) Added likes caching. Configure the [timezone](/tuning?id=Часовой-пояс).
- [#159](https://github.com/Chimildic/goofy/issues/159) Added functions for parsing elements from last.fm tag pages: [getTracksByTag](/reference?id=gettracksbytag), [getArtistsByTag](/reference?id=getartistsbytag), [getAlbumsByTag](/reference?id=getalbumsbytag).
- (10.01.22) [#155](https://github.com/Chimildic/goofy/issues/155#issuecomment-1008871981) Changed the algorithm for selecting releases due to a shortage of artist names consisting of commonly used words. Cheerio library is required.

## Version 1.6.3
- Yandex module completely removed. Yandex stopped responding to requests. Many functions can be replaced by the [YaMuTools](https://github.com/Chimildic/YaMuTools) extension. But it doesn't have scheduled tasks.
- [#155](https://github.com/Chimildic/goofy/issues/155) New function [Source.getReleasesByArtists](/reference/source?id=getreleasesbyartists). Collects artist releases significantly faster than the direct iteration previously used when combining different functions.
- [#162](https://github.com/Chimildic/goofy/issues/162) Addon for detecting and filtering the main track language moved to the main code [Filter.detectLanguage](/reference/filter?id=detectlanguage). On first run from the editor, it will require granting access to Google Sheets.
- Most of the function documentation has changed writing style, and modules are divided into separate files (old links to `/func` stop working).
- Added a link to reviews in the site navigation.

## Version 1.6.2
- Reduced the number of Google Drive requests. Read/write operations of the same file within a single script execution will take less time.
- Removed function `Cache.clear`. Use `Cache.write` to overwrite a file.

## Version 1.6.1
- Changed data structure on Drive to support multiple Spotify accounts on one Google account.
  On first run, all files from the `Goofy Data` folder will be moved to a new folder with the identifier of the user for whom the script is executed. Subsequent runs will modify files from the folder corresponding to the user.

  If you have one Spotify account, you don't need to do anything.

  If you previously configured multiple Spotify accounts on one Google account, perform the following actions:
  - run any script from the first account
  - return standard names to this account's files on Drive
  - move other accounts' files to the `Goofy Data` folder
  - repeat for each account
- Added ability to [create folders](/best-practices?id=Путь-до-файла)

## Version 1.6.0
- Fixed errors in sending requests. The number of pauses during requests will increase, pause time will decrease (on response status 429).
- Adding tracks [RecentTracks.appendTracks](/reference/recenttracks?id=appendtracks) no longer breaks the algorithm for searching new listens in the history update trigger.
- [Cache.append](/reference/cache?id=append) no longer adds data to the original array (use [push](/reference/push?id=push)). Adds only to the cache file. Returns the count of all elements after addition.
- Added ability to disable console messages from library functions. Global setting is defined in the [config file](/config). Temporary change via function `Admin.setLogLevelOnce`. Removed `isLogging` arguments from functions.
- Without specifying a username, all functions from the `Lastfm` module substitute the value from the `config file`.

## Version 1.5.4
- Added retry attempt for read/write on unknown Google Drive service error.
- Added functions: [Lastfm.rangeTags](/reference/lastfm?id=rangetags) and a number of Lastfm.getTop*ByTag.
- [addToQueue](/reference/player?id=addtoqueue) can add an array of tracks to the queue.
- Added parameter to [mineTracks](/reference/source?id=minetracks) for skipping elements.
- Added argument to [removeUnavailable](/reference/filter?id=removeunavailable) to disable log messages.

## Version 1.5.3
- New function [checkFavoriteTracks](/reference/library?id=checkfavoritetracks).
- Added parameter to [replaceWithSimilar](/reference/filter?id=replacewithsimilar) that allows removing source artists from recommendations.

## Version 1.5.2
- New functions [getSavedAlbums](/reference/source?id=getsavedalbums), [followPlaylists](/reference/library?id=followplaylists), [unfollowPlaylists](/reference/library?id=unfollowplaylists).
- Function [getFollowedTracks](/reference/source?id=getfollowedtracks) received parameter `isFlat`.
- Functions `getTracks` and `getAlbums` removed from `Yandex` module. Since Yandex stopped responding to such requests from Apps Script.

## Version 1.5.1
- New module [Player](/reference/player). Need to [update access permissions](/tuning?id=Обновить-права-доступа).
- `getPlayback` moved to `Player`.
- Added function [Player.transferPlayback](/reference/player?id=transferplayback)
- `removeTracks` and `removeArtists` received mode to check only the main track artist.
- Changed input parameter format for [replaceWithSimilar](/reference/filter?id=replacewithsimilar).
- Fixed error with `getTop*`. Spotify does not allow `locale` parameter in it. Write to the forum if you encounter `invalid request` error.

## Version 1.5.0
- Consider all track artists in filter functions `remove*`, `match*`, `dedup*`.
- [getArtistsTracks](/reference/source?id=getartiststracks) and [getArtistsAlbums](/reference/source?id=getartistsalbums) received parameter `isFlat`, allowing to group the result by artist. Similar to [getArtistsTopTracks](/reference/source?id=getartiststoptracks). Default behavior unchanged, no code changes needed.
- `rangeTracks` can filter by album type.
- `dedupTracks` received new argument to control duration deviation for tracks with identical titles. Details [here](https://github.com/Chimildic/goofy/discussions/116).
- All get-requests from `SpotifyRequest` contain locale parameter ([details](https://github.com/Chimildic/goofy/discussions/79#discussioncomment-1019029)).

## Version 1.4.9
- Experiment with function [sendMusicRequest](/reference/search?id=sendmusicrequest)
- New parameter in [Lastfm.getCustomTop](/reference/lastfm?id=getcustomtop), new function [Lastfm.convertToSpotify](/reference/lastfm?id=converttospotify) and template with their usage - [one-hit artists](/template?id=Исполнители-одного-хита).
- Removed function `getPlayingTrack`. Added [getPlayback](/reference/player?id=getplayback) instead. To use, you need to [update access permissions](/tuning?id=Обновить-права-доступа).
- `doGet` adapted for opening `launch.html`, no need to replace the function for [phone control](https://github.com/Chimildic/goofy/discussions/9)
- Modified function `replaceWithSimilar`. Should not create duplicates, requests recommendations with more data from the original track.
- Bugfix [Alisa #99](https://github.com/Chimildic/goofy/discussions/99#discussioncomment-973227)

## Version 1.4.8
- [rangeTracks](/reference/filter?id=rangetracks) now has [calculated values](https://github.com/Chimildic/goofy/discussions/87): `anger`, `happiness` and `sadness`. These are not provided by Spotify, so they cannot be used when requesting recommendations.
- All match functions now check the main track artist, in addition to album and title.
- [getCustomTop](/reference/lastfm?id=getcustomtop) has parameters for minimum and maximum play count.
- Removal of album duplicates when requesting albums from artists. Spotify bug, sometimes returns identical albums (for different countries).
- Added documentation for Search module
- Need to [update config file](https://github.com/Chimildic/goofy/blob/main/config.js):
  - Added locale parameter `RU` when requesting playlists. Due to [this post](https://github.com/Chimildic/goofy/discussions/79#discussioncomment-814744): Cyrillic name was returned in Latin script.
  - The 20 thousand track history limit is moved to config, so you can change it and not lose it on update. There's an example of stable operation with 40 thousand tracks.

## Version 1.4.7
- Experiment. On unknown error from Google during write via `Cache.write`, a retry attempt occurs after a pause.
- [06.06.21]
  - [replaceWithSimilar](/reference/filter?id=replacewithsimilar) can accept multiple arrays for replacement at once. If no replacement is found, the track is removed.
- [20.05.21]
  - Experiment. Additional check when searching for the best match (string matching), [details](https://github.com/Chimildic/goofy/discussions/64).
  - Order.sort can sort arrays of artists and albums.
  - New function [Playlist.removeTracks](/reference/playlist?id=removetracks).
  - [getTracks](/reference/source?id=gettracks) can select a limited number of tracks from playlists.

## Version 1.4.6
- New function _getPlayingTrack_. Need to [update access permissions](/tuning?id=Обновить-права-доступа).
- When creating a playlist, you can specify a static cover via a direct link to it.

## Version 1.4.5
- Now [mineTracks](/reference/source?id=minetracks) can search for keywords in album titles and tracks themselves.
- In `mineTracks` argument `playlistCount` **renamed** to `itemCount`.
- New function in Filter: [replaceWithSimilar](/reference/filter?id=replacewithsimilar).
- New function in Lastfm: [getSimilarArtists](/reference/lastfm?id=getsimilarartists).

## Version 1.4.4
- New filter [removeUnavailable](/reference/filter?id=removeunavailable).
- `Cache` can read/write files with `.txt` extension when explicitly specified in the file name.
- `getCustomTop` supports type `Date`, [details](https://github.com/Chimildic/goofy/discussions/46#discussioncomment-351974).
- Fix for logic error in `match` during selection.

## Version 1.4.3
- Now [getCustomTop](/reference/lastfm?id=getcustomtop) can create a top by albums.
- When sorting by release date, tracks preserve original order within their album, if they were initially in that order
- Bugfixes

## Version 1.4.2
- Now [craftTracks](/reference/source?id=crafttracks) can accept static `seed_*` different from `key`.
- New function for Lastfm: [getCustomTop](/reference/lastfm?id=getcustomtop).
- New function for Selector: [pickYear](/reference/selector?id=pickyear).
- New function for Order: [separateYears](/reference/order?id=separateyears).
- Improvement for search. If the same element is present multiple times in an array (i.e., has the same keyword for search), only one search request will be spent.
- `Source` functions related to playlists add an `origin` object to each track, containing the source playlist's `name` and `id`.
- An attempt was made to continue code execution after receiving exception `Exception: Address unavailable`, [details](https://github.com/Chimildic/goofy/discussions/27).

## Version 1.4.1
- Sped up execution time of `craftTracks` function.
- Found an undocumented Spotify API capability. Function [getRecomTracks](/reference/source?id=getrecomtracks) supports key `popularity`. Because of this it is **removed** from [craftTracks](/reference/source?id=crafttracks). Move it to the `query` parameter if you used it.
- Added ability to sort by album release date that the track belongs to in `Order.sort`.
- From the list of functions that can have a hidden trigger: `displayAuthResult`, `updateRecentTracks`, `logProperties`.

## Version 1.4.0
- **Removed** function `Source.getRecentTracks`. Use `RecentTracks.get` or `Cache.read` for the needed history file.
- New functions for Source: [mineTracks](/reference/source?id=minetracks), [craftTracks](/reference/source?id=crafttracks).
- New function for RecentTracks: [appendTracks](/reference/recenttracks?id=appendtracks).
- Structure of `SpotifyRecentTracks` file updated to a regular array of tracks (like other history files). Update will occur automatically on first trigger run. Before that moment `Cache.read` will return the old structure.
- Added save and delete library album functions to Library.

## Version 1.3.4
- New functions for Source: [getCategoryTracks](/reference/source?id=getcategorytracks), [getListCategory](/reference/source?id=getlistcategory).
- Added parameter [REQUESTS_IN_ROW](/config).
- When reading an empty file via Cache.read, an exception is thrown to prevent file overwrite on Google-side bug ([details](https://github.com/Chimildic/goofy/discussions/26)).
- New function [Playlist.saveWithUpdate](/reference/playlist?id=savewithupdate).
- Functions match* can accept an array of artists. In case of a track array, comparison by track title and album (without artist). In case of artist array, only the name.
- Added templates from forum to documentation (Back to this day, artist of the day)

## Version 1.3.3
- Optimization of Last.fm requests in the accumulation mechanism. Search only for tracks that are new to the listening history.
- New functions for Lastfm: [getSimilarTracks](/reference/lastfm?id=getsimilartracks), [getTopArtists](/reference/lastfm?id=gettopartists-1), [getTopAlbums](/reference/lastfm?id=gettopalbums).
- New functions for Source: [getRelatedArtists](/reference/source?id=getrelatedartists), [getAlbumsTracks](/reference/source?id=getalbumstracks).
- New function for Yandex: _getAlbums_._
- Now [dedupArtists](/reference/filter?id=dedupartists) can remove duplicates from an array of artists.
- Now [removeArtists](/reference/filter?id=removeartists) can remove by an array of artists.
- Adjustment of match* method group
- When searching and comparing, special characters (,!@# etc.) are removed from the string.
- More informative messages in logs for listening history and during search.

## Version 1.3.2
- Updated request sending mechanism. Many source functions execute faster due to asynchronous sending of N number of requests at once.
- Addition to mixin function. Now you can set ratio for more than two arrays. Details in [mixinMulti](/reference/combiner?id=mixinmulti).
- New functions: [getTopArtits](/reference/lastfm?id=gettopartists), [getArtistsTopTracks](/reference/lastfm?id=getartiststoptracks).

## Version 1.3.1
- New functions for Cache module: [rename](/reference/cache?id=rename), [remove](/reference/cache?id=remove), [clear](/reference/cache?id=clear), [compressArtists](/reference/cache?id=compressartists).
- Functions became public: [getArtists](/reference/source?id=getartists), [getArtistsAlbums](/reference/source?id=getartistsalbums), [getAlbumTracks](/reference/source?id=getalbumtracks).
- Function getTracksArtists **renamed** to getArtistsTracks.
- Repeated call to getSavedTracks in the same script sends new requests to Spotify, instead of returning previously obtained results. Use [sliceCopy](/reference/selector?id=slicecopy) to create a copy.
- Number of sent requests is now obtained through `CustomUrlFetchApp.getCountRequest`.
- Bugfix: spotify get with 404 interrupted script; lastfm with 500+ errors interrupted script.
- Bugfix: separateArtists did not separate artists.
- Many small fixes.

## Version 1.3.0
- Updated: installation instructions and video.
- Functions `removeTracks` and `removeArtists` added argument `invert` (inversion).
- Suppression of errors from lastfm to not interrupt script execution.
- Added anonymous tracking of library version distribution via Google Forms. Version value and script identifier are sent. To have an idea of the number of unique users.

## Version 1.2.0
- Added `parameters` for history tracking. Need to do [migration](https://4pda.ru/forum/index.php?act=findpost&pid=102495416&anchor=migrate_params).
- Listening history limit increased from 10 to 20 thousand.
- Lastfm history tracks get listening date added. You can use `rangeDateRel`.
- Lastfm listening accumulation mechanism, if you set `parameters`. So instead of `Lastfm.getRecentTracks` with a small number of tracks due to limits, you get many and quickly.
- Get history with a single function `RecentTracks.get`, regardless of `parameters`, including combined from two sources. Duplicates removed in combined, sorted from new to old listens.