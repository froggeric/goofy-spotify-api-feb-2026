# Spotify Web API Changes - February 2026

## Overview

On February 6, 2026, Spotify announced significant changes to their developer platform. These changes affect Development Mode apps and introduce new API endpoint structures.

**Official Documentation:**
- Changelog: https://developer.spotify.com/documentation/web-api/references/changes/february-2026
- Migration Guide: https://developer.spotify.com/documentation/web-api/tutorials/february-2026-migration-guide
- Blog Post: https://developer.spotify.com/blog/2026-02-06-update-on-developer-access-and-platform-security

---

## Development Mode Restrictions

### Timeline

| Date | Event |
|------|-------|
| February 11, 2026 | New Development Mode apps created with new restrictions |
| March 9, 2026 | Existing Development Mode apps migrated to new restrictions |

### New Restrictions

| Requirement | Limit |
|-------------|-------|
| Premium Account | Required for app owner |
| Client IDs per developer | 1 |
| Users per app | 5 |

**Important:**
- If the owner's Premium subscription lapses, the app will stop working
- Existing apps are grandfathered (retain existing Client IDs and users)
- **Extended Quota Mode apps are NOT affected** - all endpoints, fields, and behaviors remain unchanged

---

## Removed Endpoints

### Batch/Bulk Fetch Endpoints (No Replacement)

| Removed Endpoint | Description |
|------------------|-------------|
| `GET /tracks` | Get multiple tracks |
| `GET /albums` | Get multiple albums |
| `GET /artists` | Get multiple artists |
| `GET /episodes` | Get multiple episodes |
| `GET /shows` | Get multiple shows |
| `GET /audiobooks` | Get multiple audiobooks |
| `GET /chapters` | Get multiple chapters |

**Migration:** Use single-item endpoints (e.g., `GET /tracks/{id}`) with individual requests.

### Browse and Artist Endpoints (No Replacement)

| Removed Endpoint | Description |
|------------------|-------------|
| `GET /browse/new-releases` | New album releases |
| `GET /browse/categories` | Browse categories list |
| `GET /browse/categories/{id}` | Single category details |
| `GET /artists/{id}/top-tracks` | Artist's top tracks by country |
| `GET /markets` | Available markets list |

### User Data Endpoints

| Removed Endpoint | Replacement |
|------------------|-------------|
| `GET /users/{id}` | Use `GET /me` for current user |
| `GET /users/{id}/playlists` | Use `GET /me/playlists` for current user |
| `POST /users/{user_id}/playlists` | Use `POST /me/playlists` |

---

## New Endpoints

### Unified Library Management

| New Endpoint | Description |
|--------------|-------------|
| `PUT /me/library` | Save Spotify URIs to user's library (unified save/follow) |
| `DELETE /me/library` | Remove Spotify URIs from user's library (unified remove/unfollow) |
| `GET /me/library/contains` | Check if items are saved in user's library |

### Playlist Items

| New Endpoint | Description |
|--------------|-------------|
| `POST /playlists/{id}/items` | Add items to playlist |
| `GET /playlists/{id}/items` | Get playlist items |
| `DELETE /playlists/{id}/items` | Remove playlist items |
| `PUT /playlists/{id}/items` | Update/reorder playlist items |

---

## Replaced/Renamed Endpoints

### Library Management (Old → New)

```
PUT    /me/tracks                    → PUT    /me/library
PUT    /me/albums                    → PUT    /me/library
PUT    /me/episodes                  → PUT    /me/library
PUT    /me/shows                     → PUT    /me/library
PUT    /me/audiobooks                → PUT    /me/library
PUT    /me/following                 → PUT    /me/library
PUT    /playlists/{id}/followers     → PUT    /me/library

DELETE /me/tracks                    → DELETE /me/library
DELETE /me/albums                    → DELETE /me/library
DELETE /me/episodes                  → DELETE /me/library
DELETE /me/shows                     → DELETE /me/library
DELETE /me/audiobooks                → DELETE /me/library
DELETE /me/following                 → DELETE /me/library
DELETE /playlists/{id}/followers     → DELETE /me/library

GET    /me/tracks/contains           → GET    /me/library/contains
GET    /me/albums/contains           → GET    /me/library/contains
GET    /me/episodes/contains         → GET    /me/library/contains
GET    /me/shows/contains            → GET    /me/library/contains
GET    /me/audiobooks/contains       → GET    /me/library/contains
GET    /me/following/contains        → GET    /me/library/contains
GET    /playlists/{id}/followers/contains → GET /me/library/contains
```

