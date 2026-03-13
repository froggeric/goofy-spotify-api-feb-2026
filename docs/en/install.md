# Installation

You create your own copy of the library. Only you have access to everything that happens in this copy.

Performed once.

?> **Spotify API Changes (February 2026)** - Spotify introduced Development Mode with significant limitations. For full functionality, you should apply for Extended Quota Mode. See details below.

## Development Mode vs Extended Quota Mode

Spotify now operates in two modes for third-party applications:

| Feature | Development Mode | Extended Quota Mode |
|---------|-----------------|---------------------|
| Client IDs | 1 per developer | Multiple |
| Premium Required | No | Yes (required to apply) |
| Request Quota | Limited | Extended |
| Recommendations API | Limited/Unavailable | Full access |
| Search API | Limited | Full access |

**Recommendation:** Apply for Extended Quota Mode for full library functionality:
1. Go to [Spotify Dashboard](https://developer.spotify.com/dashboard/)
2. Select your app
3. Navigate to Settings
4. Request Extended Quota Mode (requires Spotify Premium)

## Step-by-Step Installation

1. Go to [Spotify Dashboard](https://developer.spotify.com/dashboard/) and click `Log in`.

2. Click the `create app` button and fill out the form. Redirect URI: `https://chimildic.github.io/spotify/auth`

   ![Create app](/img/install-step-create-app.png ':size=40%')

3. Go to the [library in Apps Script](https://script.google.com/d/1DnC4H7yjqPV2unMZ_nmB-1bDSJT9wQUJ7Wq-ijF4Nc7Fl3qnbT0FkPSr/edit?usp=sharing). Sign in to your Google account if required.

4. Select `Overview` from the dropdown menu on the left.

   ![Open menu](/img/general-property.gif ':size=60%')

   On the opened page, on the right `Make a copy`. A copy created on your account will open. Rename if needed (click on the name at the top of the page).
   
    ![Make a copy](/img/install-step-copy.png)

5. Go to the `config.gs` file. Insert `CLIENT_ID` and `CLIENT_SECRET` instead of the words `yourValue`. Get the values from the Spotify app created in step 2 (`Settings` button).

   ![Client ID and Client Secret](/img/install-step-client-id2.png)

6. Also specify values for `PRIVATE_CLIENT_ID` and `PRIVATE_CLIENT_SECRET` by getting them [here](https://script.google.com/macros/s/AKfycbwwDT25i71nYAk1aICxnrXfFVDzctcmhRMqzugjEkpqmUWjGATAbMOCL5aqvlPXOIq4/exec).
   If private keys are unavailable, copy your regular `CLIENT_ID` and `CLIENT_SECRET`. In this case, recommendations from Spotify are unavailable due to their updated policies.
   If recommendations are important, ask to increase the limit in the [Telegram chat](https://t.me/forum_goofy).

   **Important:** Set `USER_MARKET` to your Spotify market code (e.g., `US`, `RU`, `GB`). This is required since Spotify removed country detection from the `/me` endpoint in February 2026. Without it, some functions may fail or return incorrect results.

   Save the changes with <kbd>Ctrl</kbd><kbd>S</kbd> or the floppy disk icon on the action bar

7. Run the `setProperties` function in the editor.

   ![run setProperties](/img/install-run-setProperties.png)

   You'll see a popup message about the need to grant access permissions. Agree to grant them.

   ![permission request](/img/install-permission-request.png ':size=50%')

   Select the Google account where you created the library copy.

   ![Select account](/img/install-step-account.png)

   Click `Advanced`, then `Go to "Copy of Goofy (Ver. 1)"`

   ![Select account](/img/install-step-warning.png ':size=50%')

   Click the `Allow` button at the bottom of the window.

   ![Select account](/img/install-step-grant-permissions.png)

8. The window will close. Select `New deployment` - `Test deployments`

   ![Deploy web app](/img/install-step-webapp.png ':size=40%')

   Follow the link from `web app`. Grant access permissions.

   Installation and setup complete. Proceed to [first playlist](/first-playlist).
