# Useful Tips

Tips for working with goofy and Apps Script

## Algorithm Branching

An algorithm doesn't have to be linear. If for some reason you don't need to update a playlist, the function can be interrupted.

Let's say there's a daily playlist that shouldn't be updated on Wednesday. To implement this, we'll write a condition check. Thus creating a code branch.

?> Read more about how the `if` condition works separately [here](https://itchief.ru/javascript/сonditional-and-logical-operators)

The `return` keyword is used to return a value from a function. Code after this command is never executed. Using `return` without a return value is a special case where the receiving side gets an `undefined` value. When running from a trigger, this is simply equivalent to the function completing.

The example demonstrates that if today is Wednesday - we terminate the function. Code after `return` won't execute. But if today is another day of the week, execution won't enter the `if` branch, meaning the `return` command won't execute, and will continue - to the playlist update logic.
```js
if (Selector.isDayOfWeekRu('среда')){
    console.log('Today is Wednesday. The playlist will not be updated.');
    return;
}

// Playlist update logic
let tracks = Source.getPlaylistTracks('', 'id');
// ...
```

Another situation. We have a playlist with 40 tracks. We listen to it periodically. Let's add several branches depending on the number of listened tracks. For example, if 10 or fewer tracks haven't been listened to, we'll update all tracks. From 10 to 40 (exclusive), we'll remove listened tracks from the playlist. At 40, we won't touch the playlist.

```js
const ID_TARGET = 'result id';

let tracks = Source.getPlaylistTracks('', ID_TARGET);
Filter.removeTracks(tracks, RecentTracks.get());

if (tracks.length == 40) {
    return;
} else if (tracks.length > 10 && tracks.length < 40) {
    Playlist.saveWitUpdate({
        id: ID_TARGET,
        tracks: tracks,
    });
} else {
    Playlist.saveWithReplace({
        id: ID_TARGET,
        tracks: Source.getPlaylistTracks('', 'source id'),
    });
}
```

!> Pay close attention to the word `length`. It's very easy to make a typo in it.

## Running Debugging

Debugging allows you to stop code execution at any point and analyze values at that moment.

For example, you need to get all track names. But what key does this field have: `title`, `name`, `label`, `trackname`? Iterating through them is long and inconvenient. By running debugging, you'll see all available keys for a track and other elements.

?> Spotify documentation is located [here](https://developer.spotify.com/documentation/web-api/reference/)

Set a breakpoint by clicking next to the line number, select a function, and click `debug`.

![Debugging](/img/fp-debug.gif)

As a result, the debugger will open on the right, where you can view intermediate variable values, as well as continue execution step by step. There are 4 buttons at the top: run (to the next breakpoint or to the end), step over, step into, and step out. Try it yourself to understand in practice.

![Debugger](/img/debuger.png)

When running a function through a trigger, various information may be needed. For example, how many tracks were before and after filters. Use the `console.log` function to output such messages.
```js
let tracks = Source.getSavedTracks();
console.log('Number of liked tracks', tracks.length);
```

![Example logs](/img/example-log.png)

To solve the task of getting track names, we'll output them as follows
```js
let tracks = Source.getSavedTracks();
tracks.forEach(track => console.log(track.name));
```

Additionally, [keyboard shortcuts](https://github.com/Chimildic/goofy/discussions/112) are useful for quickly jumping to function descriptions right from the code editor.

## Multiple Accounts

One Google account (Apps Script) can manage multiple Spotify accounts (starting from goofy version 1.6.1).

To configure goofy for an additional Spotify account, repeat the [installation](https://chimildic.github.io/goofy/#/install) steps, but:

1. Copy the goofy project as you did in [step 4 during installation](https://chimildic.github.io/goofy/#/install).
2. Use the same values for `CLIENT_ID`, `CLIENT_SECRET`, `PRIVATE_CLIENT_ID`, `PRIVATE_CLIENT_SECRET` in the _config_ file (i.e., skip steps 1-2).
3. When granting access in step 10, log in through the new Spotify account.

Now you have two projects with goofy: the main one and a copy. Each is authorized under different Spotify accounts. The projects can interact with each other:

- Shared Drive. Using the [file path](/best-practices?id=file-path), [Cache](/reference/cache) functions can work with files from other folders. By default, each project writes files to a folder with the account id. So you can update files in one project and read them in another.
- By performing additional steps, you can call functions in the copy project while in the main project. A function launched this way will use the token of the account in whose project it resides.

### Managing the Copy

1. Go to the copy project settings and copy its identifier.

![Project identifier](/img/cross-project-id.png)

2. Now go to the main project. Next to the word `libraries`, press the plus button. Copy the project identifier from the first step and specify a readable name in the library identifier field. For example, the nickname of the second Spotify account.

![Adding library](/img/cross-add-library.png)

3. For the main project to use functions from the copy project, you need to create public functions in the copy. Usually this is just a wrapper over the goofy function. It's important to specify the return result with `return` and repeat the input arguments (sometimes there are more than one, check the documentation).

```js
// copy project
function saveWithReplace(data) {
  return Playlist.saveWithReplace(data)
}

function getFollowedTracks(params) {
  return Source.getFollowedTracks(params)
}

function getSavedTracks(limit) {
  return Source.getSavedTracks(limit)
}

function getUserId() {
  return User.id
}
```

Now you just need to access `CopyGoofy` to call functions.

```js
// main project
function example() {
  // Tracks from main
  let followedTracksFirstAccount = Source.getFollowedTracks({ type: 'followed' })
  // Tracks from copy
  let followedTracksSecondAccount = CopyGoofy.getFollowedTracks({ type: 'followed' })

  // Create playlist under main
  Playlist.saveWithReplace({
    // ...
  })

  // Create playlist under copy
  CopyGoofy.saveWithReplace({
    // ...    
  })

  // Due to Apps Script peculiarities, the following syntax is not possible. That's why wrapper functions are created.
  // Error: CopyGoofy.Playlist.saveWithReplace()
}
```

Also, the main project can read files directly from the copy's folder
```js
// main project
Cache.read(`root/${CopyGoofy.getUserId()}/filename.json`)
```

## Command Palette

The command palette is a list of actions available to the code editor. Here are some useful ones.

Place the cursor on any line of code and press <kbd>F1</kbd> or open the context menu with the right mouse button and select the palette. At the top there's an input field for quick command search. On the right of the name, keyboard shortcuts are indicated if available. For example, enter the word `font`.

![Command Palette - Font](/img/cmdp-font.gif)

- Quick copy. Place the cursor anywhere on a line. Hold <kbd>Shift</kbd><kbd>Alt</kbd> and press <kbd>↓</kbd>. The same effect occurs with a selected code fragment.
  
  ![Quick copy](/img/cmdp-fast-copy.gif)

- Vertical selection. Place the cursor in the required location. Hold <kbd>Shift</kbd><kbd>Alt</kbd> and drag the mouse to the opposite corner.
  
  ![Vertical selection](/img/cmdp-vertical-select.gif)

- Vertical selection without mouse. Navigate with arrows to the desired position. Hold <kbd>Ctrl</kbd><kbd>Alt</kbd> and press the up or down arrow. Now hold <kbd>Ctrl</kbd><kbd>Shift</kbd> and press the right or left arrow.
  
  ![Vertical selection without mouse](/img/cmdp-vertical-select-no-mouse.gif)

- Renaming. Select a word, for example a variable, and press <kbd>F2</kbd>. All occurrences will change to the new name.
  
  ![Variable renaming](/img/cmdp-rename-f2.gif)

- Moving a line. Place the cursor anywhere on the line. Hold <kbd>Alt</kbd> and press the up or down arrow. Similarly for a selection.
  
  ![Moving a line](/img/cmdp-move-code.gif)

- Comment action. Place the cursor anywhere on the line, press <kbd>Ctrl</kbd><kbd>/</kbd>, the line will be commented. Pressing again removes the comment. Similarly for a selected fragment.
  
  For multi-line comments, use the combination <kbd>Shift</kbd><kbd>Alt</kbd><kbd>A</kbd>
  
  ![Multi-line comment](img/cmdp-fast-comment.gif)

- Combination for quick formatting <kbd>Shift</kbd><kbd>Alt</kbd><kbd>F</kbd>
- Also useful may be collapsing and expanding all code. There's no shortcut, look in the palette.

## Finding Values

There are several ways to find an element's value. For example, artist genres or the minimum energy threshold for a track.

### Spotify Console

The simplest way is the Spotify console. An overlay that calls API methods with specified parameters.

1. Go to the [console](https://developer.spotify.com/console/) and find the necessary API method. For example, [artist by _id_](https://developer.spotify.com/console/get-artist/).
2. You need a token to make the request. Click the `get token` button. In the opened list, pay attention to the note `not require a specific scope`. If it's there, just click `request token`. Otherwise, below the note there will be a list of checkboxes that need to be clicked. Don't touch the large list at the bottom.
3. Add the artist _id_ to the field and click `try it`.

On the right side, the response will appear, giving all available artist attributes. For example, genres. These are what should be used in [rangeTracks](/reference/filter?id=rangetracks) for `genres` or `ban_genres`
```js
"genres": [
    "candy pop",
    "emo",
    "pixie",
    "pop emo",
    "pop punk"
]
```

### Step-by-step Debugging

A less convenient method. It largely depends on what data you need to find.

When [running debugging](/best-practices?id=running-debugging), set a breakpoint after the line of interest. For example, after getting a list of artists. The debugger will show an array of elements. It's inconvenient to search for a specific artist's value in it.

![Find artist genres](/img/find-genres.png)

### Logging Output

Elements are iterated in a loop, and the needed information is output to the log. Not suitable for finding data hidden inside the library. For example, [track features](/reference/desc?id=track-features-features) are not directly returned and functions for getting them are undocumented. Naturally, modifying the library code is allowed, but this method is not covered here.

```js
// Artist and their genres
artists.forEach(a => console.log(a.name, a.genres));

// List in format: artist - track
console.log(tracks.map(t => `${t.artists[0].name} - ${t.name}`).join('\n'));
```

## Advanced Trigger

In the [trigger economy](/best-practices?id=trigger-economy) section, a method is described for reducing triggers by combining functions with **similar schedules**. For example, updating daily playlists. That is, instead of the "1 trigger = 1 function" scheme, switching to "1 trigger = 2+ functions". This section describes the "1 trigger = all functions" method, regardless of schedule similarity.

As an illustration, the `runTasks_` function. It's activated by one trigger every 15 minutes, but executes three functions (tasks) at different times thanks to Clerk: updating listening history every 15 minutes, adding new likes to cache every day, and rewriting the likes cache, in case tracks are deleted, every week.

Clerk is the `Clerk` program module with two functions:
- `runOnceAfter` - execute a task once every day after a specified time of day.
- `runOnceAWeek` - execute a task on a specific day of the week after a specified time of day.

Function descriptions in the [documentation](/reference/clerk)

### Example Breakdown {docsify-ignore}

According to the notes in comments

1. A trigger is set every 15 minutes on the `runTasks_` function.
2. The `RecentTracks.update()` function runs every 15 minutes without additional conditions and checks.
3. Time condition check. If the `updateSavedTracks` function was launched, then `isUpdatedSavedTracks` contains `true`, otherwise it wasn't launched and `false`.
4. If the `updateSavedTracks` function wasn't launched - launch `appendSavedTracks` given a specified time. Otherwise don't launch.

```js
// Trigger: every 15 minutes (1)
function runTasks_() {
    RecentTracks.update() // runs every 15 minutes (2)
    let isUpdatedSavedTracks = Clerk.runOnceAWeek('monday', '01:00', updateSavedTracks) // (3)
    !isUpdatedSavedTracks && Clerk.runOnceAfter('01:00', appendSavedTracks) // (4)

    function updateSavedTracks(tracks) {
      // runs every Monday after 1 AM
    }

    function appendSavedTracks() {
      // runs every day after 1 AM
    }
}
```

## Hiding Functions

There are two ways to hide functions. Such a function cannot be launched directly in the code editor and is not available for triggers.

The first way. Useful for reducing the list of selectable functions when creating a trigger. Also, an example of use in [trigger economy](/best-practices?id=trigger-economy).

Add an underscore at the end of the name. In the example, the `create` function is available for launch, but `update_` is not.

```js
function create(){}

function update_(){}
```

- The `runTasks_` function is hidden this way (the trigger for it was created programmatically).
- It's not recommended to hide the `doGet` function. It's needed for authorization and [phone control](/addon?id=phone-control).
- The `setProperties` function can be hidden this way. But when you need to update parameters, you need to restore the normal name to get the ability to run in the editor.

The second way. Useful for highlighting repeating blocks. For example, when different sources require the same set of filters.

JavaScript allows defining a function inside another. Thus reducing the scope. In the example, the `get` function is available for calling inside `append`, but is not visible inside `update`.

```js
function update(){}

function append(){

    function get(){}
}
```

## Request Economy

Apps Script provides a daily quota for sending requests - 20 thousand. When the limit is reached, it's impossible to get tracks, modify playlists. The exact time of quota reset is unknown.

Functions that make many requests in one call have a corresponding note in [their description](/reference/index). Simpler functions can also take too much. The main reason is the number of tracks. For example, for 1 thousand liked tracks you'll need 10 requests. For 10 thousand already 100 requests. Projecting onto the quota, this is an insignificant amount for one day. Therefore, it's pointless to implement an accumulation mechanism through `Cache`. But it's easy to make logical errors.

?> Reducing the number of requests saves quota and execution time.

Let's say you need to remove liked tracks from a source and randomly select a dozen liked tracks for a playlist.
```js
// Correct variant
let topTracks = Source.getTopTracks('long');
let savedTracks = Source.getSavedTracks();

Filter.removeTracks(topTracks, savedTracks);
Selector.keepRandom(savedTracks, 10);
```

Several errors can be made. The examples are not made up, they were encountered in algorithms from goofy users.
```js
// Not creating a variable for savedTracks
// Error: requesting the same tracks twice
Filter.removeTracks(topTracks, Source.getSavedTracks());
let tracks = Selector.sliceRandom(Source.getSavedTracks(), 10);

// Calling Selector before Filter
// Error: not all will be removed from topTracks (unless this is specifically needed)
Selector.keepRandom(savedTracks, 10);
Filter.removeTracks(topTracks, savedTracks);

// Trying to fix the previous error
// Error: again extra requests
Selector.keepRandom(savedTracks, 10);
Filter.removeTracks(topTracks, Source.getSavedTracks());
```

1. When the algorithm requires the participation of the same set of elements, try changing the order. As done in the _correct variant_ above. That is, use the full set everywhere you need and only then modify it.
2. If this can't be done, create a copy instead of new requests.
```js
let savedTracks = Source.getSavedTracks();
let copySavedTracks = Selector.sliceCopy(savedTracks);
```

The `getCountRequest` function will return a value corresponding to the number of requests made from the beginning of execution to the function call. The value is not cached. At each startup, counting starts from zero. Add the following line of code at the end of your function. This way you can compare the quality of the optimization performed.
```js
console.log('Number of requests', CustomUrlFetchApp.getCountRequest());
```

At the same time, consider the influence of functions that can select a _random_ number of elements. For example, each artist has a different number of albums. Therefore, in the future this will affect the number of requests made at each new run.

## Trigger Economy

According to the [limitations](/details?id=limitations), one project (library copy) gets 20 triggers. At the same time, one is always occupied by updating the listening history.

Let's assume there are 3 daily playlists. Each function is called by a separate trigger.

```js
function createSavedAndForgot(){}

function createDailyMix(){}

function createRecom(){}
```

If the launch time doesn't matter much, let's combine the functions. Thus saving 2 triggers.
- [hide the functions](/best-practices?id=hiding-functions)
- delete previous triggers
- create a new trigger for the combining function `createEveryDayPlaylists`

```js
function createEveryDayPlaylists(){
    createSavedAndForgot_();
    createDailyMix_();
    createRecom_();
}

function createSavedAndForgot(){}
function createDailyMix(){}
function createRecom(){}
```

?> At the same time, it's important to monitor another limitation - execution time (6 minutes). For correct operation, the execution time of the combined functions must not exceed this limit.

## Google Drive

### File Path

Since version 1.6.1, file creation in folders is supported. All data is still placed in the root folder `Goofy Data`. Scripts from previous versions don't require changes.

Created folders are divided into two types:
- _Account folders_. Created automatically in the root with the Spotify account _id_ as the name.
- _Custom folders_. Created by you when using the [Cache](/reference/cache) module.

?> The described examples of working with folders are valid for all `Cache` functions

Example 1 - To create a file, specify only the name. It will be located in the _account folder_.
```js
Cache.write('MySavedTracks.json', []);
```

Example 2 - To create a _custom_ folder, specify in the string: folder name, slash `/`, file name. Below is an example of creating a file `example.json` in the `test` folder, which will be placed in the _account folder_.
```js
Cache.write('test/example.json', []);
```

Example 3 - A new _custom_ folder can be created in the root of `Goofy Data`. To do this, specify the reserved word `root` and add a slash `/`. Below is an example of creating a file `example.json`, which will be located in the `shared` folder in the root of `Goofy Data`.
```js
Cache.write('root/shared/example.json', []);

// Similarly
Cache.write('../shared/example.json', []);
```

Example 4 - If you don't want to get confused about locations, explicitly specify the reserved words `root` and `user` to distinguish the beginning of the path.
```js
// Shared folder in Goofy Data root
Cache.write('root/shared/example.json', []);
Cache.write('../shared/example.json', []);

// myfolder folder in account folder
Cache.write('user/myfolder/example.json', []);
Cache.write('./myfolder/example.json', []);
```

Example 5 - Folder nesting is not limited.
```js
Cache.write('root/shared/radio/rock/lastfm.json', []);
Cache.write('user/private/radio/pixie.json', []);
```

Example 6 - If you have multiple Spotify accounts using goofy projects on one Google account, you can create _shared files_. For example, one project collects tracks for a radio and saves to a shared folder, and the second account simply reads this file without spending its time on the same search.

?> You cannot place a file in the root folder `Goofy Data`. They will always be moved to the account folder. This is done so that most users don't have to manually change the Drive structure. Be sure to specify at least one folder that will be at the same level as account folders.

```js
// First project writes to file
Cache.write('root/shared/myradio.json', []);

// Second project reads it
let radioTracks = Cache.read('root/shared/myradio.json');
```

### Version Control

Files from [Cache](/reference/cache) are stored on [Google Drive](https://drive.google.com/). Including [listening history](/details?id=listening-history). A deleted file goes to the trash, where it's available for another 30 days. Each file has up to 100 versions. Each write to the same file creates a new version.

If you need to roll back to a previous file version
1. Go to the `Goofy Data` folder on [Google Drive](https://drive.google.com/)
2. Right-click on the file and select `manage versions`
3. Find the version by modification date and download it from the three-dot menu
4. In the same window, click `upload new version` and select the previously downloaded file

?> When rolling back listening history, don't leave an empty file. It should have at least an empty array `[]`.

## keep and slice

Among the `Selector` functions, there's a group that starts with the words `keep` and `slice`. When to use which?

Their difference lies in the absence and presence of a return value:
- The `keep` group internally calls the `replace` function. Thus replacing elements of the original array with new ones.
- The `slice` group doesn't modify the original array. A new array is created and returned as the result.

```js
// This code
let tracks = Source.getTopTracks('long');
tracks = Selector.sliceFirst(tracks, 10);

// Is equivalent to this
let tracks = Source.getTopTracks('long');
Selector.keepFirst(tracks, 10);
```

Then why are two groups needed? It depends on the context of use. With `keep` the code looks cleaner. No constant assignments to the same variable.

The `slice` functions are useful for combining:
```js
// Selection in one line
let tracks = Selector.sliceRandom(RecentTracks.get(), 100);

// Selection before playlist update
Playlist.saveWithReplace({
    // ...
    tracks: Selector.sliceFirst(tracks, 50),
});
```