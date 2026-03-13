# Spotify API February 2026 - Goofy Migration Plan

**Version:** 3.0  
**Date:** March 12, 2026  
**Status:** Ready for Implementation  
**Last Updated:** After Second-Pass Critical Code Review

---

## Executive Summary

This document provides a comprehensive migration plan for the Goofy project to adapt to Spotify's February 2026 API changes. The plan prioritizes backward compatibility and minimal user disruption while addressing all affected components.

### Impact Overview

| Category | Count | Priority |
|----------|-------|----------|
| Removed Endpoints | 8 | Critical |
| Renamed Endpoints | 4 | High |
| Removed Fields | 7 | High |
| Reduced Limits | 1 | Medium |
| Authentication Changes | 1 | Critical |

---

## ⚠️ CRITICAL: Pre-Migration Checklist

Before implementing ANY changes, ensure:

1. **Backup your files:**
   ```bash
   cp library.js library.js.backup
   cp config.js config.js.backup
   ```

2. **Set USER_MARKET** - This is NOW REQUIRED:
   ```javascript
   UserProperties.setProperty('USER_MARKET', 'US');  // Set to your market
   ```

3. **Test after each phase** - Do not proceed to next phase until current phase tests pass.

---

## Table of Contents

1. [Section 1: Batch Endpoints Migration](#section-1-batch-endpoints-migration)
2. [Section 2: Artist Top Tracks Migration](#section-2-artist-top-tracks-migration)
3. [Section 3: Browse Categories Removal](#section-3-browse-categories-removal)
4. [Section 4: Playlist Endpoints Rename](#section-4-playlist-endpoints-rename)
5. [Section 5: Playlist Field Rename](#section-5-playlist-field-rename)
6. [Section 6: Search Limit Migration](#section-6-search-limit-migration)
7. [Section 7: Popularity Field Removal](#section-7-popularity-field-removal)
8. [Section 8: available_markets Field Removal](#section-8-available_markets-field-removal)
9. [Section 9: followers Field Removal](#section-9-followers-field-removal)
10. [Section 10: linked_from Field Removal](#section-10-linked_from-field-removal)
11. [Section 11: User Country Field Removal](#section-11-user-country-field-removal)
12. [Section 12: Audio Features Migration](#section-12-audio-features-migration)
13. [Section 13: Recommendations Graceful Degradation](#section-13-recommendations-graceful-degradation)
14. [Section 14: Addons Migration](#section-14-addons-migration)
15. [Section 15: Authentication Migration](#section-15-authentication-migration)
16. [Implementation Checklist](#implementation-checklist)
17. [Testing Guide](#testing-guide)
18. [Rollback Plan](#rollback-plan)

---

## Section 1: Batch Endpoints Migration

### Background

Spotify removed batch endpoints:
- `GET /tracks?ids=...`
- `GET /albums?ids=...`
- `GET /artists?ids=...`
- `GET /audio-features?ids=...`

### Current State (ACTUAL CODE - Already Partially Migrated)

**File:** `library.js`  
**Function:** `SpotifyRequest.getFullObjByIds()` (line ~3440)

**Current Code (VERIFIED):**
```javascript
function getFullObjByIds(objType, ids, limit, market) {
    // Spotify removed batch endpoints in Feb 2026
    // Migration: Use individual endpoints /tracks/{id}, /albums/{id}, /artists/{id}
    // The 'limit' parameter is deprecated but kept for backward compatibility
    if (!ids || ids.length === 0) {
        return [];
    }
    const marketParam = market ? `?market=${market}` : '';
    const urls = ids.map(id => `${API_BASE_URL}/${objType}/${id}${marketParam}`);
    // getAll() uses fetchAll() with built-in rate limiting (batches of 20)
    return getAll(urls).reduce((fullObj, response) => {
        response && Combiner.push(fullObj, response);
        return fullObj;
    }, []);
}
```

### Issues Found

1. **Audio-features endpoint still uses batch** - The code for audio-features (line ~3172-3180) still calls `getFullObjByIds('audio-features', ...)` but the batch endpoint was removed!

### Required Fix for Audio Features

**Location:** `getCachedTracks()` → `cacheToFullObj()` (line ~3172-3180)

**Current Code:**
```javascript
if (uncachedTracks.features.length > 0) {
    // limit = 100, но UrlFetchApp.fetch выдает ошибку о превышении длины URL
    // При limit 85, длина URL для этого запроса 2001 символ
    let features = SpotifyRequest.getFullObjByIds('audio-features', uncachedTracks.features, 85);
    features.forEach((item, i) => {
        // ...
    });
}
```

**After (FIX for batch endpoint removal):**
```javascript
if (uncachedTracks.features.length > 0) {
    // MIGRATION: Spotify removed batch audio-features endpoint Feb 2026
    // Now using individual endpoints: /audio-features/{id}
    let features = SpotifyRequest.getFullObjByIds('audio-features', uncachedTracks.features, 85);
    features.forEach((item, i) => {
        if (item == null) {
            let id = uncachedTracks.features[i];
            cachedTracks.features[id] = {};
            Admin.printInfo(`У трека нет features, id: ${id}`);
        } else {
            item.anger = item.energy * (1 - item.valence);
            item.happiness = item.energy * item.valence;
            item.sadness = (1 - item.energy) * (1 - item.valence);
            cachedTracks.features[item.id] = item;
        }
    });
}
```

**Note:** The `getFullObjByIds` function now handles audio-features correctly since it already builds individual URLs. No code change needed, but API calls will increase significantly.

### Impact Analysis

| Metric | Before | After |
|--------|--------|-------|
| API calls for 50 tracks | 1 | 50 |
| API calls for 85 audio-features | 1 | 85 |
| Rate limit handling | Automatic (fetchAll) | Automatic (fetchAll) |
| Batch size | 20 parallel requests | 20 parallel requests (configurable) |

### Configuration Recommendation

```javascript
// In config.js - reduce concurrent requests if rate limiting issues occur
UserProperties.setProperty('REQUESTS_IN_ROW', '10');  // Default is 20
```

---

## Section 2: Artist Top Tracks Migration

### Background

Spotify removed `GET /artists/{id}/top-tracks` endpoint.

### Current State (NOT YET MIGRATED)

**File:** `library.js`  
**Function:** `Source.getArtistsTopTracks()` (line ~530)

**Current Code (VERIFIED):**
```javascript
function getArtistsTopTracks(artists, isFlat = true) {
    return getArtistsByPath(artists, '/artists/%s/top-tracks?market=from_token', isFlat);
}
```

### Problem

The endpoint `/artists/{id}/top-tracks` was removed. This function will now return 404 errors.

### Solution

Use a hybrid approach with Last.fm API as primary source and Recommendations API as fallback.

**IMPORTANT:** The `getArtistsByPath()` helper is shared with `getRelatedArtists()`. The `/artists/{id}/related-artists` endpoint was **NOT removed** - only `/artists/{id}/top-tracks` was removed. Therefore, we need a separate implementation.

### Updated Code

**Replace the entire `getArtistsTopTracks` function:**

```javascript
function getArtistsTopTracks(artists, isFlat = true) {
    // MIGRATION: Spotify removed /artists/{id}/top-tracks endpoint Feb 2026
    // Priority: Last.fm API (if configured) → Recommendations API fallback
    
    if (!artists || artists.length === 0) {
        return isFlat ? [] : [];
    }
    
    // Try Last.fm first if API key is configured
    if (KeyValue.LASTFM_API_KEY) {
        try {
            return getArtistsTopTracksFromLastfm(artists, isFlat);
        } catch (e) {
            Admin.printInfo('Last.fm top tracks failed, using recommendations fallback:', e.message || e);
        }
    }
    
    // Fallback to recommendations API
    return getArtistsTopTracksFromRecommendations(artists, isFlat);
}

/**
 * Primary: Last.fm API for artist top tracks
 * @private
 */
function getArtistsTopTracksFromLastfm(artists, isFlat = true) {
    const LASTFM_API_BASE_URL = 'https://ws.audioscrobbler.com/2.0/?';
    
    let resultGroups = artists.map(artist => {
        if (!artist || !artist.name) {
            return [];
        }
        
        let queryObj = {
            method: 'artist.gettoptracks',
            artist: artist.name,
            limit: 20,
            api_key: KeyValue.LASTFM_API_KEY,
            format: 'json',
            autocorrect: '1'
        };
        
        let url = LASTFM_API_BASE_URL + CustomUrlFetchApp.parseQuery(queryObj);
        let response;
        
        try {
            response = CustomUrlFetchApp.fetch(url);
        } catch (e) {
            Admin.printInfo('Last.fm API error for artist:', artist.name, e.message || e);
            return [];
        }
        
        if (!response || !response.toptracks || !response.toptracks.track) {
            return [];
        }
        
        // Handle both array and single object responses
        let lastfmTracks = response.toptracks.track;
        if (!Array.isArray(lastfmTracks)) {
            lastfmTracks = [lastfmTracks];
        }
        lastfmTracks = lastfmTracks.slice(0, 20);
        
        // Convert to search format
        let searchItems = lastfmTracks.map(t => ({
            name: t.name,
            artist: { name: t.artist?.name || artist.name }
        }));
        
        // Search Spotify for these tracks
        let spotifyTracks = Search.multisearchTracks(searchItems, item => 
            `${item.artist.name} ${item.name}`
        );
        
        // Filter to only include tracks where the artist matches
        return spotifyTracks.filter(track => {
            if (!track || !track.artists) return false;
            return track.artists.some(a => 
                a.id === artist.id || 
                a.name.toLowerCase() === artist.name.toLowerCase()
            );
        });
    });
    
    return isFlat ? resultGroups.flat(1) : resultGroups;
}

/**
 * Fallback: Recommendations API (lower quality but always available)
 * Note: Recommendations endpoint may be unavailable in Development Mode
 * @private
 */
function getArtistsTopTracksFromRecommendations(artists, isFlat = true) {
    let resultGroups = [];
    let batchSize = 5; // Max 5 seed artists per recommendations call
    
    for (let i = 0; i < artists.length; i += batchSize) {
        let batch = artists.slice(i, i + batchSize);
        let seedIds = batch.filter(a => a && a.id).map(a => a.id).join(',');
        
        if (!seedIds) {
            batch.forEach(() => resultGroups.push([]));
            continue;
        }
        
        let url = createUrlForRecomTracks({
            seed_artists: seedIds,
            limit: 100
        });
        
        let response;
        try {
            response = SpotifyRequest.get(url);
        } catch (e) {
            Admin.printInfo('Recommendations API error:', e.message || e);
            batch.forEach(() => resultGroups.push([]));
            continue;
        }
        
        if (!response || !response.tracks) {
            batch.forEach(() => resultGroups.push([]));
            continue;
        }
        
        let artistIdSet = new Set(batch.filter(a => a && a.id).map(a => a.id));
        let tracksByArtist = {};
        batch.forEach(a => {
            if (a && a.id) tracksByArtist[a.id] = [];
        });
        
        response.tracks.forEach(track => {
            if (track && track.artists) {
                track.artists.forEach(a => {
                    if (artistIdSet.has(a.id) && tracksByArtist[a.id].length < 10) {
                        tracksByArtist[a.id].push(track);
                    }
                });
            }
        });
        
        batch.forEach(a => {
            resultGroups.push(a && a.id ? (tracksByArtist[a.id] || []) : []);
        });
    }
    
    return isFlat ? resultGroups.flat(1) : resultGroups;
}
```

### Limitations

| Approach | Accuracy | API Calls | Requirements |
|----------|----------|-----------|--------------|
| Last.fm | High (true top tracks) | Last.fm + Spotify search | LASTFM_API_KEY configured |
| Recommendations | Medium (similar tracks) | 1 per 5 artists | Extended Quota Mode (may fail in Dev Mode) |

---

## Section 3: Browse Categories Removal

### Background

Spotify removed:
- `GET /browse/categories`
- `GET /browse/categories/{id}`

### Current State (NOT YET MIGRATED)

**File:** `library.js`  
**Functions:** `Source.getListCategory()` and `Source.getCategoryTracks()` (line ~518-528)

**Current Code (VERIFIED):**
```javascript
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

### Solution

Replace with deprecation errors that provide migration guidance.

**Updated `getListCategory()`:**
```javascript
/**
 * @deprecated REMOVED - Spotify API endpoint /browse/categories removed Feb 2026
 * @throws Error with migration guidance
 */
function getListCategory(params = {}) {
    throw new Error(
        'DEPRECATED: Source.getListCategory() is no longer available.\n' +
        'Spotify removed the /browse/categories endpoint in February 2026.\n\n' +
        'Alternatives:\n' +
        '1. Use Source.mineTracks({ keyword: ["genre"], type: "playlist" })\n' +
        '2. Use Source.getRecomTracks({ seed_genres: "genre_name" })\n' +
        '3. Use known playlist IDs directly with Source.getTracks()\n\n' +
        'See: https://chimildic.github.io/goofy/#/reference/source'
    );
}
```

**Updated `getCategoryTracks()`:**
```javascript
/**
 * @deprecated REMOVED - Spotify API endpoint /browse/categories/{id} removed Feb 2026
 * @throws Error with migration guidance
 */
function getCategoryTracks(category_id, params = {}) {
    throw new Error(
        'DEPRECATED: Source.getCategoryTracks() is no longer available.\n' +
        'Spotify removed the /browse/categories/{id} endpoint in February 2026.\n\n' +
        'Alternatives:\n' +
        '1. Use Source.mineTracks({ keyword: ["' + category_id + '"], type: "playlist" })\n' +
        '2. Use Source.getRecomTracks({ seed_genres: "' + category_id + '" })\n\n' +
        'See: https://chimildic.github.io/goofy/#/reference/source'
    );
}
```

---

## Section 4: Playlist Endpoints Rename

### Background

Spotify renamed playlist endpoints:
- `/playlists/{id}/tracks` → `/playlists/{id}/items`

**Note:** This applies to DELETE and PUT operations on playlist items.

### Current State (NOT YET MIGRATED)

**File:** `library.js`

**Location 1 - `Playlist.removeTracksRequest()` (line ~2405):**

**Current Code (VERIFIED):**
```javascript
function removeTracksRequest(id, tracks) {
    if (tracks.length > 0) {
        let params = {
            url: `${API_BASE_URL}/playlists/${id}/tracks`,  // NEEDS UPDATE
            key: 'tracks',
            limit: 100,
            items: getTrackUris(tracks, 'object'),
        }
        SpotifyRequest.deleteItems(params);
    }
}
```

**After:**
```javascript
function removeTracksRequest(id, tracks) {
    if (tracks.length > 0) {
        let params = {
            url: `${API_BASE_URL}/playlists/${id}/items`,  // MIGRATION: renamed Feb 2026
            key: 'tracks',
            limit: 100,
            items: getTrackUris(tracks, 'object'),
        }
        SpotifyRequest.deleteItems(params);
    }
}
```

**Location 2 - `Playlist.modifyTracks()` (line ~2432):**

**Current Code (VERIFIED):**
```javascript
let url = `${API_BASE_URL}/playlists/${data.id}/tracks`;  // NEEDS UPDATE
```

**After:**
```javascript
let url = `${API_BASE_URL}/playlists/${data.id}/items`;  // MIGRATION: renamed Feb 2026
```

### Verification Command

```bash
grep -n "playlists.*tracks" library.js
```

Expected output should show `/items` instead of `/tracks` for the two locations above.

---

## Section 5: Playlist Field Rename

### Background

Spotify renamed response fields:
- `playlist.tracks` → `playlist.items`
- `playlist.tracks.items` → `playlist.items.items`

**Important:** Per Spotify's changes, playlist contents (`items`) are only returned for playlists the user owns or collaborates on. For other playlists, only metadata is returned and the `items` field will be absent.

### Current State (NOT YET MIGRATED)

**File:** `library.js`  
**Function:** `Source.getItemsByPlaylistObject()` (line ~707)

**Current Code (VERIFIED):**
```javascript
function getItemsByPlaylistObject(obj) {
    let items = [];
    if (obj && obj.tracks && obj.tracks.items) {  // NEEDS UPDATE
        items = obj.tracks.total <= 100 ? obj.tracks.items : SpotifyRequest.getItemsByNext(obj.tracks);
        items.forEach((item) => (item.origin = { id: obj.id, name: obj.name, type: obj.type }));
    }
    return items;
}
```

### Solution

Add backward-compatible helper function and update the main function.

**Add new helper function in Source module (before `getItemsByPlaylistObject`):**

```javascript
/**
 * Helper for Spotify Feb 2026 API change
 * Handles both old ('tracks') and new ('items') playlist field names
 * @param {Object} playlistObj - Playlist object from Spotify API
 * @returns {Object|null} - The items container or null
 */
function getPlaylistItemsContainer(playlistObj) {
    if (!playlistObj) return null;
    
    // New format (Feb 2026+)
    if (playlistObj.items && playlistObj.items.items) {
        return playlistObj.items;
    }
    
    // Old format (pre-Feb 2026) - backward compatibility
    if (playlistObj.tracks && playlistObj.tracks.items) {
        return playlistObj.tracks;
    }
    
    // Handle case where items may be absent (playlists not owned/collaborated)
    if (!playlistObj.items && !playlistObj.tracks) {
        Admin.printInfo('Playlist items not available - may require ownership/collaboration:', playlistObj.id);
        return null;
    }
    
    return null;
}
```

**Update `getItemsByPlaylistObject()`:**

```javascript
function getItemsByPlaylistObject(obj) {
    let items = [];
    let container = getPlaylistItemsContainer(obj);
    if (container) {
        items = container.total <= 100 ? container.items : SpotifyRequest.getItemsByNext(container);
        items.forEach((item) => (item.origin = { id: obj.id, name: obj.name, type: obj.type }));
    }
    return items;
}
```

---

## Section 6: Search Limit Migration

### Current State (ALREADY MIGRATED ✓)

**File:** `library.js`  
**Search module**

The actual code ALREADY has this migration:

**Line ~2865 (find function):**
```javascript
// MIGRATION: Spotify February 2026 - Search limit reduced from 50 to 10
// To get more results, increase requestCount parameter (uses pagination)
function find(keywords, type, requestCount = 1) {
    const limit = 10;  // MIGRATION: Changed from 50 (Spotify API limit February 2026)
```

**Line ~2827 (findBest function):**
```javascript
// MIGRATION: Spotify February 2026 - Search limit reduced from 20 to 10
function findBest(keywords, type) {
    let urls = keywords.map((keyword) => {
        let queryObj = { q: keyword.slice(0, 100), type: type, limit: 10 };  // Changed from 20
```

### No Action Required

This migration is already complete in the codebase.

### Usage Change Notice

To get same number of results as before:
```javascript
// Before:
let results = Search.findTracks(['keyword'], 1);  // 50 results per page

// After:
let results = Search.findTracks(['keyword'], 5);  // 5 pages × 10 = 50 results
```

---

## Section 7: Popularity Field Removal

### Background

Spotify removed `popularity` field from Track, Album, Artist objects.

### ⚠️ CRITICAL: `isSimplified()` Function

**File:** `library.js`  
**Function:** `isSimplified()` in `getCachedTracks` module (line ~3195)

**Current Code (VERIFIED - BROKEN):**
```javascript
// В объектах Track, Album, Artist Simplified нет ключа popularity
function isSimplified(item) {
    return !item.popularity;
}
```

**This is BROKEN** - With popularity removed, this always returns `true`, causing:
- Excessive API calls to fetch "full" objects
- Every track/album/artist treated as "simplified"
- Cache not working properly

**After (FIX):**
```javascript
/**
 * Determines if an object is a "simplified" version
 * MIGRATION: popularity field removed Feb 2026 - now uses type-specific fields
 * @param {Object} item - Track, Album, Artist, or AudioFeatures object
 * @returns {boolean} - true if simplified, false if full object
 */
function isSimplified(item) {
    if (!item) return true;
    
    // Audio-features objects are never "simplified" in the Spotify sense
    if (item.type === undefined && item.danceability !== undefined) {
        return false;
    }
    
    // Track: simplified tracks don't have album.id
    if (item.type === 'track') {
        return !item.album || !item.album.id;
    }
    
    // Album: simplified albums don't have total_tracks
    if (item.type === 'album') {
        return item.total_tracks === undefined;
    }
    
    // Artist: simplified artists don't have genres OR have undefined genres
    if (item.type === 'artist') {
        return !item.genres; // undefined or null means simplified
    }
    
    // Playlist item: simplified items don't have added_at
    if (item.added_at !== undefined && item.track) {
        return false; // Full playlist item
    }
    
    // Default: if no recognized type, assume full object
    return false;
}
```

### Update `Order.sort()` for Popularity Deprecation

**File:** `library.js`  
**Function:** `Order.sort()` → `sortMeta()` (line ~2078)

**Current Code:**
```javascript
function sortMeta() {
    // name, popularity, duration_ms, explicit, dates
    let hasKey = _source.every((t) => t[_key] != undefined);
    // ...
}
```

**After:**
```javascript
function sortMeta() {
    // name, popularity, duration_ms, explicit, dates
    
    // MIGRATION: popularity removed Feb 2026
    if (_key == 'popularity') {
        Admin.printInfo('WARNING: Sorting by popularity is no longer supported. Spotify removed this field in Feb 2026.');
        return;  // Skip sorting
    }
    
    let hasKey = _source.every((t) => t[_key] != undefined);
    // ... rest of existing code
}
```

### Update `sortAlbum()` (line ~2101):

**Current Code:**
```javascript
function sortAlbum() {
    // popularity, name, release_date
    let hasKey = _source.every((t) => extract(t)[_key] != undefined);
    // ...
}
```

**After:**
```javascript
function sortAlbum() {
    // popularity, name, release_date
    
    // MIGRATION: popularity removed Feb 2026
    if (_key == 'popularity') {
        Admin.printInfo('WARNING: Sorting albums by popularity is no longer supported.');
        return;
    }
    
    let hasKey = _source.every((t) => extract(t)[_key] != undefined);
    // ... rest of existing code
}
```

### Update `Source.mineTracks()` for Popularity Filter

**File:** `library.js`  
**Function:** `filterPopularity()` inside `mineTracks()` (line ~778)

**Current Code:**
```javascript
function filterPopularity(array) {
    if (!params.hasOwnProperty('popularity')) {
        return array;
    }
    if (array[0] && !array[0].hasOwnProperty('popularity')) {
        let ids = array.map((t) => t.id);
        array = SpotifyRequest.getFullObjByIds('tracks', ids, 50);
    }
    return array.filter((t) => t.popularity >= (params.popularity || 0));
}
```

**After:**
```javascript
function filterPopularity(array) {
    if (!params.hasOwnProperty('popularity')) {
        return array;
    }
    
    // MIGRATION: popularity field removed Feb 2026
    Admin.printInfo('WARNING: Filtering by popularity is no longer supported. Spotify removed this field in Feb 2026.');
    
    // Return original array - popularity filtering is no longer possible
    return array;
}
```

---

## Section 8: available_markets Field Removal

### Background

Spotify removed `available_markets` field. Use `is_playable` field instead.

### Current State (NOT YET MIGRATED)

**File:** `library.js`  
**Function:** `Filter.removeUnavailable()` (line ~1310)

**Current Code (VERIFIED):**
```javascript
function removeUnavailable(tracks, market) {
    market = market || User.country;  // BROKEN: User.country now undefined
    let availableState = [];
    let unavailableState = [];
    let unclearState = [];

    identifyState();
    defineUnavailableState();
    removeUnavailableTracks();

    function identifyState() {
        tracks.forEach((t) => {
            if (t.hasOwnProperty('available_markets') && t.available_markets.includes(market)) {  // BROKEN: field removed
                availableState.push(t.id);
            } else {
                unclearState.push(t.id);
            }
        });
    }

    function defineUnavailableState() {
        if (unclearState.length == 0) return;
        SpotifyRequest.getFullObjByIds('tracks', unclearState, 50, market).forEach((t, i) => {
            if (!t) {
                let id = unclearState[i];
                let track = tracks.find(t => t.id == id);
                Admin.printInfo(`У трека изменился id, старое значение ${id} (${getTrackKeys(track)})`);
                unavailableState.push(id);
            } else if (t.hasOwnProperty('is_playable') && t.is_playable) {
                let id = t.linked_from ? t.linked_from.id : t.id;  // BROKEN: linked_from removed
                availableState.push(id);
            } else {
                unavailableState.push(t.id);
                Admin.printInfo('Трек нельзя послушать:', t.id, '-', getTrackKeys(t)[0]);
            }
        });
    }
    // ...
}
```

### Multiple Issues:

1. `User.country` returns undefined (field removed from `/me`)
2. `available_markets` field removed
3. `linked_from` field removed

### Complete Rewrite

```javascript
function removeUnavailable(tracks, market) {
    // MIGRATION: available_markets removed Feb 2026, use is_playable field
    // MIGRATION: linked_from removed Feb 2026
    // MIGRATION: User.country removed Feb 2026, use USER_MARKET property
    
    if (!tracks || tracks.length === 0) return;
    
    market = market || UserProperties.getProperty('USER_MARKET') || 'from_token';
    let unavailableState = [];
    let trackIds = tracks.map(t => t.id);
    
    // Fetch tracks with market parameter to get is_playable field
    let fullTracks = SpotifyRequest.getFullObjByIds('tracks', trackIds, 50, market);
    
    let trackMap = {};
    fullTracks.forEach(t => {
        if (t && t.id) trackMap[t.id] = t;
    });
    
    tracks.forEach((track) => {
        let fullTrack = trackMap[track.id];
        if (!fullTrack) {
            Admin.printInfo('Track not found in Spotify:', track.id);
            unavailableState.push(track.id);
        } else if (fullTrack.is_playable === true) {
            // Track is available - no action needed
        } else if (fullTrack.is_playable === false) {
            unavailableState.push(track.id);
            Admin.printInfo('Track not playable:', track.id, '-', getTrackKeys(track)[0]);
        } else {
            // is_playable is undefined - may be in Extended Quota Mode or market not specified
            // Conservative approach: assume playable
            Admin.printInfo('Could not determine playability for track:', track.id, '- assuming playable');
        }
    });

    if (unavailableState.length > 0) {
        let availableTracks = tracks.filter((t) => !unavailableState.includes(t.id));
        Combiner.replace(tracks, availableTracks);
    }
}
```

### Cache Compression Update

**File:** `library.js`  
**Function:** `Cache.compressTracks()` (line ~3686)

**Current Code:**
```javascript
delete item.available_markets;  // Harmless but unnecessary
```

**After:**
```javascript
// available_markets field removed from API - no need to delete
// delete item.available_markets;  // Removed: field no longer exists
```

**Same in `compressAlbum()`:**
```javascript
// available_markets field removed from API
// delete item.available_markets;  // Removed: field no longer exists
```

---

## Section 9: followers Field Removal

### Background

Spotify removed `followers` field from Artist and Playlist objects.

### Location 1: `Source.getArtists()` (line ~609)

**Current Code:**
```javascript
artists = artists.filter((artist) => {
    artist.followers = artist.followers.total || artist.followers;  // BROKEN: followers removed
    return (
        RangeTracks.isBelong(artist, paramsArtist) &&
        RangeTracks.isBelongGenres(artist.genres, paramsArtist.genres) &&
        !RangeTracks.isBelongBanGenres(artist.genres, paramsArtist.ban_genres)
    );
});
```

**After:**
```javascript
artists = artists.filter((artist) => {
    // MIGRATION: followers removed Feb 2026 - handle both old and new format
    if (artist.followers && typeof artist.followers === 'object') {
        artist.followers = artist.followers.total || 0;
    } else if (artist.followers === undefined) {
        artist.followers = 0;  // Default value for comparison
    }
    return (
        RangeTracks.isBelong(artist, paramsArtist) &&
        RangeTracks.isBelongGenres(artist.genres, paramsArtist.genres) &&
        !RangeTracks.isBelongBanGenres(artist.genres, paramsArtist.ban_genres)
    );
});
```

### Location 2: `Source.mineTracks()` - filterByFollowers (line ~787)

**Current Code:**
```javascript
function filterByFollowers() {
    for (let i = 0; i < result.length; i++) {
        result[i] = getFullPlaylistObject(result[i]).filter((p) => {
            return isBelongRangeFollowers(p.followers.total);  // BROKEN: followers removed
        });
    }
}
```

**After:**
```javascript
function filterByFollowers() {
    // MIGRATION: followers removed Feb 2026 - this filter no longer works
    Admin.printInfo('WARNING: Filtering by playlist followers is no longer supported. Spotify removed this field in Feb 2026.');
    // Function is now a no-op - playlists cannot be filtered by followers
}
```

### Location 3: `RangeTracks.isBelongArtist()` (line ~1289)

**Current Code:**
```javascript
if (trackArtist.followers && typeof trackArtist.followers === 'object') {
    trackArtist.followers = trackArtist.followers.total;
}
```

**After:**
```javascript
// MIGRATION: followers may not exist in Feb 2026 API
if (trackArtist.followers && typeof trackArtist.followers === 'object') {
    trackArtist.followers = trackArtist.followers.total || 0;
} else if (!trackArtist.hasOwnProperty('followers')) {
    trackArtist.followers = 0;  // Default for comparison
}
```

### Location 4: `Order.sort()` - sortArtist() (line ~1975)

**Current Code:**
```javascript
if (_key == 'followers') {
    for (let id in items) {
        if (items[id].followers && typeof items[id].followers == 'object') {
            items[id].followers = items[id].followers.total;
        }
    }
}
```

**After:**
```javascript
// MIGRATION: followers removed Feb 2026
if (_key == 'followers') {
    Admin.printInfo('WARNING: Sorting by followers is no longer supported. Spotify removed this field in Feb 2026.');
    return;  // Skip sorting
}
```

### Location 5: `Order.separateArtists()` - getArtistId() (line ~2105)

**Current Code:**
```javascript
function getArtistId(item) {
    return item.followers ? item.id : item.artists[0].id;  // BROKEN: followers removed
}
```

**After:**
```javascript
function getArtistId(item) {
    // MIGRATION: followers removed Feb 2026 - use type field instead
    if (item.type === 'artist') {
        return item.id;
    } else if (item.artists && item.artists.length > 0) {
        return item.artists[0].id;
    }
    return item.id;  // Fallback
}
```

### Location 6: `Cache.compressArtists()` (line ~3772)

**Current Code:**
```javascript
if (item.followers && item.followers.total) {
    item.followers = item.followers.total;
}
```

**After:**
```javascript
// MIGRATION: followers removed Feb 2026 - handle both old and new format
if (item.followers) {
    if (typeof item.followers === 'object' && item.followers.total) {
        item.followers = item.followers.total;
    }
    // If it's already a number, keep it
} else {
    delete item.followers;  // Remove undefined/null followers
}
```

---

## Section 10: linked_from Field Removal

### Background

Spotify removed `linked_from` field which was used for track relinking between markets.

### Impact

The `linked_from` field was used to get the original track ID when a track was "relinked" for a different market. Since this field is removed:

1. Simply use `t.id` directly
2. The `is_playable` field still indicates availability

### Location

The fix for `Filter.removeUnavailable()` in Section 8 already handles this by removing the `linked_from` reference.

### No Additional Changes Needed

---

## Section 11: User Country Field Removal

### Background

Spotify removed `country` field from `GET /me` response.

### Current State (NOT YET MIGRATED)

**File:** `library.js`  
**Function:** `User` module (line ~3520)

**Current Code (VERIFIED):**
```javascript
const User = (function () {
    !KeyValue['userId'] && setProfile();
    return {
        get id() { return KeyValue['userId'] },
        get country() { return getUser().country },  // BROKEN: country removed
        getUser,
    };

    function setProfile() {
        let user = getUser();
        if (user) {
            KeyValue['userId'] = user.id;
            UserProperties.setProperty('userId', KeyValue['userId']);
        }
    }

    function getUser() {
        try {
            return SpotifyRequest.get(API_BASE_URL + '/me');
        } catch (error) {
            return undefined;
        }
    }
})();
```

### Complete Rewrite

```javascript
const User = (function () {
    !KeyValue['userId'] && setProfile();
    
    // MIGRATION: country removed from /me response Feb 2026
    // Now uses USER_MARKET property with fallback
    const DEFAULT_MARKET = 'US';
    
    return {
        get id() { return KeyValue['userId'] },
        get country() { 
            return UserProperties.getProperty('USER_MARKET') || KeyValue.USER_MARKET || DEFAULT_MARKET;
        },
        getUser,
        getMarket: () => UserProperties.getProperty('USER_MARKET') || KeyValue.USER_MARKET || DEFAULT_MARKET,
    };

    function setProfile() {
        let user = getUser();
        if (user) {
            KeyValue['userId'] = user.id;
            UserProperties.setProperty('userId', KeyValue['userId']);
            
            // MIGRATION: country no longer available from /me
            // User must set USER_MARKET in config
            if (!UserProperties.getProperty('USER_MARKET') && !KeyValue.USER_MARKET) {
                Admin.printInfo('WARNING: USER_MARKET not set. Set it in config.js for proper market handling.');
            }
        }
    }

    function getUser() {
        try {
            return SpotifyRequest.get(API_BASE_URL + '/me');
        } catch (error) {
            return undefined;
        }
    }
})();
```

### config.js Update

**File:** `config.js`

Add the required `USER_MARKET` property:

```javascript
function setProperties() {
    // Spotify credentials
    UserProperties.setProperty('CLIENT_ID', 'вашеЗначение');
    UserProperties.setProperty('CLIENT_SECRET', 'вашеЗначение');
    UserProperties.setProperty('PRIVATE_CLIENT_ID', 'вашеЗначение');
    UserProperties.setProperty('PRIVATE_CLIENT_SECRET', 'вашеЗначение');

    // REQUIRED: User market (Spotify removed country from /me in Feb 2026)
    // Set to your Spotify market code (e.g., 'US', 'GB', 'DE', 'RU')
    UserProperties.setProperty('USER_MARKET', 'US');

    // Last.fm
    UserProperties.setProperty('LASTFM_API_KEY', 'вашеЗначение');
    UserProperties.setProperty('LASTFM_LOGIN', 'вашЛогин');
    UserProperties.setProperty('LASTFM_RANGE_RECENT_TRACKS', '30');

    // Other settings
    UserProperties.setProperty('ON_SPOTIFY_RECENT_TRACKS', 'true');
    UserProperties.setProperty('ON_LASTFM_RECENT_TRACKS', 'false');
    UserProperties.setProperty('COUNT_RECENT_TRACKS', '60000');

    UserProperties.setProperty('LOG_LEVEL', 'info');
    UserProperties.setProperty('LOCALE', 'RU');
    UserProperties.setProperty('REQUESTS_IN_ROW', '20');
    UserProperties.setProperty('MIN_DICE_RATING', '0.6005');
}

// ============================================================
// SPOTIFY API FEBRUARY 2026 MIGRATION NOTES
// ============================================================
//
// 1. USER_MARKET is now REQUIRED - Spotify removed country from /me
//    Set this to your market code (e.g., 'US', 'GB', 'DE', 'RU')
//
// 2. Extended Quota Mode apps have different behavior:
//    - Recommendations endpoint: Available in Extended Quota, may fail in Dev Mode
//    - Audio features: Available in Extended Quota, may fail in Dev Mode
//    - Artist top-tracks: Removed for ALL modes (use Last.fm fallback)
//    - Browse/categories: Removed for ALL modes
//
// 3. Field removals (affect ALL modes):
//    - popularity: Removed from Track, Album, Artist
//    - followers: Removed from Artist, Playlist
//    - available_markets: Removed from Track
//    - linked_from: Removed from Track
//    - country: Removed from User
```

---

## Section 12: Audio Features Migration

### Background

The batch endpoint `GET /audio-features?ids=...` was removed along with other batch endpoints. Additionally, in Development Mode, audio features may be completely unavailable.

### Current State

The `getFullObjByIds()` function already handles individual requests, but there's no graceful degradation when audio features are unavailable.

### Add Graceful Degradation

**File:** `library.js`  
**Function:** `getCachedTracks()` → `cacheToFullObj()` (line ~3172)

**Current Code:**
```javascript
if (uncachedTracks.features.length > 0) {
    // limit = 100, но UrlFetchApp.fetch выдает ошибку о превышении длины URL
    // При limit 85, длина URL для этого запроса 2001 символ
    let features = SpotifyRequest.getFullObjByIds('audio-features', uncachedTracks.features, 85);
    features.forEach((item, i) => {
        if (item == null) {
            let id = uncachedTracks.features[i];
            cachedTracks.features[id] = {};
            Admin.printInfo(`У трека нет features, id: ${id}`);
        } else {
            item.anger = item.energy * (1 - item.valence);
            item.happiness = item.energy * item.valence;
            item.sadness = (1 - item.energy) * (1 - item.valence);
            cachedTracks.features[item.id] = item;
        }
    });
}
```

**After:**
```javascript
if (uncachedTracks.features.length > 0) {
    // MIGRATION: Spotify removed batch audio-features endpoint Feb 2026
    // Now uses individual endpoints via getFullObjByIds
    // Note: Audio features may be unavailable in Development Mode
    
    let features;
    try {
        features = SpotifyRequest.getFullObjByIds('audio-features', uncachedTracks.features, 85);
    } catch (e) {
        Admin.printInfo('Audio features unavailable - may be in Development Mode:', e.message || e);
        // Set empty features for all requested tracks
        uncachedTracks.features.forEach(id => {
            cachedTracks.features[id] = {};
        });
        return;
    }
    
    features.forEach((item, i) => {
        if (item == null) {
            let id = uncachedTracks.features[i];
            cachedTracks.features[id] = {};
            Admin.printInfo(`У трека нет features, id: ${id}`);
        } else {
            item.anger = item.energy * (1 - item.valence);
            item.happiness = item.energy * item.valence;
            item.sadness = (1 - item.energy) * (1 - item.valence);
            cachedTracks.features[item.id] = item;
        }
    });
}
```

---

## Section 13: Recommendations Graceful Degradation

### Background

The `/recommendations` endpoint may be unavailable in Development Mode. Several functions rely on this endpoint.

### Affected Functions

1. `Source.getRecomTracks()` - Primary recommendations function
2. `Source.craftTracks()` - Uses recommendations for track crafting
3. `Source.getRecomArtists()` - Uses recommendations for artist discovery
4. `Filter.replaceWithSimilar()` - Uses recommendations for similar tracks

### Solution

Add graceful degradation with warning messages.

**File:** `library.js`  
**Function:** `Source.getRecomTracks()` (line ~574)

**Current Code:**
```javascript
function getRecomTracks(queryObj) {
    let url = createUrlForRecomTracks(queryObj);
    return SpotifyRequest.get(url).tracks;
}
```

**After:**
```javascript
function getRecomTracks(queryObj) {
    let url = createUrlForRecomTracks(queryObj);
    try {
        let response = SpotifyRequest.get(url);
        return response ? response.tracks : [];
    } catch (e) {
        Admin.printInfo('Recommendations unavailable - may require Extended Quota Mode:', e.message || e);
        return [];
    }
}
```

**Function:** `Source.craftTracks()` (line ~802)

Add try-catch around the recommendations call:

```javascript
function craftTracks(tracks, params = {}) {
    let recomTracks;
    try {
        recomTracks = SpotifyRequest.getAll(createUrls()).reduce((recomTracks, response) => {
            return Combiner.push(recomTracks, response.tracks || []);
        }, []);
    } catch (e) {
        Admin.printInfo('craftTracks: Recommendations unavailable - returning empty result:', e.message || e);
        return [];
    }
    Filter.dedupTracks(recomTracks);
    return recomTracks;
    // ... rest of function
}
```

---

## Section 14: Addons Migration

### Status: Review Required

All addons use `library.js` abstractions and will automatically benefit from core migrations. However, review each addon for direct API usage.

| Addon | Status | Notes |
|-------|--------|-------|
| history-manager.js | Auto-migrated | Uses `getFullObjByIds()` |
| release.js | Auto-migrated | Uses `getFullObjByIds()` |
| radio.js | Auto-migrated | Uses `Search.multisearchTracks()` |
| helper.js | Not affected | No Spotify API calls |

### Verification Steps

1. Search each addon file for:
   - Direct API calls to Spotify
   - Usage of removed fields (popularity, followers, available_markets)
   - Usage of removed endpoints (browse/categories, artists/top-tracks)

2. Test each addon after migration

---

## Section 15: Authentication Migration

### Background

Spotify Development Mode now limits to 1 Client ID per developer. Goofy previously required 2 for Extended Quota Mode.

### Current State

The Auth module handles private credentials but doesn't detect credential mode or Extended Quota availability.

### Add Mode Detection (Optional Enhancement)

**File:** `library.js`  
**Function:** `Auth` module

Add these utility methods to the Auth module:

```javascript
const Auth = (function () {
    // ... existing code ...
    
    // Detect credential mode
    const hasPrivateCredentials = !!(KeyValue.PRIVATE_CLIENT_ID && KeyValue.PRIVATE_CLIENT_SECRET);
    const isSingleCredentialMode = !hasPrivateCredentials || 
        (KeyValue.CLIENT_ID === KeyValue.PRIVATE_CLIENT_ID);
    
    // ... existing return statement, add:
    return {
        reset, hasAccess, getPublicAccessToken, getPrivateAccessToken, displayAuthPage, displayAuthResult,
        // New methods for Feb 2026 migration
        isSingleCredentialMode: () => isSingleCredentialMode,
        hasPrivateCredentials: () => hasPrivateCredentials,
    };
    
    // ... rest of existing code ...
})();
```

---

## Implementation Checklist

### Phase 1: Critical Changes (Do First) ⚠️

- [x] ~~Update `getFullObjByIds()`~~ - Already migrated in code
- [ ] **Fix `isSimplified()` function** - CRITICAL: Prevents excessive API calls
- [ ] **Update `User.country` getter** - Now requires `USER_MARKET` property
- [ ] **Add `USER_MARKET` to config.js** - Required for market-specific operations
- [ ] **Update `Filter.removeUnavailable()`** - Uses removed fields

### Phase 2: High Priority Changes

- [ ] **Update playlist endpoint URLs** - `/tracks` → `/items` (2 locations)
- [ ] **Add `getPlaylistItemsContainer()` helper** - Backward-compatible field access
- [ ] **Update `getItemsByPlaylistObject()`** - Use new helper function
- [x] ~~Update search limits~~ - Already migrated in code

### Phase 3: Medium Priority Changes

- [ ] **Implement `getArtistsTopTracks()` migration** - Last.fm + Recommendations fallback
- [ ] **Add deprecation stubs for browse categories** - `getListCategory()`, `getCategoryTracks()`
- [ ] **Remove `followers` field handling** - 6+ locations
- [ ] **Add popularity deprecation warnings** - In sorting and filtering

### Phase 4: Graceful Degradation

- [ ] **Add audio features error handling** - Graceful degradation for Dev Mode
- [ ] **Add recommendations error handling** - Graceful degradation for Dev Mode
- [ ] **Add Auth mode detection** - Optional utility methods

### Phase 5: Documentation

- [ ] **Update `docs/install.md`** - Add `USER_MARKET` requirement
- [ ] **Update `docs/reference/source.md`** - Document deprecations
- [ ] **Update `docs/changelog.md`** - Breaking changes
- [ ] **Update `config.js` comments** - Migration notes

### Phase 6: Testing

- [ ] Test batch endpoint replacement with various object types
- [ ] Test `isSimplified()` with all object types
- [ ] Test playlist operations with field rename
- [ ] Test availability filtering with `is_playable`
- [ ] Test search with pagination (already migrated)
- [ ] Test artist top tracks with Last.fm and fallback
- [ ] Test `User.country` getter with USER_MARKET
- [ ] Test all followers-dependent code paths

---

## Testing Guide

### Test 1: isSimplified Function (CRITICAL)

```javascript
function testIsSimplified() {
    // Simplified track (no album.id)
    let simplifiedTrack = { type: 'track', id: '123', name: 'Test', artists: [] };
    console.log('Simplified track:', isSimplified(simplifiedTrack), 'Expected: true');
    
    // Full track
    let fullTrack = { type: 'track', id: '123', album: { id: '456' } };
    console.log('Full track:', isSimplified(fullTrack), 'Expected: false');
    
    // Simplified artist
    let simplifiedArtist = { type: 'artist', id: '123', name: 'Test' };
    console.log('Simplified artist:', isSimplified(simplifiedArtist), 'Expected: true');
    
    // Full artist
    let fullArtist = { type: 'artist', id: '123', genres: ['pop'] };
    console.log('Full artist:', isSimplified(fullArtist), 'Expected: false');
    
    // Simplified album
    let simplifiedAlbum = { type: 'album', id: '123', name: 'Test' };
    console.log('Simplified album:', isSimplified(simplifiedAlbum), 'Expected: true');
    
    // Full album
    let fullAlbum = { type: 'album', id: '123', total_tracks: 10 };
    console.log('Full album:', isSimplified(fullAlbum), 'Expected: false');
    
    // Audio features (never simplified)
    let features = { danceability: 0.5, energy: 0.8 };
    console.log('Audio features:', isSimplified(features), 'Expected: false');
}
```

### Test 2: User Country

```javascript
function testUserCountry() {
    // Before setting USER_MARKET
    console.log('User.country (no USER_MARKET):', User.country, 'Expected: "US" (default)');
    
    // Set USER_MARKET
    UserProperties.setProperty('USER_MARKET', 'GB');
    console.log('User.country (GB):', User.country, 'Expected: "GB"');
    
    // Test getMarket
    console.log('User.getMarket():', User.getMarket(), 'Expected: "GB"');
}
```

### Test 3: Playlist Operations

```javascript
function testPlaylistOperations() {
    // Test with a known playlist ID
    let playlistId = '37i9dQZEVXbMDoHDwVN2tF';  // Spotify Today's Top Hits
    let playlist = Playlist.getById(playlistId);
    
    // Test field rename handling
    let items = Source.getItemsByPlaylistObject(playlist);
    console.log('Playlist items:', items.length);
    
    // Verify helper works with both old and new formats
    let container = getPlaylistItemsContainer(playlist);
    console.log('Container found:', !!container);
}
```

### Test 4: Availability Filter

```javascript
function testAvailabilityFilter() {
    // Get some tracks
    let tracks = Source.getTracks([{ id: '37i9dQZEVXbMDoHDwVN2tF' }]);
    let testTracks = tracks.slice(0, 10);
    
    console.log('Before filter:', testTracks.length);
    
    // Set market
    UserProperties.setProperty('USER_MARKET', 'US');
    
    // Filter unavailable
    Filter.removeUnavailable(testTracks);
    console.log('After filter:', testTracks.length);
}
```

### Test 5: Artist Top Tracks

```javascript
function testArtistTopTracks() {
    let artists = [{ id: '4q3ewBCX7sLwd24euuV69X', name: 'Coldplay' }];
    
    // Test the migration
    let tracks = Source.getArtistsTopTracks(artists);
    console.log('Top tracks found:', tracks.length);
    
    // Verify tracks are actually by the artist
    let matchingTracks = tracks.filter(t => 
        t.artists.some(a => a.name.toLowerCase() === 'coldplay')
    );
    console.log('Matching tracks:', matchingTracks.length, 'of', tracks.length);
}
```

### Test 6: Batch Endpoints

```javascript
function testBatchEndpoints() {
    // Test tracks
    let trackIds = ['4iV5W9uYEdYUVa79Axb7Rh', '1301WleyT98MSxVHPZCA6M'];
    let tracks = SpotifyRequest.getFullObjByIds('tracks', trackIds, 50);
    console.log('Tracks fetched:', tracks.length, 'Expected: 2');
    
    // Test artists
    let artistIds = ['4q3ewBCX7sLwd24euuV69X', '4q3ewBCX7sLwd24euuV69Y'];
    let artists = SpotifyRequest.getFullObjByIds('artists', artistIds, 50);
    console.log('Artists fetched:', artists.length, 'Expected: 2');
}
```

---

## Rollback Plan

If migration causes critical issues, rollback in this order:

### Priority 1: Most Impactful Changes

1. **Revert `isSimplified()`** - Most impactful, affects caching
2. **Revert `User.country` changes** - May break availability filtering
3. **Revert `Filter.removeUnavailable()` changes** - May break availability detection

### Priority 2: Feature Changes

4. **Revert `getArtistsTopTracks()` changes** - Use old endpoint (Extended Quota only)
5. **Revert `getItemsByPlaylistObject()` changes** - May break playlist reading

### Backup Strategy

```bash
# Before migration, create backup
cp library.js library.js.backup
cp config.js config.js.backup

# To rollback
cp library.js.backup library.js
cp config.js.backup config.js
```

### Partial Rollback

If only specific features cause issues, you can selectively revert:

1. **If availability filtering fails:** Add `SKIP_AVAILABILITY_FILTER` property and check it in `removeUnavailable()`
2. **If artist top tracks fails:** The code already has fallback mechanism
3. **If playlist reading fails:** Check if `getPlaylistItemsContainer()` is working correctly

---

## Support

For questions or issues:
- GitHub Issues: https://github.com/Chimildic/goofy/issues
- Documentation: https://chimildic.github.io/goofy/
- Telegram: https://t.me/forum_goofy

---

## Appendix: Complete Code Location Reference

Use these search patterns to find exact code locations:

| Change | File | Line (approx) | Search Pattern |
|--------|------|---------------|----------------|
| isSimplified | library.js | ~3195 | `function isSimplified` |
| User.country | library.js | ~3520 | `get country()` |
| getFullObjByIds | library.js | ~3440 | `function getFullObjByIds` |
| Playlist URLs | library.js | ~2405, ~2432 | `playlists.*tracks` |
| Playlist fields | library.js | ~707 | `obj\.tracks` |
| removeUnavailable | library.js | ~1310 | `available_markets` |
| followers (getArtists) | library.js | ~609 | `artist.followers` |
| followers (mineTracks) | library.js | ~787 | `filterByFollowers` |
| followers (isBelongArtist) | library.js | ~1289 | `trackArtist.followers` |
| followers (sortArtist) | library.js | ~1975 | `_key == 'followers'` |
| followers (separateArtists) | library.js | ~2105 | `getArtistId` |
| followers (compressArtists) | library.js | ~3772 | `item.followers.total` |
| getArtistsTopTracks | library.js | ~530 | `getArtistsTopTracks` |
| browse categories | library.js | ~518 | `browse/categories` |
| search limit | library.js | ~2865, ~2827 | `limit.*10` |
| popularity (sortMeta) | library.js | ~2078 | `sortMeta` |
| popularity (sortAlbum) | library.js | ~2101 | `sortAlbum` |
| popularity (filterPopularity) | library.js | ~778 | `filterPopularity` |
| audio features | library.js | ~3172 | `uncachedTracks.features` |

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 3.0 | March 12, 2026 | Second-pass review: Added missing sections, fixed code issues, aligned with actual codebase |
| 2.0 | March 12, 2026 | Initial review updates |
| 1.0 | March 12, 2026 | Initial draft |

---

*End of Migration Plan v3.0*
