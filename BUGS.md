# Known Bugs

This file records issues found while testing the playlist generator. Locations link to the relevant code.

The issues listed below have been addressed in the current implementation. The entries remain as a record of the original behavior and the corresponding repair.

## Reported testing issues

### 1. Duplicate songs are unchecked (fixed)

The add-song flow previously appended every submitted song without checking whether the same title and artist already existed. The UI now rejects duplicate title/artist pairs after normalization.

Location: [app.py](app.py#L240-L255)

The data model now exposes a normalized title/artist identity check used by the add-song flow: [playlist_logic.py](playlist_logic.py#L30-L69)

### 2. Energy-5 songs disappeared when Mixed was hidden (fixed)

With the default thresholds, an energy-5 song is neither Hype nor Chill, so it is classified as Mixed. The app now always renders the Mixed tab, so these songs remain visible.

Locations: [playlist_logic.py](playlist_logic.py#L57-L80), [app.py](app.py#L258-L270)

### 3. Energy-4 songs were hidden from the visible playlists (fixed)

The Chill rule remains `energy <= chill_max_energy`, with a default maximum of 3, so energy-4 songs are classified as Mixed. The app now always renders Mixed, preventing those songs from being hidden. This preserves the intended Chill range of levels 1-3.

Locations: [playlist_logic.py](playlist_logic.py#L57-L80), [app.py](app.py#L206-L210)

### 4. Most common artist appeared not to update after adding songs (fixed)

The reported behavior was related to the unreliable add/reset workflow. The add flow now preserves existing songs, rejects only true duplicates, and the stats calculation counts all songs.

Locations: [app.py](app.py#L251-L255), [playlist_logic.py](playlist_logic.py#L109-L138), [playlist_logic.py](playlist_logic.py#L141-L154)

### 5. Lucky pick never selected Mixed songs (fixed)

The `any` mode now combines Hype, Chill, and Mixed playlists.

Location: [playlist_logic.py](playlist_logic.py#L177-L189)

## Additional confirmed issues

### Lucky pick crashed for an empty playlist (fixed)

`random_choice_or_none()` documents an optional result but calls `random.choice()` without checking for an empty list. The warning branch in the UI is therefore unreachable for empty Hype, Chill, or combined playlists.

Location: [playlist_logic.py](playlist_logic.py#L192-L196)

### Artist search failed for partial queries (fixed)

The search condition checks whether the full stored artist name is contained in the query. Searching for `laufey` will not match `laufey feat. ...` or other longer values as users would expect; the predicate should normally check whether the query is contained in the stored value.

Location: [playlist_logic.py](playlist_logic.py#L157-L174)

### Playlist statistics used incorrect denominators (fixed)

`hype_ratio` divides the number of Hype songs by itself, so it is always `1.0` whenever Hype is non-empty. `avg_energy` sums only Hype-song energy but divides by the total number of songs, so it is not the average energy of the playlist.

Location: [playlist_logic.py](playlist_logic.py#L109-L138)

### Invalid energy values could crash classification (fixed)

String energy values that are not integers become `0`, but values such as `None` remain unchanged. Classification can then fail when comparing them with the numeric thresholds.

Locations: [playlist_logic.py](playlist_logic.py#L30-L45), [playlist_logic.py](playlist_logic.py#L57-L80)

### Playlist merging mutated its input lists (fixed)

`merge_playlists()` assigns each list from the first input directly into the result and extends it. As a result, changing the returned playlist can also change the original input playlist.

Location: [playlist_logic.py](playlist_logic.py#L100-L106)