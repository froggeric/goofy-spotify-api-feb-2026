# Errors

If your error is not on this page, write to [Telegram](https://t.me/forum_goofy) or the [forum](https://github.com/Chimildic/goofy/discussions). Describe your actions in detail and add the code.

## Apps Script

| Error | Solution |
|-|-|
| Access not granted or expired | No access to Spotify. Often occurs due to password change. Go through authorization again (`New deployment` - `Test deployment` - `Web app`). |
| Exceeded maximum execution time | Function execution time [exceeded 6 minutes](/details?id=Limitations). When each run takes several minutes, revise the algorithm, optimize according to documentation notes and [best practices](/best-practices). In rare cases observed in functions that usually complete in seconds. Currently explained by platform lag, since it's impossible to reproduce the error for debugging. |
| Limit exceeded | One of the platform limits exceeded. In the classic version, limitations are described [here](/details?id=Limitations). If experimenting with other platform capabilities, there are [others](https://developers.google.com/apps-script/guides/services/quotas). |
| Service invoked too many times </br> Service using too much computer time for one day | Special case of previously described errors. The exact difference is unknown. In the best case, will pass after a short pause. In the worst case, on quota update, i.e., after 24 hours. |
| Client ID is required | Client ID from Spotify is missing in saved parameters. Call the `setProperties` function from the `config` file. In this case, the `CLIENT_ID` and `CLIENT_SECRET` parameters must have values from the Spotify developer dashboard (not the text `yourValue`) |
| Address unavailable | Cause of occurrence unknown. When it occurs, a pause is automatically waited and a retry request is sent. As a rule, the error doesn't occur on the second attempt. |
| Service error: Drive | Cause of occurrence unknown. Since version 1.5.4, an attempt is made to catch the error and repeat the operation. |
| ReferenceError: Cheerio is not defined | Add the [Cheerio](https://github.com/Chimildic/goofy/discussions/91#discussioncomment-1931923) library. |

## Requests

| Status | Description |
|-|-|
| 400 | Server received an incorrect request. Write to the forum or Telegram if the error is consistently reproducible. |
| 401 </br> 403 | Request is composed correctly, but access is missing. [Update access permissions](/tuning?id=Update-access-permissions). Note that [Player](/reference/player) functions are only available with an active subscription (Spotify limitation). |
| 404 | Requested data not found. For example, a non-existent playlist `id` was specified. |
| 413 | Known case of occurrence when uploading playlist cover. During encoding, data size exceeded 256 KB. Therefore Spotify rejected the request. Use another cover. With `randomCover` with value `update`, the cover will update next time. |
| 429 | Too many requests per unit of time. The server stopped responding with content. As a rule, a pause of several seconds is automatically waited and the process resumes. |
| 500 </br> 503 | Internal error on the Spotify or Last.fm server side. In the normal case, will disappear after a few minutes. During critical failures, it may continue for an indefinite time. Nothing can be fixed on your side. |
| 504 | Response wait time from server exceeded. There will be an attempt to retry the request. |
