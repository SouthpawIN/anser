---
name: media-interfaces
description: "Media interface integrations: GIF search via Tenor API and Spotify playback control."
author: Hermes Agent
version: 1.0.0
license: MIT
tags: ["Umbrella", "Media", "Interfaces", "Consolidated"]
related_skills: ["gif-search", "spotify"]
---

# Media Interfaces Umbrella

Consolidates agent-created media integration skills.

## GIF Search


# GIF Search (Tenor API)

Search and download GIFs directly via the Tenor API using curl. No extra tools needed.

## When to use

Useful for finding reaction GIFs, creating visual content, and sending GIFs in chat.

## Setup

Set your Tenor API key in your environment (add to `~/.hermes/.env`):

```bash
TENOR_API_KEY=your_key_here
```

Get a free API key at https://developers.google.com/tenor/guides/quickstart — the Google Cloud Console Tenor API key is free and has generous rate limits.

## Prerequisites

- `curl` and `jq` (both standard on macOS/Linux)
- `TENOR_API_KEY` environment variable

## Search for GIFs

```bash
# Search and get GIF URLs
curl -s "https://tenor.googleapis.com/v2/search?q=thumbs+up&limit=5&key=${TENOR_API_KEY}" | jq -r '.results[].media_formats.gif.url'

# Get smaller/preview versions
curl -s "https://tenor.googleapis.com/v2/search?q=nice+work&limit=3&key=${TENOR_API_KEY}" | jq -r '.results[].media_formats.tinygif.url'
```

## Download a GIF

```bash
# Search and download the top result
URL=$(curl -s "https://tenor.googleapis.com/v2/search?q=celebration&limit=1&key=${TENOR_API_KEY}" | jq -r '.results[0].media_formats.gif.url')
curl -sL "$URL" -o celebration.gif
```

## Get Full Metadata

```bash
curl -s "https://tenor.googleapis.com/v2/search?q=cat&limit=3&key=${TENOR_API_KEY}" | jq '.results[] | {title: .title, url: .media_formats.gif.url, preview: .media_formats.tinygif.url, dimensions: .media_formats.gif.dims}'
```

## API Parameters

| Parameter | Description |
|-----------|-------------|
| `q` | Search query (URL-encode spaces as `+`) |
| `limit` | Max results (1-50, default 20) |
| `key` | API key (from `$TENOR_API_KEY` env var) |
| `media_filter` | Filter formats: `gif`, `tinygif`, `mp4`, `tinymp4`, `webm` |
| `contentfilter` | Safety: `off`, `low`, `medium`, `high` |
| `locale` | Language: `en_US`, `es`, `fr`, etc. |

## Available Media Formats

Each result has multiple formats under `.media_formats`:

| Format | Use case |
|--------|----------|
| `gif` | Full quality GIF |
| `tinygif` | Small preview GIF |
| `mp4` | Video version (smaller file size) |
| `tinymp4` | Small preview video |
| `webm` | WebM video |
| `nanogif` | Tiny thumbnail |

## Notes

- URL-encode the query: spaces as `+`, special chars as `%XX`
- For sending in chat, `tinygif` URLs are lighter weight
- GIF URLs can be used directly in markdown: `![alt](url)`


## Spotify Playback


# Spotify

Control the user's Spotify account via the Hermes Spotify toolset (7 tools). Setup guide: https://hermes-agent.nousresearch.com/docs/user-guide/features/spotify

## When to use this skill

The user says something like "play X", "pause", "skip", "queue up X", "what's playing", "search for X", "add to my X playlist", "make a playlist", "save this to my library", etc.

## The 7 tools

- `spotify_playback` — play, pause, next, previous, seek, set_repeat, set_shuffle, set_volume, get_state, get_currently_playing, recently_played
- `spotify_devices` — list, transfer
- `spotify_queue` — get, add
- `spotify_search` — search the catalog
- `spotify_playlists` — list, get, create, add_items, remove_items, update_details
- `spotify_albums` — get, tracks
- `spotify_library` — list/save/remove with `kind: "tracks"|"albums"`

Playback-mutating actions require Spotify Premium; search/library/playlist ops work on Free.

## Canonical patterns (minimize tool calls)

### "Play <artist/track/album>"
One search, then play by URI. Do NOT loop through search results describing them unless the user asked for options.

```
spotify_search({"query": "miles davis kind of blue", "types": ["album"], "limit": 1})
→ got album URI spotify:album:1weenld61qoidwYuZ1GESA
spotify_playback({"action": "play", "context_uri": "spotify:album:1weenld61qoidwYuZ1GESA"})
```

