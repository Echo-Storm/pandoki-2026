# Pandoki — Kodi Pandora Client
### Patched Build — v0.10.6

Based on the original by **gominoa, rocky4546, truckershitch**  
Source: https://github.com/rocky4546/gominoa-xbmc-addons  
License: GNU GPL v3

---

## What This Is

Pandoki is a Kodi audio plugin that connects to Pandora Internet Radio. It lets you browse and play your Pandora stations directly from Kodi, with support for thumbs up/down rating, station creation and management, song caching, and optional local library saving.

This is a patched version of v0.10.5 with bug fixes and quality-of-life improvements applied. No structural changes were made — the core Pandora/pithos connection logic, caching system, and playlist management are untouched.

---

## Requirements

- Kodi 19 (Matrix) or later
- Python 3
- `script.module.urllib3` (version 1.25.8+matrix.1 or later) — installable from the official Kodi repository
- A Pandora account (free or Pandora One/Premium)
- Pandora must be accessible from your network (US only without a proxy)

---

## Installation

1. Download `plugin.audio.pandoki_0.10.6.zip`
2. In Kodi: Add-ons → Install from zip file → select the zip
3. Open the addon settings and enter your Pandora credentials under the Profile tab
4. Launch the addon from Add-ons → Music

---

## Settings Reference

### Profile Tab

| Setting | Description |
|---|---|
| Account | Switch between up to 3 saved Pandora accounts |
| Username | Your Pandora email address |
| Password | Your Pandora password |
| Pandora One | Enable if you have a Pandora One or Premium subscription |
| Use Proxy | Global (Kodi default), None, or Custom proxy settings |
| SNI Support | Enable for stricter SSL certificate validation via urllib3 |

### Playback Tab

| Setting | Description |
|---|---|
| Auto Play Last Station | Resumes your last station when the addon opens |
| Sort Stations | Newest / A-Z / Oldest / Shuffle |
| Audio Quality | Low / Medium / High |
| Rating Mode | Basic (thumbs) or Expert (branch/seed on 5-star rating) |
| Track Handling | Stream Only (no disk use) or Cache Only (downloads before playing) |
| Concurrent Downloads | How many songs to prefetch at once (Cache mode only) |
| Save Playlist | Write an M3U playlist file alongside saved songs |
| Skip Small Songs | Auto-skip tracks below the Min Song Size threshold (usually ads) |

### Advanced Tab

| Setting | Description |
|---|---|
| Debug | Write verbose logs to the Kodi log file |
| Notification | Show on-screen popups for caching, buffering, and rating events |
| Playlist History | How many songs to keep in the Kodi playlist before pruning old ones |
| Min Song Size | File size threshold in KB below which a track is treated as an ad |
| Track Prefetch | Seconds of audio to buffer before queuing the next song |

### Folders Tab

| Setting | Description |
|---|---|
| Cache | Where downloaded songs are stored temporarily |
| Library | Where liked/saved songs are stored permanently |

---

## Rating Modes

**Basic mode** (default):
- 5 stars → Thumb Up + save to library
- 4 stars → Thumb Up + save to library
- 3 stars → Thumb Up + save to library
- 2 stars → Thumb Down + skip
- 1 star → Thumb Down + skip
- Empty → Clear feedback

