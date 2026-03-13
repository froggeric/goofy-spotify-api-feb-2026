## Main Pages
- [Projects](https://script.google.com/home/my) - list of projects on your account, selecting a project opens the _code editor_
- [Executions](https://script.google.com/home/executions) - contains a list with _results_ of executed functions
- [Triggers](https://script.google.com/home/triggers) - schedule for executing functions

## Creating a Playlist

Let's complete the following task:
- take tracks from `daily mix` playlists
- remove liked tracks from them
- add recommendations
- create a playlist
- set a schedule (trigger)

1. Open the `main` file in the _code editor_ and paste the following code into it and __save__ (floppy disk icon or <kbd>Ctrl</kbd><kbd>S</kbd>):
```js
function createFirstPlaylist() {
    // 1 - collect
    let mixTracks = Source.getTracks([
        { name: 'Daily Mix 1', id: 'yourId' },
        { name: 'Daily Mix 2', id: 'yourId' },
    ]);
    let recomTracks = Source.craftTracks(mixTracks);
    let savedTracks = Source.getSavedTracks();   

    // 2 - process
    Combiner.push(mixTracks, recomTracks);   
    Filter.removeTracks(mixTracks, savedTracks);
    Selector.keepRandom(mixTracks, 20);
    
    // 3 - create playlist
    Playlist.saveWithReplace({
        // id: 'yourId',
        name: 'First Playlist',
        tracks: mixTracks,
        randomCover: 'update',
    });
}
```

2. The logical structure of the code consists of three blocks. First, tracks are requested. Then operations are performed on them. And finally, the command to create a playlist with the resulting set of tracks.
   
   Treat each line as a command to do something. Such commands are called _functions_. A function can be _defined_ (created) and _called_. That is, set an algorithm and execute it on our request. In this case, the `createFirstPlaylist` function is created, which inside itself calls Goofy library functions. A set of commands (function calls) forms code.

   To understand what the code does, just read it like a book. Every word has a definition in the dictionary. So here, every function has a description of what it does. Often the main meaning is conveyed by the function name itself. So knowing English or using a translator will help a lot. However, there are always nuances: what to pass to a function, what it will do, what it will return?

   Function descriptions are available in the _function list_. For example, let's find the description of the `Source.getTracks` function. In the left menu, right-click on `Module List` and open the page in a new tab. Go to that tab and find `Source`, then `getTracks`. We'll see the description and usage examples.

3. Currently the code will fail with an error. Because the `id` of the `daily mix` playlists is not set.
   
   To get the `id`, you need to go to the playlist page in Spotify and click `share`. This way we get a link to the playlist. From it, you need to extract the `id`.

   > If you open the `share` menu and press the `Alt` key, Spotify will offer to copy the URI instead of the link.

   ![Share](img/spotify-share.gif)

   Go to [this page](/reference/desc) in a new tab to see how to extract `id` from a link or URI.

   Go to the `Daily Mix` playlist page (Search section - For You - Daily Mix), copy the link, paste it into the code and remove everything extra, leaving only the `id`. Repeat for the second playlist.

   The result will be the following
   ```js
   let mixTracks = Source.getTracks([
       { name: 'Daily Mix 1', id: '491ZfFnGxaBF445JOhhxiO' },
       { name: 'Daily Mix 2', id: '426ZfFnGxaBF445JfOJefE' },
   ]);

   // Errors!
   // { name: 'Daily Mix 1', id: 'https://open.spotify.com/playlist/491ZfFnGxaBF445JOhhxiO?si=343F7972b107494a' },
   // { name: 'Daily Mix 1', id: 'spotify:playlist:426ZfFnGxaBF445JfOJefE' },
   ```
   
4. The rest of the code doesn't require configuration. On the control panel, select our function and click `run`.
   
   ![Run function](/img/run-func.gif)

   > You might see an error with number 500. Don't pay attention. Sometimes Spotify responds poorly to the recommendation request.

5. Go to Spotify. You now have a playlist named `First Playlist`. It doesn't have liked tracks. At the same time, some tracks came from recommendations, and some from daily mixes. This uncertainty was intentional. Because the code calls the `Selector.keepRandom` function, which makes a random selection of the specified number of tracks.

   Find the line with the `Combiner.push` function in the code. And put `//` symbols at the beginning of the line

   ```js
   // Combiner.push(mixTracks, recomTracks);
   ```
   
   You _commented out_ the line. Any text after `//` symbols is ignored, won't be called.
   
   Now find the lines `let recomTracks` and `Selector.keepRandom` - comment them out too.
   
   What did you do? Stopped requesting recommendations, so we don't merge them with tracks from mixes and removed random selection.

6. Run the function again. Now `First Playlist` will only have tracks from `daily mixes`, with likes removed from them.
   
   Note that a new playlist wasn't created, but the existing one was updated. Thanks to the `Playlist.saveWithReplace` function.
   
   However, there's an important nuance. The function searched among your playlists by name `First Playlist` to get the `id` and send requests to add tracks. Theoretically, your library may have several playlists with __identical__ names. The search won't be able to determine which specific playlist you want to add tracks to. Therefore, it will take the first one it finds. This may be undesirable, lead to an error, or update the wrong playlist.
   
   To avoid this, after the first function run, return to the code to explicitly specify the playlist `id`. That is, go to the playlist page the same way, copy the link and paste the `id` into the code. Where? In the list of arguments of the function associated with creating the playlist.
   
   In our example, you need to remove the comment sign `//` from `id` and insert the value itself
   ```js
   // Before
   Playlist.saveWithReplace({
       // id: 'yourId',
       name: 'First Playlist',
       tracks: mixTracks,
       randomCover: 'update',
   });

   // After
   Playlist.saveWithReplace({
       id: '476ZfFnGxaBF4В5JАhhxiO',
       name: 'First Playlist',
       tracks: mixTracks,
       randomCover: 'update',
   });
   ```

7. We created `First Playlist` thanks to the `createFirstPlaylist` function. Now let's learn how to create a schedule for automatic function launching using triggers.
   
   A trigger needs to be set a launch time. For example, every day at seven in the morning or every Monday. And also the name of the function to run.

   At the same time, we'll touch on the question of how to create several playlists at once.
   Currently in the `main` file you have one function. Add a _new function at the end of the file_ (don't forget to save the file):
```js
function createRandomSavedTracks() {
    let tracks = Source.getSavedTracks();

    Selector.keepRandom(tracks, 5);

    Playlist.saveWithReplace({
        name: 'Random Favorite Tracks',
        tracks: tracks,
    });
}
```

?> Your project has three files. `library` contains Goofy library functions. You use them to create playlists. The `config` file contains settings. In `main` your code is located. Since you can create (copy) many functions, it makes sense to create new files for logical separation. So that a huge wall of code doesn't accumulate in `main`.

8. In the left menu, select triggers
   
   ![Current project triggers](/img/fp-triggers-open.gif ':size=60%')
   
   Bottom right there's an `Add trigger` button
   - Select the `createRandomSavedTracks` function
   - Time-driven trigger
   - By minutes
   - Every minute
   - Save

9. Go to Spotify. In a minute a new playlist will appear and every following minute its content will be updated with 5 random favorite tracks.

   After checking this, go to the [trigger list](https://script.google.com/home/triggers). You'll see two triggers: for the `createRandomSavedTracks` function, created manually earlier, and for the `runTasks_` function, created automatically. Read more about it in [listening history](/details?id=Listening-history).
   
   Delete the trigger for the `createRandomSavedTracks` function: click three dots on the right, delete trigger. In the same menu, you can open the list of executions (results) of a *specific* trigger.
   
   Go to the [my executions](https://script.google.com/home/executions) section. You'll see a general list of completed or executing operations, their running time, completion status, logs.

## Continuing Work

Above you got acquainted with the basic principles of operation. It's recommended to continue acquaintance through [templates](/template). They show basic techniques for working with the library, they're easier to modify. Especially if you don't have programming skills.

Development of your own algorithms should start with describing the idea. What will be the source of tracks, what operations to perform (get recommendations, filter, etc.), where to save, how often to update. Then find the corresponding functions in the [documentation](/reference/index). Most often these are the `Source`, `Filter` and `Playlist` modules.

?> [On the forum](https://github.com/Chimildic/goofy/discussions) they'll help you figure out the error and create an algorithm. Always attach your code.

The next step will be optimization. To do this, familiarize yourself with [limitations](/details?id=Limitations) and [best practices](/best-practices).

Extended capabilities are available with [addons](/addon). For example, phone control.

## Syntax

-  `Function`
   ```js
   function myName(){
        // Function body
   }
   ```

   - The `function` keyword is required.
   - Then an arbitrary name, here it's `myName`.
   - Parentheses `()` for listing arguments (input data). Here there are no arguments.
   - Curly braces `{}` define the function boundary.
   - The `//` symbols for writing a comment.

- `Variable`
    ```js
    let tracks = 5;
    tracks = 10;
    ```

    - The `let` keyword is required when first declaring a variable.
    - Then an arbitrary name. Here it's `tracks`.
    - `= 5` assigns the value `5` to the `tracks` variable. Assignment from right to left.
    - Semicolon `;` is optional in this language. But desirable for avoiding complex errors.
    - On the second line, assignment of 10. The value 5 is lost.

-   Using `function` and `variable`
    ```js
    Module.myName(tracks);
    ```

    - Calling the `myName` function from the `Module` module and passing the `tracks` variable.
