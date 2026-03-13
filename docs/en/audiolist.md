# Audiolist

goofy `2.1+` can interact with the [Audiolist Android app](https://play.google.com/store/apps/details?id=ru.chimildic.audiolist). The app can simply launch any goofy function, or build complex interactions with parameter and track passing.

## Setup

### goofy

1. Update goofy to the latest version ([library file](https://github.com/Chimildic/goofy/blob/main/library.js)). To migrate from version `1.x` to `2.x`, use the [migration guide](/en/migrate2).
2. In the code editor, click _start deployment_ > _new deployment_. Make sure a configuration with web application and access for everyone is selected. Click _start deployment_.

   ![New deployment](/img/new-deploy-audiolist.png ':size=60%')

3. Copy the `deployment ID` and transfer it to the device where Audiolist is installed using your preferred method.

   !> Do not share the ID with other users. They will be able to run your functions.

### Audiolist

1. Create a program in Audiolist and add the `goofy function` command.
2. Enter the `deployment ID` obtained during goofy setup. For the _function name_, write `Audiolist.hello` as an example (built-in functions are available via the <kbd>fx</kbd> button).

  !["goofy function" command](/img/goofy-command-audiolist.jpg ':size=60%')

1. Exit the program editor and run the program. If everything is done correctly, a message from goofy will appear in the logs.

## Important

1. Correct function name in the command settings. Letter case matters.
2. Returning the result. Audiolist uses a typed programming language and expects a strict response format.
3. Updating the deployment after any goofy code changes. A deployment is an isolated copy. If you add/change a function and don't update the deployment, Audiolist won't be able to call it.

## Updating a Deployment

Click _start deployment_ > _manage deployments_. Select Audiolist from the left list, click the edit button. Select _new version_ from the list and click _start deployment_.

Sometimes Apps Script resets deployment names (appears to be a bug). You need to update the deployment whose ID is specified in the "goofy function" command.

   ![New deployment](/img/update-deploy-audiolist.png ':size=60%')

### Deployment Archive

Old deployment versions go to the archive. Apps Script sets a limit on the number of deployments (~200). If you're notified about the limit or have accumulated many unnecessary versions, select _version history_ from the left Apps Script menu. A _project history_ list will appear on the right. At the bottom of the list there is a _bulk delete versions_ button. It won't delete the current version.


## Usage

### Running a Function

When you need _only_ to run a function, return an empty response. The function name `doSomething` is specified in the corresponding field of the `goofy function` command.

```js
function doSomething() {
    // Function algorithm
    // ...

    return Audiolist.response()
}
```

### Response to Execution

#### Message

To return a result as a message, specify the text and type. The type determines the text color in the program logs: normal - `DEFAULT`, warning - `WARNINNG`, error - `ERROR`. When selecting `ERROR` type, the program on the Audiolist side will be interrupted.

```js
function doSomethingMessage() {
    // Function algorithm
    // ...

    let text = "This text will appear in Audiolist logs"
    return Audiolist.responseMessage(text, Audiolist.MESSAGE_TYPES.WARNINNG)
}
```

#### Tracks

For Audiolist to accept tracks from goofy, you need to specify their type: Spotify tracks - `SPOTIFY_TRACK`, last.fm tracks - `LASTFM_TRACK`, tracks as line-by-line text - `TEXT_TRACK`.


```js
function doSomethingItems() {
    let tracks = Source.getSavedTracks(20)
    // ...

    return Audiolist.responseItems(tracks, Audiolist.VARIABLE_TYPES.SPOTIFY_TRACK)
}
```

### Receiving Data

The function accepts a `data` argument, which contains `inputVariables` with an array of variables and an `ini` object with parameters from Audiolist.

#### Variables

Use the `data.getItems` function to get an array of elements by variable name.
To get all elements at once, use the `data.combineItems` function. Note that in the second case, all variables must be of the same type. For example, you cannot mix tracks from different platforms.

```js
function complexExample(data) {
    let variableItems = data.getItems('Variable name as in Audiolist')
    let allItems = data.combineItems()

    let modifiedItems = // ...

    return Audiolist.response({
        message: 'Message for logs',
        messageType: Audiolist.MESSAGE_TYPES.DEFAULT,
        variableType: Audiolist.VARIABLE_TYPES.SPOTIFY_TRACK,
        items: modifiedItems,
    })
}
```

#### Parameters

The `goofy function` command has a `parameters` field for sending strings and numbers.

The field is filled in `ini` format: each line specifies a `key=value` pair, where `key` is the identifier for the parameter, `value` is the value. The key name is arbitrary but without spaces and in English. Letter case matters.

For example, the task is to get 50 tracks from a cache file from goofy. To make the function universal, we'll make the file name `filename` and track count `count` parameters.

```
filename=SavedTracks
count=50
```

Then the function will receive them via `data.ini.filename` and `data.ini.count`. As a result, you can change the file and count only in the app. The `getTracksFromCache` function remains unchanged.

```js
function getTracksFromCache(data) {
    let tracks = Cache.read(data.ini.filename)

    return Audiolist.response({
        message: `${data.ini.count} tracks from file "${data.ini.filename}"`,
        messageType: Audiolist.MESSAGE_TYPES.DEFAULT,
        variableType: Audiolist.VARIABLE_TYPES.SPOTIFY_TRACK,
        items: Selector.sliceFirst(tracks, data.ini.count),
    })
}
```

In more complex scenarios, you may need arrays or nested objects. Array values are separated by commas. Nested objects are created with dots.

```
array=a,b,c
playlist.other.test=false

[playlist]
name=Playlist name
description=Description

[playlist.counters]
size=10
likes=3
```

The result will be `data.ini` with the following structure.
```
{
    array: ["a", "b", "c"],
    playlist: {
        name: "Playlist name",
        description: "Description",
        other: {
            test: false
        }
        counters: {
            size: 10,
            likes: 3
        }
    }
}
```

### Built-in Functions

The app has quick selection of built-in functions located in the `Audiolist` module.

- `Audiolist.hello` - returns a text message to check the connection with goofy.
  - `username` - test parameter for testing how to work with parameters.
- `Audiolist.getRecentTracks` - returns listening history from `RecentTracks.get`.
  - `limit` - track count limit, optional.
  - `sinceDays` and `beforeDays` - relative date filter, optional.
- `Audiolist.syncRecentTracks` - listening history synchronization. Assumes Audiolist will send history from monitoring for the day, so this function can add them to the goofy cache. The function will also return goofy history for the day so Audiolist can supplement its history (with a separate command). Since the app doesn't have access to plays from other devices.
  - `sinceDays` and `beforeDays` - history period adjustment, defaults to current day.
- `Audiolist.readCache` - read cache file.
  - `filename` - file name.
  - `limit` - element count limit, optional.
  - `sinceDays` and `beforeDays` - relative date filter, optional.
- `Audiolist.writeCache` - write to cache file.
  - `filename` - file name.
- `Audiolist.appendCache` - append to cache elements.
  - `filename` - file name.
  - `place` - add to beginning `begin` or end `end`.