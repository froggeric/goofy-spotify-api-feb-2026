# Spotify API February 2026 Changes - Goofy Impact Report

**Generated:** March 12, 2026  
**Goofy Version:** 2.4.1

---

## Executive Summary

This report provides a systematic analysis of how Spotify's February 2026 API changes affect the Goofy project. The project is **significantly impacted** with multiple core functions using deprecated endpoints and removed fields.

### Impact Level: HIGH

| Category | Count | Severity |
|----------|-------|----------|
| Removed Endpoints Used | 8 | Critical |
| Removed Fields Used | 7 | High |
| Renamed Endpoints | 4 | Medium |
| Reduced Limits | 1 | Medium |
| Potential Workarounds | 3 | - |

---

## Part 1: Affected Components

### 1.1 REMOVED ENDPOINTS - Critical Impact

#### 1.1.1 `GET /artists/{id}/top-tracks` - REMOVED

**Location:** `library.js` - `Source.getArtistsTopTracks()` and `Source.getArtistsByPath()`

```javascript
// Line ~220
function getArtistsTopTracks(artists, isFlat = true) {
    return getArtistsByPath(artists, '/artists/%s/top-tracks?market=from_token', isFlat);
}
```

**Impact:** 
- Function `Source.getArtistsTopTracks()` will completely fail
- Users cannot retrieve artist's top tracks anymore
- No direct replacement endpoint available

**Migration Options:**
1. Use `GET /artists/{id}` to get artist info, then use search to find popular tracks
2. Use recommendations API with artist as seed
3. Remove feature entirely

---

#### 1.1.2 `GET /browse/categories` and `GET /browse/categories/{id}` - REMOVED

**Location:** `library.js` - `Source.getListCategory()` and `Source.getCategoryTracks()`

```javascript
// Lines ~180-190
function getListCategory(params = {}) {
    let query = CustomUrlFetchApp.parseQuery(params);
    let url = `${API_BASE_URL}/browse/categories?${query}`;
    return SpotifyRequest.get(url).items;
}

function getCategoryTracks(category_id, params = {}) {
    let query = CustomUrlFetchApp.parseQuery(params);
    let url = `${API_BASE_URL}/browse/categories/${category_id}/playlists?${query}`;
    return getTracks(SpotifyRequest.get(url).items);
}
```

**Impact:**
- Browse category functionality completely broken
- Users cannot discover playlists by category
- No replacement available

---

#### 1.1.3 `GET /browse/new-releases` - REMOVED

**Location:** Documented in guide but no direct implementation found

**Impact:** Any templates or user code relying on new releases browsing will fail

---

#### 1.1.4 `GET /tracks`, `GET /albums`, `GET /artists` - REMOVED (Batch Endpoints)

**Location:** `library.js` - `SpotifyRequest.getFullObjByIds()`

```javascript
// Lines ~3400-3410
function getFullObjByIds(objType, ids, limit, market) {
    market = market ? `&market=${market}` : '';
    let requestCount = Math.ceil(ids.length / limit);
    let offset = limit;
    let urls = [];
    for (let i = 0; i < requestCount; i++) {
        let strIds = ids.slice(i * limit, offset).join(',');
        urls.push(`${API_BASE_URL}/${objType}/?ids=${strIds}${market}`);
        offset += limit;
    }
    // ...
}
```

**Impact:** CRITICAL - This function is used extensively throughout the codebase:

| Calling Function | File | Purpose |
|-----------------|------|---------|
| `Filter.removeUnavailable()` | library.js | Check track availability |
| `Source.getArtistsById()` | library.js | Get artist details |
| `Source.mineTracks()` | library.js | Filter by popularity |
| `Cache.cacheToFullObj()` | library.js | Get full track/artist/album objects |
| `HistoryManager.getTracks()` | addons | Get tracks by IDs |

**Migration Required:**
- Must change to individual `GET /tracks/{id}`, `GET /albums/{id}`, `GET /artists/{id}` calls
- Will significantly increase API call count (1 call per item instead of batch)
- May hit rate limits more frequently

---

#### 1.1.5 `GET /users/{id}` and `GET /users/{id}/playlists` - REMOVED