**Expert mode:**
- 5 stars → Branch (create a new station based on this song)
- 4 stars → Seed (add this song as a seed to the current station)
- 3 stars → Thumb Up + save to library
- 2 stars → Set tired (Pandora won't play this for 30 days)
- 1 star → Thumb Down + skip
- Empty → Clear feedback

Context menu options while playing:
- **Thumb Up** — like the current song
- **Thumb Down** — dislike and skip
- **I'm Tired of This** — snooze for 30 days
- **Clear Vote** — remove your rating
- **Branch Station** — create a new station from this song
- **Seed Station** — add this song as a seed to the current station

---

## Changelog

### v0.10.6 — Patched Build
*Bugs fixed:*

- **`Val()` write sentinel** — Setting a value to an empty string, `0`, or `False` now correctly saves the setting. Previously, passing any falsy value would silently read instead of write, meaning clearing a setting field had no effect.

- **Filename sanitizer** — Removed `.` (period) from the list of illegal filename characters. It is valid on all platforms Kodi runs on, and its removal was mangling artist and album names like "Jr.", "Vol. 1", and "U.S.A." in both the cache and library paths.

- **MusicBrainz `Tag()` crash** — The function previously hardcoded `medium-list[1]`, which caused an `IndexError` crash on any single-disc release (the most common case). Now correctly falls back to index 0 when only one disc is present. Affected: song library saving with MusicBrainz tagging enabled.

- **`notification()` broken call** — In the "Song Stopped Buffering" code path, a missing comma between two string literals caused Python to silently concatenate them, dropping the first argument entirely. The notification was malformed and the function was also being called with 3 arguments instead of the required 4.

- **Bare `except` clauses** — Replaced bare `except:` with `except Exception:` in the cache download and tag lookup paths. Bare `except` catches `SystemExit` and `KeyboardInterrupt`, which can cause Kodi to hang on shutdown or fail to abort cleanly.

*Improvements:*

- **Shuffle sort option** — Added "Shuffle" as a fourth option in Sort Stations. Randomizes the station list order each time the directory loads (QuickMix always stays at the top).

- **Better login error message** — Login failure dialog now shows which account number failed ("Account 1", "Account 2", etc.) and lists the three things to check: username, password, and the Pandora One toggle.

- **Station list cache reduced** — Reduced from 5 minutes to 2 minutes. After renaming, creating, or deleting a station, your changes now appear in the list sooner without meaningfully increasing API calls.

- **Main loop poll rate reduced** — The internal event loop polled at 0.2 seconds (5 times per second). Reduced to 0.5 seconds (2 times per second). A music streaming client has no time-critical work to do at that frequency; this reduces idle CPU usage with no functional impact.

---

### v0.10.9
*Changes:*

- **Music Virtualizer context menu label is now state-aware** — Instead of always showing "Toggle Music Virtualizer", the label now reads the current setting when the station list loads and displays either `Music Virtualizer: ON  [toggle off]` or `Music Virtualizer: OFF [toggle on]`. Falls back to the neutral label if the state can't be read. The JSON-RPC call is made once per directory load, not once per station item.

---

### v0.10.8
*Changes:*

- **Renamed "Toggle Virtualizer" to "Toggle Music Virtualizer"** — label clarified to distinguish it clearly from the visualizer feature added in the same version.

- **Launch Visualizer** context menu item — Right-click any station → "Launch Visualizer" opens Kodi's active music visualizer (Milkdrop, Spectrum, or whatever is configured in Kodi's music settings). Only meaningful while a station is actually playing.

---

### v0.10.7
*Bugs fixed (from log analysis):*

- **`Search()` crash on Pandora API error** — When Pandora returned an error during station search (e.g. auth lapse, rate limit, or server hiccup), a `PithosError` propagated uncaught all the way through `Loop()` and crashed the plugin instance. Now caught at the `Search()` level: shows a notification, closes the directory cleanly, and the plugin keeps running.

- **`setInfo()` deprecation warnings** — `li.setInfo('music', ...)` is deprecated in Kodi 21 and was flooding the log with warnings every time a song was added to the playlist. Replaced with `li.getMusicInfoTag()` setter calls which are the correct API for Kodi 21+. The log is now clean during normal playback.

*New feature:*

- **Toggle Virtualizer** context menu item — Right-click any station in the station list to toggle Kodi's audio virtualizer (stereo upmix / headphone surround) on or off without leaving the addon. Shows a green ON / red OFF notification confirming the new state. Uses JSON-RPC to read and write `audiooutput.stereoupmix` directly. The notification setting must be enabled in Advanced settings for the confirmation popup to appear.

---

### v0.10.5 — Original Upstream
Base version by gominoa, rocky4546, truckershitch. See upstream repo for full history.

---

## Known Limitations

- Pandora is only available in the United States. Outside the US you will need a VPN or proxy configured in the addon settings.
- The free Pandora tier limits skips per hour. Thumb Down counts as a skip.
- MusicBrainz tagging (used for library saving) requires a 100% confidence match. Songs that don't match are cached but not saved to the library — this is by design.
- Advertisement detection is based on file size. The default threshold works for most cases but may need tuning in the Advanced settings if legitimate short songs are being skipped.
- The addon uses Pandora's unofficial API via the mypithos library. Pandora could change or restrict this API at any time without notice.

---

## Files Modified in This Patch

| File | Changes |
|---|---|
| `resources/lib/pandoki/pandoki.py` | All bug fixes, improvements, and new features |
| `default.py` | Added `virtualizer` query parameter parsing and dispatch |
| `resources/settings.xml` | Added Shuffle option to Sort Stations |
| `addon.xml` | Version bumped to 0.10.9 |