For "play some <artist>" (no specific song), prefer `types: ["artist"]` and play the artist context URI — Spotify handles smart shuffle. If the user says "the song" or "that track", search `types: ["track"]` and pass `uris: [track_uri]` to play.

### "What's playing?" / "What am I listening to?"
Single call — don't chain get_state after get_currently_playing.

```
spotify_playback({"action": "get_currently_playing"})
```

If it returns 204/empty (`is_playing: false`), tell the user nothing is playing. Don't retry.

### "Pause" / "Skip" / "Volume 50"
Direct action, no preflight inspection needed.

```
spotify_playback({"action": "pause"})
spotify_playback({"action": "next"})
spotify_playback({"action": "set_volume", "volume_percent": 50})
```

### "Add to my <playlist name> playlist"
1. `spotify_playlists list` to find the playlist ID by name
2. Get the track URI (from currently playing, or search)
3. `spotify_playlists add_items` with the playlist_id and URIs

```
spotify_playlists({"action": "list"})
→ found "Late Night Jazz" = 37i9dQZF1DX4wta20PHgwo
spotify_playback({"action": "get_currently_playing"})
→ current track uri = spotify:track:0DiWol3AO6WpXZgp0goxAV
spotify_playlists({"action": "add_items",
                   "playlist_id": "37i9dQZF1DX4wta20PHgwo",
                   "uris": ["spotify:track:0DiWol3AO6WpXZgp0goxAV"]})
```

### "Create a playlist called X and add the last 3 songs I played"
```
spotify_playback({"action": "recently_played", "limit": 3})
spotify_playlists({"action": "create", "name": "Focus 2026"})
→ got playlist_id back in response
spotify_playlists({"action": "add_items", "playlist_id": <id>, "uris": [<3 uris>]})
```

### "Save / unsave / is this saved?"
Use `spotify_library` with the right `kind`.

```
spotify_library({"kind": "tracks", "action": "save", "uris": ["spotify:track:..."]})
spotify_library({"kind": "albums", "action": "list", "limit": 50})
```

### "Transfer playback to my <device>"
```
spotify_devices({"action": "list"})
→ pick the device_id by matching name/type
spotify_devices({"action": "transfer", "device_id": "<id>", "play": true})
```

## Critical failure modes

**`403 Forbidden — No active device found`** on any playback action means Spotify isn't running anywhere. Tell the user: "Open Spotify on your phone/desktop/web player first, start any track for a second, then retry." Don't retry the tool call blindly — it will fail the same way. You can call `spotify_devices list` to confirm; an empty list means no active device.

**`403 Forbidden — Premium required`** means the user is on Free and tried to mutate playback. Don't retry; tell them this action needs Premium. Reads still work (search, playlists, library, get_state).

**`204 No Content` on `get_currently_playing`** is NOT an error — it means nothing is playing. The tool returns `is_playing: false`. Just report that to the user.

**`429 Too Many Requests`** = rate limit. Wait and retry once. If it keeps happening, you're looping — stop.

**`401 Unauthorized` after a retry** — refresh token revoked. Tell the user to run `hermes auth spotify` again.

## URI and ID formats

Spotify uses three interchangeable ID formats. The tools accept all three and normalize:

- URI: `spotify:track:0DiWol3AO6WpXZgp0goxAV` (preferred)
- URL: `https://open.spotify.com/track/0DiWol3AO6WpXZgp0goxAV`
- Bare ID: `0DiWol3AO6WpXZgp0goxAV`

When in doubt, use full URIs. Search results return URIs in the `uri` field — pass those directly.

Entity types: `track`, `album`, `artist`, `playlist`, `show`, `episode`. Use the right type for the action — `spotify_playback.play` with a `context_uri` expects album/playlist/artist; `uris` expects an array of track URIs.

## What NOT to do

- **Don't call `get_state` before every action.** Spotify accepts play/pause/skip without preflight. Only inspect state when the user asked "what's playing" or you need to reason about device/track.
- **Don't describe search results unless asked.** If the user said "play X", search, grab the top URI, play it. They'll hear it's wrong if it's wrong.
- **Don't retry on `403 Premium required` or `403 No active device`.** Those are permanent until user action.
- **Don't use `spotify_search` to find a playlist by name** — that searches the public Spotify catalog. User playlists come from `spotify_playlists list`.
- **Don't mix `kind: "tracks"` with album URIs** in `spotify_library` (or vice versa). The tool normalizes IDs but the API endpoint differs.