**Location:** `library.js` - `Playlist.getPlaylistArray()` and `Source.getFollowedPlaylists()`

```javascript
// Lines ~2600-2620
const getPlaylistArray = (function () {
    let playlistsOfUsers = {};
    return get;

    function get(userId) {
        let key = userId == null ? 'me' : userId;
        if (playlistsOfUsers[key] == null) {
            let path = userId == null ? 'me/playlists' : `users/${userId}/playlists`;
            playlistsOfUsers[key] = SpotifyRequest.getItemsByPath(path + ARGS);
        }
        return playlistsOfUsers[key];
    }
})();
```

**Impact:**
- Cannot fetch other users' profiles or playlists
- `Source.getFollowedPlaylists()` with specific userId will fail
- Only `me/playlists` (current user) still works

---

### 1.2 RENAMED ENDPOINTS - Medium Impact

#### 1.2.1 Playlist Track Endpoints → Items Endpoints

| Old Endpoint | New Endpoint | Status |
|--------------|--------------|--------|
| `POST /playlists/{id}/tracks` | `POST /playlists/{id}/items` | Must Update |
| `GET /playlists/{id}/tracks` | `GET /playlists/{id}/items` | Must Update |
| `DELETE /playlists/{id}/tracks` | `DELETE /playlists/{id}/items` | Must Update |
| `PUT /playlists/{id}/tracks` | `PUT /playlists/{id}/items` | Must Update |

**Location:** `library.js` - `Playlist` module

```javascript
// Lines ~2700-2800
function modifyTracks(requestType, data) {
    // ...
    let url = `${API_BASE_URL}/playlists/${data.id}/tracks`;
    // ...
}

function removeTracksRequest(id, tracks) {
    // ...
    let params = {
        url: `${API_BASE_URL}/playlists/${id}/tracks`,
        // ...
    }
}
```

**Impact:** All playlist modification operations will fail

**Migration:** Change `/tracks` to `/items` in all playlist URLs

---

### 1.3 REMOVED FIELDS - High Impact

#### 1.3.1 `popularity` Field - REMOVED from Track, Album, Artist

**Location:** Multiple locations

| Location | Code | Impact |
|----------|------|--------|
| `RangeTracks.isBelong()` | Line ~1650 | Filter by popularity broken |
| `Order.sort()` | Lines ~2100-2150 | Cannot sort by popularity |
| `Filter.removeUnavailable()` | Line ~1590 | Availability check logic affected |
| `Source.mineTracks()` | Line ~820 | Popularity filtering broken |
| `Source.getArtists()` | Line ~280 | Artist filtering by popularity broken |

```javascript
// Example: RangeTracks.isBelong() - Line ~1650
function isBelong(obj, args) {
    // ...
    // This will fail if args contains popularity filter
    // because obj.popularity is now undefined
}
```

**Impact:**
- All popularity-based filtering will silently fail (always returns false for popularity checks)
- Sorting by popularity will produce incorrect results
- Documentation mentions `popularity` as a filter parameter - this feature is now broken

---

#### 1.3.2 `available_markets` Field - REMOVED from Track, Album, Show, Audiobook, Chapter

**Location:** `library.js` - `Filter.removeUnavailable()`

```javascript
// Lines ~1570-1600
function removeUnavailable(tracks, market) {
    market = market || User.country;
    // ...
    function identifyState() {
        tracks.forEach((t) => {
            if (t.hasOwnProperty('available_markets') && t.available_markets.includes(market)) {
                availableState.push(t.id);
            } else {
                unclearState.push(t.id);
            }
        });
    }
    // ...
}
```

**Impact:**
- Track availability checking is now impossible
- All tracks will fall into "unclear" state
- `Filter.removeUnavailable()` will make individual API calls for every track (much slower)

---

#### 1.3.3 `followers` Field - REMOVED from Artist, User

**Location:** `library.js` - Multiple locations

