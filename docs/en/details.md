# How it Works

Goofy is a builder with a set of JavaScript functions. With them, you compose an algorithm that automates various tasks. For example, finding favorite tracks that haven't been listened to in a long time.

The algorithm execution happens on the Google Apps Script platform. That is, tasks are executed on a schedule without your participation.

?> During installation or version change, a single request is sent to Google Forms. To have an idea of how many users actively use Goofy.

## Differences from Smarter Playlists

The main difference is in the way the algorithm is composed. In Smarter Playlists there's a visual language, a diagram. In Goofy's case, a programming language is used.

Example of the following algorithm in Smarter Playlists: take tracks from two playlists, perform random sorting, save the first 50 tracks to a new playlist.

![Example of creating a playlist in Smarter Playlists](/img/SmarterPlaylistsExample1.png)

Now the same with Goofy
```js
let tracks = Source.getTracks([
  { name: 'Daily Mix 1', id: '123' },
  { name: 'Daily Mix 2', id: '456' },
]);

Order.shuffle(tracks);

Playlist.saveAsNew({
  name: 'Personal Daily Mix',
  tracks: Selector.sliceFirst(tracks, 50),
});
```

## Listening History

Goofy automatically tracks listening history. When the limit (60 thousand) is reached, new listens are still saved by deleting the oldest ones. The tracking process starts immediately after setup is complete. Listens that occurred before setup will not be included in the list. However, there's a way to add them manually if you have last.fm.

?> In a new last.fm account you can [add](https://support.last.fm/t/how-to-add-scrobbles-history-from-spotify-to-last-fm/40038) listening history that existed before the account was created.

The Spotify API has a function for getting listening history. But *maximum* 50 last tracks that were listened to for more than 30 seconds. Podcasts and tracks listened to in private mode don't make it into the history.

Goofy's ability to track history comes from the Apps Script platform and its access to Google Drive. Every 15 minutes Goofy accesses Spotify and updates the contents of the file on Drive with new data.

Theoretically, the history limit can be increased to 100 thousand. But the question arises of performance within the Apps Script platform limits, Drive space availability, and the overall feasibility of such an array. Since in most cases such history is needed to remove tracks from newly created playlists. Top-ranking tasks are easily solved by other means with Last.fm.

Last.fm can track Spotify listens itself (connection in [profile](https://www.last.fm/settings/applications)). But all of the above doesn't allow intercepting quick track skips. There are two solutions for this:

- The first is [described in detail on the forum](https://github.com/Chimildic/goofy/discussions/53).
- The second is to disable Last.fm tracking and use third-party programs for PC and smartphone that allow you to set an interval of several seconds after which a track is saved to history. For example, [Pano Scrobbler](https://4pda.to/forum/index.php?showtopic=887068).

In practice, the Spotify API works unstably. A track after 30 seconds may return with a delay or get lost altogether. The problem is on Spotify's side and cannot be solved. However, together with Last.fm this becomes quite insignificant.

If you frequently listen to downloaded tracks without internet connection, use third-party scrobblers as in the example above. They allow saving listens locally and removing dependence on offline and Spotify failures. If offline listening is a rare scenario, connecting through the profile is enough.

## Limitations

?> During initial acquaintance, some details may be unclear

The library is subject to limitations from the platforms. Below is a description of specific indicators and what they affect based on reference information offered by the platforms.

### Apps Script {docsify-ignore}
- Script execution (6 minutes / one execution)

  - The total maximum duration of *one* script run. As a rule, lightweight templates complete in a matter of seconds. Approaching a minute or several is possible in the case of large volumes of input and/or output data.
  - For example, the [getFollowedTracks](/reference/source?id=getfollowedtracks) function for user [spotify](https://open.spotify.com/user/spotify) and the `owned` argument runs on average in 4 minutes. While getting 1.4 thousand playlists and 102 thousand tracks. After removing duplicates, 78 thousand remain.
  - If you call [rangeTracks](/reference/filter?id=rangetracks) for 78 thousand, the 6-minute limit will be exceeded. But by preliminarily discarding unsuitable tracks, for example using [rangeDateRel](/reference/filter?id=rangedaterel), [match](/reference/filter?id=match) and others, you can significantly and quickly reduce the number of tracks.

- Number of requests (20 thousand / day)

  - As a rule, 1 request to Spotify is getting 50 playlists or 50 tracks. In some cases 100.
  - The example above got 1.4 thousand playlists and 102 thousand tracks in 1,735 requests.
  - Getting 11 thousand tracks from a playlist takes 110 requests and 25 seconds. About the same for creating a playlist with that many tracks.
  - Getting 10 thousand favorite tracks will take 200 requests.
  - In general, it's hard to imagine a function with 20 thousand requests due to the 6-minute execution limit. For this reason, you can say you can't go through all playlists of robot users with thousands of playlists. But a personal profile or average authors are possible.

- Trigger execution (90 minutes / day)

   - The total maximum duration of trigger execution. The only way to reach the limit is to call a 6-minute function 15 times in one day. It's hard to imagine a task that would require this and justify itself.

- Number of triggers (20 / user / script)
  
   - In rough terms, this is 20 playlists that are created on *completely different* schedules.
   - In practice, multiple functions can be called from one other function, which allows creating N playlists in one trigger. More details [here](/best-practices?id=Trigger-economy).
   - Additionally, you can create another copy of the library and also get a quota for 20 triggers.
  
     > If you need to create another copy, you can reuse CLIENT_ID and CLIENT_SECRET values and not create a new app on the Spotify side.

Other Apps Script limitations don't apply to the library. They're related to mail, tables and other services. Or are unattainable due to Spotify Web API limitations. More details [here](https://developers.google.com/apps-script/guides/services/quotas).

### Spotify Web API {docsify-ignore}
- Local files are ignored. [API doesn't allow](https://developer.spotify.com/documentation/general/guides/local-files-spotify-playlists/) adding such tracks to new playlists and they carry practically no data for filtering, sorting.
  
  > It is not currently possible to add local files to playlists using the Web API, but they can be reordered or removed.

- Number of tracks 
  - When adding to a playlist up to 11 thousand tracks.
  - When getting from one playlist also 11 thousand.
  - Favorite tracks up to 20 thousand.
  - When filtering, sorting, selecting the number is unlimited. But within the Apps Script quota.
- Number of playlists
  - Theoretically up to 11 thousand, but the Apps Script quota won't be enough to get tracks from them. The real value is within 2 thousand. Depends on the total number of tracks.
- Number of requests
  - There's no exact number. With too many requests in a short period of time, errors 500, 503 and similar may appear. They pass after a pause.
  
### Google Drive {docsify-ignore}
- The size of one text file is limited to 50 MB. To reduce volume you can use the [Cache.compressTracks](/reference/cache?id=compresstracks) function. Experimentally it was possible to create a file with 100 thousand **compressed** tracks and fit within 50 MB.
