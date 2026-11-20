# Updating the catalog

This is the runbook for refreshing `games.json` from PlayStation, Steam and Apple. A scheduled task runs it every 3 months; it also works by hand.

The goal of each run is small: find games that are new, or that moved from unfinished to beaten, and record them. Most of the catalog never changes.

## Ground rules

1. **Never touch an entry with `"manual": true`.** Those are calls Jason made himself (games with no finishing achievement, Demon's Souls' good ending). Don't change their status, proof, note or date.
2. **Never downgrade.** A game marked `beaten` stays beaten. A run can only move `unfinished` or `unknown` to `beaten`, or add new games.
3. **Only add games that were actually played.** At least one trophy or achievement, or (Steam only) an hour of playtime. Untouched library games stay out.
4. **When the finishing achievement is unclear, don't guess.** Set `status: "unknown"`, write what you saw in `note`, and list the game in the PR description under "Needs your call".
5. **Open a pull request; don't push to `main`.** Branch name `update/YYYY-MM-DD`. Jason reviews and merges.

## What "beaten" means

A game is beaten when the trophy or achievement for the final story mission, the final boss or the credits is unlocked. Any ending counts. A platinum or 100% counts. Roguelikes count after the first full win (Hades: first escape; Balatro: "Win a run"). Games with no ending at all (sports, multiplayer, pinball, endless runners) are `endless`.

Read the achievement list, find the one that marks the ending, and check whether it's unlocked. The highest-rarity story achievements are usually the progression ones, which helps when names are cryptic. Watch for these traps:

- The finishing achievement can be for one specific choice (Demon's Souls PS5 has a trophy for the bad ending only). If no trophy covers the ending, use `unknown`.
- Collections hold more than one game. Split them (Uncharted: Legacy of Thieves is two entries).
- A game can appear twice on PSN (PS4 and PS5 versions). Keep the one with more progress.

## Entry format

See `README.md`. Fields a run sets:

| Field | Value |
|---|---|
| `proof` | Name of the deciding achievement plus a short description, e.g. `Mono No Aware (final story trophy)` |
| `lastPlayed` | `YYYY-MM` of the most recent unlock |
| `platinum` | PlayStation only: `true` if the platinum is earned |
| `note` | Anything still open ("True ending open", "Chapter 9 of 10") |
| `cover` | Local path under `covers/` (never a remote URL) |
| `coverFit` | `"square"` for PlayStation and Apple icons |
| `steamAppId` | Steam only |

Also update the top-level `updated` date.

## Work incrementally

Before checking anything, load `games.json` and skip:

- every `manual` entry
- every `beaten` and `endless` entry

Re-check only `unfinished` and `unknown` entries, plus games that aren't in the file yet. On a typical run that's a few dozen pages, not hundreds.

## PlayStation (PSNProfiles)

Profile: https://psnprofiles.com/MayoTheCondiment

- PSNProfiles blocks `curl` and WebFetch (HTTP 403). Use a real browser: the built-in browser or Claude in Chrome.
- On the profile page, the games table is `#gamesTable`. Each row has `a.title` (name and link), the platform, `X of Y Trophies`, the last trophy date and an icon `img` (`/game/s/` is 100px; swap to `/game/l/` for 320px).
- Each game's trophy page is `/trophies/<id>-<slug>/MayoTheCondiment`. Trophy rows are `table.zebra tr` with `a.title`; an earned trophy's row has class `completed`. The last `%` in the row is the PSNProfiles rarity.
- Fetch trophy pages with `fetch()` from inside the profile tab. PSNProfiles rate-limits: keep about 3 seconds between requests and retry 429s or empty responses after 15 seconds.
- The cover image download from `img.psnprofiles.com` works with plain `curl`. Save it as WebP at `covers/psn/<gameId>.webp`.

## Steam

Profile: https://steamcommunity.com/id/maondas/

- The games list (`/id/maondas/games/?tab=all`) only loads for a signed-in visitor, so the browser must be signed in to Steam. If it isn't, stop the Steam part and say so in the PR.
- The list is virtualized. Scroll it in steps and collect `a[href*="store.steampowered.com/app/"]` with the `img[alt]` name.
- Achievements: `/id/maondas/achievements/<appid>`. The page embeds JSON in `"queryData":"…"` (a JSON string inside the HTML). Parse it. The queries you need:
  - `["achievements", appid, "schema_v2"]`: names, descriptions, DLC group
  - `["achievements", appid, "user_achievements", steamid]`: `unlocked`, `unlock_time`
  - `["achievements", appid, "globalpercentages"]`
  - `["FriendsGameplayInfo", steamid, appid, true]`: `your_info.minutes_played_forever`
- About 1 request per second is fine.
- Cover art: `https://shared.akamai.steamstatic.com/store_item_assets/steam/apps/<appid>/header.jpg` works with `curl`. Save it as WebP at `covers/steam/<appid>.webp`.

## Apple (Game Center)

There is no web page for Game Center. The data is only in the **Games** app on Jason's Mac (Library → Achievements), so this part needs computer use on that Mac.

- The Achievements screen lists every game. Games with no "Played …" line have zero achievements, so skip them.
- Opening a game shows `N/M Total Completed`, a Completed row (newest first) and a Locked grid. Read the Locked grid: if the ending achievement is locked, the game isn't beaten.
- Use the background `app_*` tools; don't take over Jason's screen.
- Icons: the iTunes Search API (`https://itunes.apple.com/search?term=…&entity=software`) works with `curl`. Check that `trackName` actually matches before using `artworkUrl512`, because search often returns a sequel or an unrelated app. Save as WebP at `covers/apple/<slug>.webp`, resized to 256px.

## Finish

1. Validate `games.json` (it must parse, and every `cover` path must exist).
2. Commit on `update/YYYY-MM-DD` and open a PR. The description should list:
   - newly beaten games and the deciding achievement for each
   - new games added, by status
   - **Needs your call**: anything set to `unknown`
   - anything that couldn't run (Steam signed out, Mac offline)
3. If nothing changed, don't open a PR. Just report "no changes".