```javascript
// Source.mineTracks() - filterByFollowers()
function filterByFollowers() {
    for (let i = 0; i < result.length; i++) {
        result[i] = getFullPlaylistObject(result[i]).filter((p) => {
            return isBelongRangeFollowers(p.followers.total);
        });
    }
}

// RangeTracks.isBelongArtist()
if (trackArtist.followers && typeof trackArtist.followers === 'object') {
    trackArtist.followers = trackArtist.followers.total;
}
```

**Impact:**
- Cannot filter artists by follower count
- Playlist follower count filtering may still work (from playlist object)

---

#### 1.3.4 `linked_from` Field - REMOVED from Track

**Location:** `library.js` - `Filter.removeUnavailable()`

```javascript
// Line ~1600
if (t.hasOwnProperty('is_playable') && t.is_playable) {
    let id = t.linked_from ? t.linked_from.id : t.id;
    availableState.push(id);
}
```

**Impact:** Track relinking detection will not work properly

---

#### 1.3.5 User Profile Fields - REMOVED

**Location:** `library.js` - `User` module

```javascript
// User.country - used in Filter.removeUnavailable()
function removeUnavailable(tracks, market) {
    market = market || User.country;  // This will now be undefined
    // ...
}
```

**Removed fields:**
- `country` - Used as default market for availability checks
- `email` - Not used in codebase
- `explicit_content` - Not used in codebase  
- `followers` - Not used in codebase
- `product` - Not used in codebase

**Impact:** Default market for availability checks will be undefined

---

### 1.4 REDUCED LIMITS - Medium Impact

#### 1.4.1 Search `limit` Parameter Reduced

| Parameter | Before | After |
|-----------|--------|-------|
| Maximum limit | 50 | 10 |
| Default limit | 20 | 5 |

**Location:** `library.js` - `Search` module

```javascript
// Lines ~3200-3250
function find(keywords, type, requestCount = 1) {
    const limit = 50;  // This must be changed to 10
    // ...
}
```

**Impact:**
- Search functions will return fewer results per request
- Need to implement pagination for larger result sets
- Code currently uses `limit = 50` which exceeds new maximum

---

### 1.5 Playlist Response Structure Changes

#### 1.5.1 Field Renames in Playlist Object

| Old Field | New Field |
|-----------|-----------|
| `tracks` | `items` |
| `tracks.tracks` | `items.items` |
| `tracks.tracks.track` | `items.items.item` |

**Location:** `library.js` - `Source.getItemsByPlaylistObject()`

```javascript
// Line ~920
function getItemsByPlaylistObject(obj) {
    let items = [];
    if (obj && obj.tracks && obj.tracks.items) {  // Must change to obj.items
        items = obj.tracks.total <= 100 ? obj.tracks.items : SpotifyRequest.getItemsByNext(obj.tracks);
        // ...
    }
    return items;
}
```

**Impact:**
- Playlist track retrieval will fail
- Only user's own playlists will return track contents
- Other playlists will only return metadata (no tracks)

---

## Part 2: Impact on Addons

### 2.1 HistoryManager (addons/app-listening-history/history-manager.js)

**Affected Code:**
```javascript
function getTracks() {
    return SpotifyRequest.getFullObjByIds('tracks', readTrackIds(), 50);
}
```

**Impact:** HIGH - Uses batch endpoint `GET /tracks` which is removed

**Migration:** Change to individual track fetch calls

---

### 2.2 Release (addons/genre-releases/release.js)

**Affected Code:**
```javascript
let albums = SpotifyRequest.getFullObjByIds('albums', albumIds, 20);
```

**Impact:** HIGH - Uses batch endpoint `GET /albums` which is removed

**Migration:** Change to individual album fetch calls

---

### 2.3 Radio (addons/fm-radio-tracks/radio.js)

**Status:** Not affected - Uses Search.multisearchTracks() which still works

---

### 2.4 Helper (addons/phone-launch/helper.js)

**Status:** Not affected - Only uses Cache operations, no Spotify API calls

---

## Part 3: Summary of Required Changes

### 3.1 Critical Changes (Must Fix)

