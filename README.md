# Playlist Picker

A single-page tablet remote for a young child. It shows the songs from the parent-chosen `Kid -` Spotify playlist as a scrolling grid of album-art tiles. Tapping a tile plays that song on the living room TV, and a bar at the bottom has one big play/pause button. The grid is paged, so it works for hundreds of songs.

On a phone or tablet the page cannot stop a child from leaving it. Use Guided Access (iPhone/iPad) or screen pinning (Android), described below, to lock the device to the page.

The page is one static file (`index.html`). It has no server, no build step, and no dependencies beyond Google Fonts.

## How it works

- The page signs in to Spotify using the Authorization Code flow with PKCE, which needs no client secret and no backend.
- It lists the signed-in account's playlists and keeps only those whose names start with `Kid -`.
- Tapping a tile calls Spotify's "start playback" endpoint with that playlist and the chosen TV as the target device. The tablet is only a remote control. The audio plays on the TV.
- The player buttons call the previous, pause/resume, and next endpoints.
- Every few seconds the page checks Spotify for the current state, so it stays correct if the music is changed from a phone or the TV remote.

## Requirements

- A Spotify **Premium** account. Spotify only allows remote playback control on Premium.
- A TV that appears as a Spotify Connect device. A Roku TV works when the Spotify app on the TV is signed in to the same Premium account.
- A free Spotify developer app (for the Client ID).
- Any host that serves static files over HTTPS (GitHub Pages and Netlify both have free plans that work). Spotify rejects plain `http://` redirect addresses, except loopback addresses such as `http://127.0.0.1`.

## Setup

### 1. Create the Spotify developer app

1. Go to https://developer.spotify.com/dashboard and create an app.
2. Choose **Web API** when asked which APIs you plan to use.
3. Copy the app's **Client ID**.
4. Under the app's settings, add a **Redirect URI** that matches the final address of the page exactly, including any trailing slash. Example: `https://YOUR-USERNAME.github.io/kid-jukebox/`

### 2. Set the Client ID

Open `index.html` and find the configuration block near the top of the script:

```js
var CLIENT_ID = "your-client-id";
var PREFIX_RE = /^kid\s*[-–—:]\s*/i;   // playlists must start with "Kid -"
var TV_HINT = "roku";                   // used until a device is chosen in grown-up settings
```

Replace the value of `CLIENT_ID` with your own. The Client ID is not a secret.

### 3. Publish the page

**GitHub Pages**

1. Put `index.html` in a **public** repository. GitHub Pages is free for public repositories only.
2. In the repository, go to Settings, then Pages, and publish from the `main` branch.
3. The page is served at `https://YOUR-USERNAME.github.io/REPOSITORY-NAME/`.

**Netlify Drop** (no repository needed)

1. Go to https://app.netlify.com/drop.
2. Drag `index.html` onto the page.
3. Use the `https://...netlify.app` address it gives you as the Redirect URI.

### 4. Sign in once

1. Open the page on the tablet and tap **Sign in with Spotify**.
2. Approve the permissions. The login is stored on the tablet, so this is normally a one-time step.
3. Add the page to the tablet's home screen.

## Choosing which playlists appear

Name any Spotify playlist so that it starts with `Kid -`, for example `Kid - Dino Stomp`. The prefix is removed on the tablet, so the tile reads "Dino Stomp".

- No code changes are needed. Rename a playlist in Spotify to add it or remove it.
- Playlists owned by another account (for example a spouse's) appear only if the signed-in account follows them or they are collaborative.
- Press **Refresh playlists** in the grown-up settings, or reopen the page, to pick up changes.

## Grown-up settings

Press and hold the **Our Music** title for two seconds. The settings let you:

- Choose which device plays the music. The choice is saved on the tablet.
- Refresh the playlist list.
- Sign out.

If no device has been chosen, the page picks one in this order: a device whose name contains the `TV_HINT` text, then a device Spotify labels as a TV, then any device that is not a phone, tablet, or computer.

## Screens

- **Playlists:** large tiles, swipe sideways, tap to play. A **Now playing** button appears while music is playing so the child can return to the player.
- **Player:** three full-height columns, so a tap anywhere in a column counts. Previous is on the left, play/pause is in the middle (the widest), and next is on the right. **Playlists** at the top returns to the tiles.

## Keeping the tablet locked to the page

- **iPad:** turn on Guided Access (Settings, Accessibility, Guided Access) and start it with a triple-click of the side button while the page is open.
- **Android:** turn on screen pinning (Settings, Security) and pin the browser.

## Updating the page

1. Replace `index.html` in the repository, or drag the new file onto the same Netlify site's Deploys page.
2. Wait a minute or two for the host to publish.
3. On the tablet, close the page and reopen it. If the old version still shows, add `?v=2` (or a new number) to the end of the address, or remove and re-add the home-screen shortcut.

When uploading from a browser download, make sure the file is named exactly `index.html`. A name such as `index (1).html` is treated as a separate file.

## Troubleshooting

| Message or symptom | Likely cause and fix |
| --- | --- |
| "TV not found" with a list of devices | The TV is not among the devices Spotify returned. Press and hold **Our Music** and choose the TV from the list. |
| "Spotify can't see the TV" | The Spotify app on the TV is closed or signed out. Open it, sign in with the same Premium account, and play a song on the TV once. |
| TV works from the phone but not from the tablet | Casting from a phone can use a temporary session that the control API does not list. Sign in to the Spotify app on the TV with the same account. |
| "Spotify says no. Premium is needed." | The signed-in account does not have Premium. |
| "Too many taps" | Spotify rate-limited the page. Wait a moment. |
| No tiles appear | No playlist name starts with `Kid -`, or the account that owns them is not the one signed in. |
| Sign-in loops or fails | The Redirect URI in the Spotify dashboard does not exactly match the page address, including the trailing slash. |
| Old behavior after an update | The tablet is showing a cached copy. See "Updating the page". |

## Limits

- Spotify's control API cannot switch the Roku between audio and video mode. That choice is made in the Spotify app on the TV.
- Developer-mode Spotify apps work for the app owner and up to 25 added users, which is enough for personal use.

## Privacy and security

- The Client ID in the page is public by design. There are no passwords or client secrets in the file.
- Access and refresh tokens are stored in the tablet browser's local storage only and are sent only to Spotify.
- The page requests these permissions: read playback state, modify playback, read private playlists, and read collaborative playlists. It cannot change playlists or read anything else.
- **Sign out** in the grown-up settings clears the stored login. You can also remove the app's access at https://www.spotify.com/account/apps/.

## Files

- `index.html`: the entire app (markup, styles, and script).
- `README.md`: this document.
