# Completed games

A single-page catalog of games I've finished, across PlayStation, Steam and Apple.

The page is `index.html`. All the data lives in `games.json`, so adding a game means adding one entry there.

## Entry format

```json
{
  "title": "Elden Ring",
  "platform": "PlayStation",
  "system": "PS5",
  "status": "beaten",
  "proof": "Platinum (all 3 endings)",
  "lastPlayed": "2022-05",
  "platinum": true,
  "note": "",
  "cover": ""
}
```

- `platform`: the store or ecosystem: `PlayStation`, `Steam` or `Apple`. Each value gets its own filter tab automatically.
- `system`: the badge on the card, such as `PS5`, `PS VR2`, `PC`, `Mac` or `iPhone`.
- `status`: one of `beaten`, `unfinished`, `unknown` (shown as "No proof": the game has no achievement that marks the ending, so it can't be verified) or `endless` (games with no ending, like Rocket League).
- `proof`: the trophy or achievement that shows the game was finished.
- `lastPlayed`: `YYYY-MM`. The month of the last trophy or achievement earned.
- `cover`: optional image URL. Without one, the card uses a colored tile with the game's initials.

## Steam

Steam games use the store's header art as the cover. The list includes every game with an achievement unlocked or at least an hour of playtime; untouched library games are left out.

## How "beaten" is decided

A game counts as beaten when the trophy or achievement for the final story mission, the final boss or the credits is earned. Any ending counts. Where a game has a harder true ending, it's mentioned in `note`. When no trophy covers the ending (Demon's Souls' good ending, for example), the call is made by hand.

## Hosting

GitHub Pages: Settings → Pages → Deploy from a branch → `main` / root.