| Priority | File | Function | Issue | Migration |
|----------|------|----------|-------|-----------|
| 1 | library.js | `SpotifyRequest.getFullObjByIds()` | Batch endpoints removed | Individual fetches |
| 2 | library.js | `Source.getArtistsTopTracks()` | Endpoint removed | Remove or use recommendations |
| 3 | library.js | `Source.getListCategory()` | Endpoints removed | Remove feature |
| 4 | library.js | `Source.getCategoryTracks()` | Endpoints removed | Remove feature |
| 5 | library.js | `Playlist.modifyTracks()` | Endpoint renamed | Change `/tracks` to `/items` |
| 6 | library.js | `Source.getItemsByPlaylistObject()` | Field renamed | Change `tracks` to `items` |
| 7 | library.js | `Search.find()` | Limit exceeded | Change limit from 50 to 10 |

### 3.2 High Impact Changes

| Priority | File | Function | Issue | Migration |
|----------|------|----------|-------|-----------|
| 8 | library.js | `RangeTracks.isBelong()` | `popularity` removed | Remove popularity filtering |
| 9 | library.js | `Filter.removeUnavailable()` | `available_markets` removed | Use `is_playable` field |
| 10 | library.js | `Order.sort()` | `popularity` removed | Remove popularity sorting |
| 11 | library.js | `User.country` | Field removed | Use config value or remove |

### 3.3 Addon Changes

| Priority | Addon | Function | Issue | Migration |
|----------|-------|----------|-------|-----------|
| 12 | history-manager.js | `getTracks()` | Batch endpoint | Individual fetches |
| 13 | release.js | `getAlbums()` | Batch endpoint | Individual fetches |

---

## Part 4: Detailed Migration Code Examples

### 4.1 Batch Endpoint Migration

**Before:**
```javascript
function getFullObjByIds(objType, ids, limit, market) {
    let urls = [];
    for (let i = 0; i < requestCount; i++) {
        let strIds = ids.slice(i * limit, offset).join(',');
        urls.push(`${API_BASE_URL}/${objType}/?ids=${strIds}${market}`);
    }
    return getAll(urls).reduce(...);
}
```

**After:**
```javascript
function getFullObjByIds(objType, ids, limit, market) {
    // Must use individual endpoints now
    let urls = ids.map(id => `${API_BASE_URL}/${objType}/${id}${market ? '?market=' + market : ''}`);
    // Use rate-limited parallel requests
    return getAll(urls).reduce(...);
}
```

**Note:** This will increase API calls significantly. Consider implementing caching and rate limiting.

---

### 4.2 Playlist Endpoints Migration

**Before:**
```javascript
let url = `${API_BASE_URL}/playlists/${data.id}/tracks`;
```

**After:**
```javascript
let url = `${API_BASE_URL}/playlists/${data.id}/items`;
```

---

### 4.3 Playlist Object Field Migration

**Before:**
```javascript
if (obj && obj.tracks && obj.tracks.items) {
    items = obj.tracks.total <= 100 ? obj.tracks.items : SpotifyRequest.getItemsByNext(obj.tracks);
}
```

**After:**
```javascript
if (obj && obj.items && obj.items.items) {
    items = obj.items.total <= 100 ? obj.items.items : SpotifyRequest.getItemsByNext(obj.items);
}
```

---

### 4.4 Search Limit Migration

**Before:**
```javascript
function find(keywords, type, requestCount = 1) {
    const limit = 50;
    // ...
}
```

**After:**
```javascript
function find(keywords, type, requestCount = 1) {
    const limit = 10;  // New maximum
    // Implement pagination if more results needed
    // ...
}
```

---

### 4.5 Remove Unavailable Tracks Migration

**Before:**
```javascript
tracks.forEach((t) => {
    if (t.hasOwnProperty('available_markets') && t.available_markets.includes(market)) {
        availableState.push(t.id);
    }
});
```

**After:**
```javascript
// available_markets is removed, use is_playable field instead
tracks.forEach((t) => {
    // Check is_playable from individual track fetch
    // This requires individual API calls for each track
});
```

---

## Part 5: Recommendations

### 5.1 Immediate Actions

1. **Update `SpotifyRequest.getFullObjByIds()`** - This is the most critical change affecting many parts of the codebase
2. **Update playlist endpoints** - Change `/tracks` to `/items` in all playlist URLs
3. **Update playlist field handling** - Change `tracks` to `items` in response parsing
4. **Update search limit** - Reduce from 50 to 10

