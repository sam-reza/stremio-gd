# stremio-gd

A Cloudflare Workers addon for [Stremio](https://www.stremio.com/) that lets you stream video files directly from your Google Drive.

By default, it searches your entire Google Drive and any shared drives you have access to for videos and presents them in Stremio. You can optionally restrict results to specific folders via `CONFIG.driveFolderIds`.

Based on [Viren070/stremio-gdrive-addon](https://github.com/Viren070/stremio-gdrive-addon) — with bug fixes applied. See [Changes in v2](#changes-in-v2).

---

## Features

- Search your Google Drive and shared drives for videos
- Parses filenames accurately using regex and displays results in an appealing format
- Catalog support — both search and full list on the home page
- Kitsu support
- TMDB metadata support if a TMDB API key is provided
- Optional filtering by specific Google Drive folders (non-recursive)
- Easily configurable via the `CONFIG` object at the top of the script — see [Configuration](#configuration)
- Single file deployment — easy to deploy and update

---

## Files

| File | Description |
|------|-------------|
| `stremio-gdrive-addon-v1.js` | Original script (unmodified) |
| `stremio-gdrive-addon-v2.js` | Patched script (recommended) |
| `gdrive-token-tool.html` | OAuth helper to get your refresh token |

**Token tool (GitHub Pages):** [gdrive-token-tool.html](https://sam-reza.github.io/stremio-gd/gdrive-token-tool.html)

---

## Deployment

### Setting up the Google App

1. Go to the [Google Cloud Console](https://console.cloud.google.com/) and sign in.

2. Create a new project and select it:

    1. Click **Select a project** in the top left
    2. Click **New Project**
    3. Enter a project name (e.g. `Stremio-GDrive`) and click **Create** — leave Organization blank
    4. Once created, click **Select Project** from the notification (or use the same dropdown)

3. Set up the Google Auth Platform:

    1. In the search bar at the top, search for `Google Auth Platform` and click the result
    2. You should see a message saying *Google Auth Platform not configured yet* — click **Get Started**
    3. Fill in the form:
        - **App name:** `Stremio GDrive` (or anything you like)
        - **User support email:** your email
        - **Audience / User type:** External
        - **Contact email:** your email
        - Check the box to agree to the Google API Services User Data Policy
    4. Click **Create**

4. Enable the Google Drive API:

    1. In the search bar at the top, search for `Google Drive API` and click the result
    2. Click **Enable**

5. Create an OAuth client:

    1. Go back to the **Google Auth Platform** page
    2. Click **Clients** in the sidebar → **+ Create Client**
    3. Fill in the form:
        - **Application type:** Desktop app
        - **Name:** `Stremio GDrive` (or anything)
    4. Click **Create**
    5. From the clients list, click the **download icon** for the client you just created
    6. A popup will show your **Client ID** and **Client Secret** — copy both

6. Get your refresh token:

    Open the token tool: **[gdrive-token-tool.html](https://sam-reza.github.io/stremio-gd/gdrive-token-tool.html)**

    1. Select scope — **Read-only** (recommended) or **Full Drive**
    2. Enter your **Client ID** and **Client Secret**
    3. Click **Open Google Auth** — sign in and allow access
        > You may see a warning: *Google hasn't verified this app*. Click **Advanced** → **Go to... (unsafe)**. This is normal for personal apps.
    4. Your browser will show a blank or error page — paste the full URL from the address bar into the token tool
    5. Click **Get Refresh Token** and copy the result

---

### Setting up the Cloudflare Worker

1. Go to [Cloudflare Workers](https://workers.cloudflare.com/) and log in or sign up.

2. From the Workers & Pages dashboard, click **Create**.

3. Make sure you're on the **Workers** tab and click **Create Worker**.

4. Give your worker a name — this becomes part of your addon URL. Click **Deploy**.

5. Once deployed, click **Edit code**.

6. In the editor, delete all existing code and paste the contents of `stremio-gdrive-addon-v2.js`.

7. At the top of the pasted code, fill in your credentials:

    ```js
    const CREDENTIALS = {
        clientId: "YOUR_CLIENT_ID",
        clientSecret: "YOUR_CLIENT_SECRET",
        refreshToken: "YOUR_REFRESH_TOKEN",
    };
    ```

8. Click **Deploy** in the top right.

9. Once deployed, click **Visit** to open your addon URL. You should be redirected to `/manifest.json`. If not, append `/manifest.json` to the URL manually.

10. Copy the URL and add it to Stremio:
    - Open Stremio → **Settings** → **Addons** → **Add addon**
    - Paste your manifest URL and confirm

Done! You can now stream videos from your Google Drive and shared drives directly in Stremio.

---

## Configuration

Edit the `CONFIG` block near the top of the script before deploying.

> All values are case sensitive unless stated otherwise.

| Name | Type | Values | Description |
|------|------|--------|-------------|
| `resolutions` | `String[]` | `"2160p"`, `"1080p"`, `"720p"`, `"480p"`, `"Unknown"` | Controls which resolutions appear in results. Order determines sort priority. Remove a value to exclude that resolution. `Unknown` appears when resolution cannot be determined from the filename. |
| `qualities` | `String[]` | `"BluRay REMUX"`, `"BluRay"`, `"WEB-DL"`, `"WEBRip"`, `"HDRip"`, `"HC HD-Rip"`, `"DVDRip"`, `"HDTV"`, `"CAM"`, `"TS"`, `"TC"`, `"SCR"` | Controls which qualities appear. Order determines sort priority. Remove a value to exclude that quality. |
| `visualTags` | `String[]` | `"HDR10+"`, `"HDR10"`, `"HDR"`, `"DV"`, `"IMAX"`, `"AI"` | Controls which visual tags appear and their sort priority. |
| `sortBy` | `String[]` | `"resolution"`, `"quality"`, `"size"`, `"visualTag"` | Order determines sort priority across criteria. E.g. `["resolution", "size"]` sorts by resolution first, then size within each resolution group. |
| `considerHdrTagsAsEqual` | `boolean` | `true`, `false` | When `true`, all HDR variants (HDR, HDR10, HDR10+) are treated equally when sorting by `visualTag`. When `false`, their position in the `visualTags` list determines individual priority. |
| `addonName` | `String` | any | Name shown in Stremio addon list and stream results. |
| `prioritiseLanguage` | `String` \| `null` | See `languages` in `REGEX_PATTERNS` | Pushes results matching this language to the top. Set to `null` to disable. |
| `proxiedPlayback` | `boolean` | `true`, `false` | When enabled, streams are proxied through the Worker. Required for Stremio Web and iOS external players. Recommended to leave enabled. |
| `tmdbApiKey` | `String` \| `null` | TMDB API key or `null` | Enables TMDB metadata lookup. Get a key at [themoviedb.org](https://www.themoviedb.org/). |
| `enableSearchCatalog` | `boolean` | `true`, `false` | Enable the search catalog in Stremio. |
| `enableVideoCatalog` | `boolean` | `true`, `false` | Enable the browse/list catalog on the Stremio home page. |
| `showAudioFiles` | `boolean` | `true`, `false` | Include audio files in stream results. |
| `maxFilesToFetch` | `number` | any integer | Maximum number of Drive files to scan per request. |
| `strictTitleCheck` | `boolean` | `true`, `false` | Stricter title matching to reduce false positives. |
| `driveQueryTerms.episodeFormat` | `String` | `"name"`, `"fullText"` | Field used to query episode formats (e.g. s01e03). Default `fullText` is recommended; switch to `name` if getting incorrect matches. |
| `driveQueryTerms.movieYear` | `String` | `"name"`, `"fullText"` | Field used to query movie release year. Default `name` is recommended. |
| `driveFolderIds` | `String[]` | Drive folder IDs | Restricts results to files directly inside the listed folders. Non-recursive — subfolders are not included automatically. |

### Folder filtering

```js
const CONFIG = {
    // ...
    driveFolderIds: [
        "1aBcD2E3fG4hiJ5k6LMnOpQRs7tuVXy-8",
        "another-folder-id"
    ]
};
```

**How to get a folder ID:**
- Open the folder in Google Drive — copy the part after `/folders/` in the URL
- Or right-click the folder → **Get link** → the ID is the string between `/folders/` and `?`
- Works for personal Drive and shared drives

> The filter is non-recursive. Add each subfolder ID separately if needed. The authorized account must have access to all listed folders.

---

## Changes in v2

Bug fixes applied to the original script:

| # | Issue | Fix |
|---|-------|-----|
| 1 | `MANIFEST` object mutated on every request — catalog entries accumulate on warm Workers | `structuredClone(MANIFEST)` instead of direct reference |
| 2 | `getAccessToken` crashes on non-JSON error responses (e.g. 502 HTML pages) | Safe `JSON.parse` with fallback to raw message |
| 3 | `%3A` URL decode only replaces first occurrence | Changed `.replace("%3A", ":")` to `.replace(/%3A/g, ":")` |
| 4 | `enableSearchCatalog` and `enableVideoCatalog` flags wired to wrong catalog IDs | Swapped to match correct `gdrive_list` / `gdrive_search` IDs |
| 5 | `formatDuration` produces `1:5:3` instead of `1:05:03` | Added `padStart(2, "0")` to minutes and seconds |
| 6 | Audio files filtered out when `"Unknown"` removed from resolutions/qualities | Audio mimeTypes now bypass the resolution/quality filter |
| 7 | Proxied stream response missing `Content-Type` header | `Content-Type` forwarded from Drive response |

---

## Notes

- Credentials are stored in plaintext inside the Worker script. For better security, use [Cloudflare Worker Secrets](https://developers.cloudflare.com/workers/configuration/secrets/) instead.
- Each OAuth authorization code can only be used once. If token exchange fails, re-authorize via the token tool to get a fresh code.
- The Cloudflare Workers free tier allows 100,000 requests/day — sufficient for personal use.

---

## License

MIT
