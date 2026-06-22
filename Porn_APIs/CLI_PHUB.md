# PornHub API - Command Line Interface (CLI)

The `phub` package includes a built-in Command Line Interface (CLI) that allows you to easily download content directly from the terminal without writing any code. The CLI supports downloading individual videos, playlists, shorts, GIFs, albums, and even user profiles or models.

You can also log in to your account through the CLI to download your personalized lists, such as liked videos, recommended videos, and watch history!

## Usage

You can run the CLI directly from the module:

```bash
phub [OPTIONS]
```

## Basic Arguments

- `--download <URL>`: A direct link to the content you want to download. This can be a Video, Short, GIF, Album, Playlist, Channel, User, Model, or Pornstar URL.
- `--file <PATH>`: Path to a text file containing a list of URLs to download (separated by newlines). Use this if you want to batch download multiple links.
- `--quality <best|half|worst>`: **(Required)** Specifies the desired video quality for the download.
- `--output <PATH>`: **(Required)** Specifies the output directory (or absolute path) where the content will be saved.
- `--no-title <True|False>`: **(Required)** Whether to automatically apply the video's title to the output file. If set to `True`, the script expects `--output` to be the full file path.
- `--id-as-title`: A flag that overrides the title mechanism to use the Video ID as the filename instead of the default title. When using this, ensure your `--output` points to a directory.
- `--limit <INT>`: The maximum number of videos to download during the session. Highly recommended when downloading from large user profiles or playlists.
- `--pages <INT>`: The number of pages to fetch when parsing iterables (like Playlists or Profiles). Defaults to 1 page.

## Account & Login Arguments

To access features that require an account (such as liked videos or recommendations), you must provide your login credentials:

- `--email <EMAIL>`: Your account email.
- `--password <PASSWORD>`: Your account password.
- `--liked`: Downloads videos from your Liked/Favorites list.
- `--recommended`: Downloads videos from your Recommended list.
- `--watched`: Downloads videos from your Watch History.

*(Note: The `--liked`, `--recommended`, and `--watched` flags will be ignored if `--email` and `--password` are not provided).*

---

## Examples

### 1. Download a single video
Downloads a single video in the `best` quality and saves it to the `./downloads/` folder using its default title.

```bash
phub --download "https://www.pornhub.com/view_video.php?viewkey=ph12345678" \
    --quality best \
    --output "./downloads/" \
    --no-title False
```

### 2. Download a video and use its ID as the filename
Downloads a video but ignores the website title, opting instead to name the file using its unique video ID (e.g., `ph12345678.mp4`).

```bash
phub --download "https://www.pornhub.com/view_video.php?viewkey=ph12345678" \
    --quality best \
    --output "./downloads/" \
    --no-title True \
    --id-as-title
```

### 3. Download the first 5 videos from a playlist
Iterates through a given playlist and stops after successfully downloading 5 videos.

```bash
phub --download "https://www.pornhub.com/playlists/54321" \
    --quality best \
    --output "./playlist_downloads/" \
    --no-title False \
    --limit 5
```

### 4. Download your liked videos
Logs into your account and downloads the first 10 videos you've liked/favorited.

```bash
phub --email "your_email@example.com" \
    --password "your_password" \
    --liked \
    --quality best \
    --output "./my_likes/" \
    --no-title False \
    --limit 10
```

### 5. Download from a list of URLs
Reads a text file (`urls.txt`) containing various links and downloads all of them in half-quality.

```bash
phub --file "urls.txt" \
    --quality half \
    --output "./bulk_downloads/" \
    --no-title False
```