### Playlist Endpoints (Renamed)

| Old Endpoint | New Endpoint |
|--------------|--------------|
| `POST /playlists/{id}/tracks` | `POST /playlists/{id}/items` |
| `GET /playlists/{id}/tracks` | `GET /playlists/{id}/items` |
| `DELETE /playlists/{id}/tracks` | `DELETE /playlists/{id}/items` |
| `PUT /playlists/{playlist_id}/tracks` | `PUT /playlists/{id}/items` |

---

## Modified Endpoints

### Search Endpoint

| Parameter | Before | After |
|-----------|--------|-------|
| `limit` maximum | 50 | 10 |
| `limit` default | 20 | 5 |

**Migration:** Use pagination with `offset` parameter if you need more than 10 results.

---

## Removed Fields

### Track Object

| Field | Status |
|-------|--------|
| `available_markets` | Removed |
| `external_ids` | Removed *(reverted in March 2026)* |
| `linked_from` | Removed |
| `popularity` | Removed |

### Album Object

| Field | Status |
|-------|--------|
| `album_group` | Removed |
| `available_markets` | Removed |
| `external_ids` | Removed *(reverted in March 2026)* |
| `label` | Removed |
| `popularity` | Removed |

### Artist Object

| Field | Status |
|-------|--------|
| `followers` | Removed |
| `popularity` | Removed |

### User Object (from `GET /me`)

| Field | Status |
|-------|--------|
| `country` | Removed |
| `email` | Removed |
| `explicit_content` | Removed |
| `followers` | Removed |
| `product` | Removed |

### Show Object

| Field | Status |
|-------|--------|
| `available_markets` | Removed |
| `publisher` | Removed |

### Audiobook/Chapter Object

| Field | Status |
|-------|--------|
| `available_markets` | Removed |
| `publisher` | Removed (audiobook only) |

---

## Renamed Fields

### Playlist Object

| Old Field | New Field |
|-----------|-----------|
| `tracks` | `items` |
| `tracks.tracks` | `items.items` |
| `tracks.tracks.track` | `items.items.item` |

**Critical:** Playlist contents (`items`) are only returned for playlists the user owns or collaborates on. For other playlists, only metadata is returned and the `items` field will be absent.

---

## Still Available Endpoints

### Library

- `PUT /playlists/{id}` - Change playlist details
- `POST /me/playlists` - Create playlist
- `GET /me/playlists` - Get current user's playlists
- `GET /me/following` - Get followed artists
- `GET /me/albums` - Get saved albums
- `GET /me/audiobooks` - Get saved audiobooks
- `GET /me/episodes` - Get saved episodes
- `GET /me/shows` - Get saved shows
- `GET /me/tracks` - Get saved tracks

### Metadata (Single Item Fetch)

- `GET /albums/{id}`, `GET /albums/{id}/tracks`
- `GET /artists/{id}`, `GET /artists/{id}/albums`
- `GET /audiobooks/{id}`, `GET /audiobooks/{id}/chapters`
- `GET /tracks/{id}`
- `GET /episodes/{id}`
- `GET /shows/{id}`, `GET /shows/{id}/episodes`
- `GET /chapters/{id}`
- `GET /search`

### User

- `GET /me` - Get current user's profile

### Personalization

- `GET /me/top/{type}` - Get user's top artists/tracks

### Player (All Preserved)