### 5.2 Feature Deprecation Considerations

1. **Artist Top Tracks** - Consider removing or using recommendations API as workaround
2. **Browse Categories** - Remove this feature entirely (no replacement)
3. **Popularity Filtering** - Remove from documentation and code, or find alternative metrics

### 5.3 Documentation Updates Required

1. Remove references to `popularity` in filtering documentation
2. Remove browse category functionality from guides
3. Update playlist examples for new field names
4. Add rate limiting warnings due to increased API calls

### 5.4 Testing Recommendations

1. Test all playlist operations after migration
2. Test search with pagination
3. Test with Extended Quota Mode if possible (unaffected)
4. Monitor API rate limit errors after migration

---

## Part 6: Authentication & Deployment Impact

### 6.1 Current Architecture

Goofy uses a **distributed deployment model** where each user creates their own copy on Google Apps Script and configures their own Spotify API credentials.

**Configuration (config.js):**
```javascript
UserProperties.setProperty('CLIENT_ID', 'userValue');
UserProperties.setProperty('CLIENT_SECRET', 'userValue');
UserProperties.setProperty('PRIVATE_CLIENT_ID', 'userValue');
UserProperties.setProperty('PRIVATE_CLIENT_SECRET', 'userValue');
```

**Dual Credential System (library.js - Auth module):**
```javascript
const publicService = createService('spotify', false)
const privateService = KeyValue.CLIENT_ID == KeyValue.PRIVATE_CLIENT_ID 
    ? publicService 
    : createService('private-spotify', true)

// Private API endpoints require private service token
const PRIVATE_API = ['37i9d', '/users/.*/playlists', 'me/playlists', 
    '/recommendations', '/related-artists', '/audio-features', '/browse'];

function getHeaders(url, method = 'get') {
    let isPrivate = method == 'get' && isBelongPrivateAPI(url)
    return {
        Authorization: `Bearer ${isPrivate ? Auth.getPrivateAccessToken() : Auth.getPublicAccessToken()}`,
    }
}
```

**Purpose of dual credentials:**
- Spotify has different access levels for different apps
- Private credentials provided by Goofy author for enhanced features (recommendations, audio-features)
- Without private credentials, some features may not work

---

### 6.2 Development Mode Restrictions (Effective March 9, 2026)

| Restriction | Before | After | Impact on Goofy |
|-------------|--------|-------|-----------------|
| Premium account required | No | **Yes** | Users must have Spotify Premium |
| Client IDs per developer | Unlimited | **1** | **Cannot use dual credentials** |
| Users per app | Unlimited | **5** | Not an issue (1 user per deployment) |
| API endpoints | All | Limited | Many features broken |

---

### 6.3 Critical Issue: Client ID Limit

**Problem:** Goofy requires **2 Client IDs** (public + private), but new Development Mode apps are limited to **1 Client ID**.

#### Three User Scenarios

| Scenario | Description | Impact |
|----------|-------------|--------|
| **Existing users** (before March 9, 2026) | Already have 2 Client IDs | Grandfathered - keep both IDs, but endpoints still affected |
| **New users** (after March 9, 2026) | Can only create 1 Client ID | Cannot use dual-credential architecture |
| **Extended Quota Mode** | Special approval from Spotify | **Unaffected** - all endpoints/fields preserved |

---

### 6.4 Extended Quota Mode

Extended Quota Mode apps are **completely unaffected** by all February 2026 changes:
- All endpoints remain available
- All fields remain in responses
- No Premium requirement
- Higher rate limits

**How to Apply:**
1. Go to Spotify Developer Dashboard
2. Navigate to your app settings
3. Request Extended Quota Mode
4. Provide business justification

**Considerations:**
- Requires legitimate commercial/large-scale use case
- Not guaranteed approval
- Individual hobbyist users unlikely to qualify

---

### 6.5 Migration Options for New Users

#### Option A: Single Credential Mode

Use the same credentials for both public and private service:

