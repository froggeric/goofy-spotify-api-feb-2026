# Clerk {docsify-ignore}

Methods for executing multiple functions at different times within a single trigger. More details in [best practices](/best-practices?id=Advanced-trigger)

| Method | Result Type | Brief Description |
|--------|-------------|-------------------|
| [runOnceAfter](/reference/clerk?id=runonceafter) | Boolean | Execute task every day after specified time. |
| [runOnceAWeek](/reference/clerk?id=runonceaweek) | Boolean | Execute task on a specific day of the week. |

## runOnceAfter

Execute task every day after specified time.

### Arguments :id=runonceafter-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `timeStr` | String | Time after which the task runs once. |
| `callback` | Function | Function to execute. Function name must be unique among all functions called by _Clerk_. |

### Return :id=runonceafter-return {docsify-ignore}

`isRun` (boolean) - `true` if function was executed, `false` if function was not started.

### Examples :id=runonceafter-examples {docsify-ignore}

1. Execute three functions at different times within one trigger

```js
// Trigger every hour
function updatePlaylists() {
    updateEveryHour() // every hour
    Clerk.runOnceAfter('15:00', updateInDay) // once a day
    Clerk.runOnceAfter('21:00', updateInEvening) // once a day

    function updateEveryHour() {
        // ...
    }

    function updateInDay() {
        // ...
    }

    function updateInEvening() {
        // ...
    }
}
```

## runOnceAWeek

Execute task on a specific day of the week.

### Arguments :id=runonceaweek-arguments {docsify-ignore}

| Name | Type | Description |
|------|------|-------------|
| `dayStr` | String | Day of week in English. |
| `timeStr` | String | Time after which the task runs once. |
| `callback` | Function | Function to execute. Function name must be unique among all functions called by _Clerk_. |

### Return :id=runonceaweek-return {docsify-ignore}

`isRun` (boolean) - `true` if function was executed, `false` if function was not started.

### Examples :id=runonceaweek-examples {docsify-ignore}

1. Execute three functions at different times within one trigger

```js
// Trigger every 15 minutes
function updatePlaylists() {
    update15() // every 15 minutes
    Clerk.runOnceAWeek('monday', '12:00', updateMonday) // every monday after 12
    Clerk.runOnceAWeek('saturday', '16:00', updateSaturday) // every saturday after 16

    function update15() {
        // ...
    }

    function updateMonday() {
        // ...
    }

    function updateSaturday() {
        // ...
    }
}
```