- `POST /me/player/queue` - Add to queue
- `GET /me/player/devices` - Get available devices
- `GET /me/player/currently-playing` - Get currently playing
- `GET /me/player` - Get playback state
- `GET /me/player/recently-played` - Get recently played
- `GET /me/player/queue` - Get queue
- `PUT /me/player/pause` - Pause playback
- `PUT /me/player/seek` - Seek to position
- `PUT /me/player/repeat` - Set repeat mode
- `PUT /me/player/volume` - Set volume
- `POST /me/player/next` - Skip to next
- `POST /me/player/previous` - Skip to previous
- `PUT /me/player/play` - Start/resume playback
- `PUT /me/player/shuffle` - Toggle shuffle
- `PUT /me/player` - Transfer playback

### Playlist

- `GET /playlists/{id}` - Get playlist details
- `GET /playlists/{id}/images` - Get playlist cover image
- `PUT /playlists/{id}/images` - Upload custom cover image

---

## Migration Examples

### Saving Items to Library

```javascript
// Before
await spotify.put('/me/tracks', { ids: ['trackId1', 'trackId2'] });
await spotify.put('/me/albums', { ids: ['albumId1'] });
await spotify.put('/me/following', { ids: ['artistId1'], type: 'artist' });

// After - all entity types in a single call
await spotify.put('/me/library', {
  uris: [
    'spotify:track:trackId1',
    'spotify:track:trackId2',
    'spotify:album:albumId1',
    'spotify:artist:artistId1'
  ]
});
```

### Batch Fetch Migration

```javascript
// Before: Batch fetch
const response = await fetch('/v1/tracks?ids=id1,id2,id3');
const { tracks } = await response.json();

// After: Individual fetches
const trackIds = ['id1', 'id2', 'id3'];
const tracks = await Promise.all(
  trackIds.map(id =>
    fetch(`/v1/tracks/${id}`).then(r => r.json())
  )
);
```

### Handling Missing Popularity Field

```javascript
// Before
const sortedTracks = tracks.sort((a, b) => b.popularity - a.popularity);

// After - popularity is no longer available
// Use alternative sorting criteria
const sortedTracks = tracks.sort((a, b) =>
  a.name.localeCompare(b.name)
);
```

### Handling Playlist Field Rename

```javascript
// Before
const trackCount = playlist.tracks.total;
const firstTrack = playlist.tracks.items[0].track;

// After
const trackCount = playlist.items?.total ?? 0;
const firstTrack = playlist.items?.items?.[0]?.item;

// Note: items may be undefined for playlists you don't own
if (!playlist.items) {
  console.log('Track details not available for this playlist');
}
```

---

## Migration Checklist

- [ ] **Account:** Ensure app owner has Spotify Premium
- [ ] **Library endpoints:** Replace content-specific save/remove/follow/unfollow calls with `PUT/DELETE /me/library`
- [ ] **Contains checks:** Replace content-specific "contains" calls with `GET /me/library/contains`
- [ ] **Batch fetches:** Replace batch endpoints with individual fetch calls
- [ ] **Browse and artist endpoints:** Remove features using browse categories, new releases, or artist top tracks
- [ ] **Other users:** Remove features that fetch other users' profiles or playlists
- [ ] **Playlist endpoints:** Update `/playlists/{id}/tracks` calls to `/playlists/{id}/items`
- [ ] **Playlist handling:** Update code for `tracks` → `items` field rename
- [ ] **Removed fields:** Handle missing fields gracefully (check for undefined/null)
- [ ] **Search pagination:** Update search requests for reduced `limit` (max 10), paginate if needed
- [ ] **Rate limiting:** Limit API calls to avoid hitting rate limits

---

## Impact on Goofy Project

This project uses several affected endpoints. Key areas to review:

1. **`Source.getTopTracks()`** - Uses `GET /artists/{id}/top-tracks` (REMOVED)
2. **`Source.getCategoryTracks()`** - Uses browse categories (REMOVED)
3. **`Source.getArtistsById()`** - Uses `GET /artists` batch endpoint (REMOVED)
4. **Playlist item access** - Field `tracks` renamed to `items`
5. **`popularity` field** - Used in `RangeTracks.isBelong()` filters

See `library.js` for implementation details and required updates.