```javascript
// config.js
UserProperties.setProperty('CLIENT_ID', 'yourValue');
UserProperties.setProperty('CLIENT_SECRET', 'yourValue');
UserProperties.setProperty('PRIVATE_CLIENT_ID', 'yourValue');      // SAME as CLIENT_ID
UserProperties.setProperty('PRIVATE_CLIENT_SECRET', 'yourValue');  // SAME as CLIENT_SECRET
```

**Result:** Works but with reduced functionality:
- Recommendations API may fail (Spotify restricted access)
- Audio-features API may fail
- All other endpoint/field restrictions still apply

---

#### Option B: Code Modification - Remove Private Service

Modify `Auth` module to use single service:

```javascript
// Simplified Auth - single service only
const Auth = (function () {
    const service = createService('spotify', false);
    return {
        hasAccess: () => service.hasAccess(),
        getAccessToken: () => service.getAccessToken(),
        // Remove getPrivateAccessToken, use getAccessToken everywhere
    };
})();
```

---

#### Option C: Apply for Extended Quota Mode

Recommended for users who want full functionality:
- All features work
- No API changes affect the app
- Requires application and approval

---

### 6.6 Summary Table: Authentication Impact

| Aspect | Before Feb 2026 | After Feb 2026 (Dev Mode) | Extended Quota |
|--------|-----------------|---------------------------|----------------|
| Premium required | No | **Yes** | No |
| Client IDs per user | Unlimited | **1** | Unlimited |
| Dual credentials | Possible | **Not possible** | Possible |
| Endpoint access | Full | Limited | Full |
| Field access | Full | Limited | Full |
| Rate limits | Standard | Standard | Higher |

---

### 6.7 Recommendations for Documentation

Update installation guide (`docs/install.md`) to include:

1. **Premium requirement warning** - New users must have Spotify Premium
2. **Single Client ID limitation** - Explain dual credential issue
3. **Extended Quota Mode option** - How to apply
4. **Fallback configuration** - Single credential setup instructions
5. **Timeline note** - Existing users grandfathered until March 9, 2026

---

## Appendix: Complete Endpoint Usage Map

| Endpoint | Status | Used In | Lines |
|----------|--------|---------|-------|
| `GET /tracks` | REMOVED | `SpotifyRequest.getFullObjByIds()` | ~3400 |
| `GET /albums` | REMOVED | `SpotifyRequest.getFullObjByIds()` | ~3400 |
| `GET /artists` | REMOVED | `SpotifyRequest.getFullObjByIds()` | ~3400 |
| `GET /artists/{id}/top-tracks` | REMOVED | `Source.getArtistsTopTracks()` | ~220 |
| `GET /browse/categories` | REMOVED | `Source.getListCategory()` | ~180 |
| `GET /browse/categories/{id}` | REMOVED | `Source.getCategoryTracks()` | ~185 |
| `GET /users/{id}` | REMOVED | (documented) | - |
| `GET /users/{id}/playlists` | REMOVED | `Playlist.getPlaylistArray()` | ~2600 |
| `GET /playlists/{id}/tracks` | RENAMED | `Playlist` module | ~2700-2800 |
| `POST /playlists/{id}/tracks` | RENAMED | `Playlist.addTracks()` | ~2700 |
| `DELETE /playlists/{id}/tracks` | RENAMED | `Playlist.removeTracksRequest()` | ~2750 |
| `PUT /playlists/{id}/tracks` | RENAMED | `Playlist.replaceTracks()` | ~2700 |
| `GET /search` | MODIFIED | `Search` module | ~3200 |
| `GET /recommendations` | AVAILABLE | `Source.craftTracks()` | ~850 |
| `GET /me/player/*` | AVAILABLE | `Player` module | ~1000-1100 |
| `GET /me/top/{type}` | AVAILABLE | `Source.getTop()` | ~150 |
| `GET /me/tracks` | AVAILABLE | `Source.getSavedTracks()` | ~940 |
| `GET /me/playlists` | AVAILABLE | `Playlist.getPlaylistArray()` | ~2600 |
| `GET /playlists/{id}` | AVAILABLE | `Playlist.getById()` | ~2570 |
