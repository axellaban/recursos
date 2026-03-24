# YouTube Transcript Scraping Strategy

## The Core Problem

The npm package `youtube-transcript` fails silently on many videos. **Don't use it.**

## The Solution: `yt-dlp`

Use `yt-dlp` (a CLI tool) to download auto-generated subtitles as `.vtt` files.

```bash
brew install yt-dlp
```

---

## Pipeline Overview

### Step 1 — Get All Video IDs via YouTube Data API v3

Use the channel's **uploads playlist ID** (not the channel ID directly) to paginate through all videos.

```
GET https://www.googleapis.com/youtube/v3/playlistItems
  ?part=snippet,contentDetails
  &playlistId=<UPLOADS_PLAYLIST_ID>
  &maxResults=50
  &key=<YOUTUBE_API_KEY>
  &pageToken=<nextPageToken>  ← paginate until no nextPageToken
```

The uploads playlist ID is derived from the channel ID by replacing `UC` with `UU` at the start.
e.g. channel `UCxVxcTULO9cFU6SB9qVaisQ` → playlist `UUxVxcTULO9cFU6SB9qVaisQ`

---

### Step 2 — Download Subtitles with `yt-dlp`

For each `videoId`, call `yt-dlp` with these flags:

```js
execFileSync("yt-dlp", [
  "--write-auto-sub",   // download auto-generated captions
  "--sub-lang", "en",  // English only
  "--sub-format", "vtt", // WebVTT format
  "--skip-download",   // don't download the video, only the subtitles
  "--no-warnings",
  "-q",
  "-o", `/path/to/vtt/%(id)s`,  // output filename = video ID
  `https://www.youtube.com/watch?v=${videoId}`,
], { timeout: 60000, stdio: "pipe" });
```

Output file will be: `<videoId>.en.vtt`

**Why this works:** `yt-dlp` handles YouTube's anti-bot measures and can pull auto-generated captions even when no manual transcript exists.

---

### Step 3 — Parse the VTT File

VTT files contain timestamped cue blocks. Parse them into `{ start, end, text }` segments.

Key parsing rules:
- Match timestamp lines with regex: `/(\d{1,2}:\d{2}:\d{2}\.\d{3})\s+-->\s+(\d{1,2}:\d{2}:\d{2}\.\d{3})/`
- Strip VTT/HTML tags from text: `/<[^>]+>/g`
- Decode HTML entities (`&amp;`, `&#39;`, etc.)
- **Deduplicate** — VTT files often repeat the same line across multiple cues (rolling captions). Use a `Set` to skip already-seen lines.

---

### Step 4 — Chunk with Overlap

Split the full transcript into overlapping chunks for RAG retrieval:

- **Chunk size**: 500 words
- **Overlap**: 75 words (keep last 75 words of previous chunk as prefix of next)
- **Preserve timestamps**: each chunk stores `start_time`, `end_time`, and a `timestamp_url` (`?t=Xs`) so you can link directly to that moment in the video
- **Skip short transcripts**: if `fullText.length < 100` or `segments.length < 5`, the video probably has no real captions — skip it

---

### Step 5 — Index to Vector DB (Pinecone)

Each chunk becomes a record with these fields:

```json
{
  "_id": "<videoId>_chunk_0",
  "text": "...",
  "title": "Video title",
  "video_id": "abc123",
  "video_url": "https://www.youtube.com/watch?v=abc123",
  "timestamp_url": "https://www.youtube.com/watch?v=abc123&t=142s",
  "start_time": 142.5,
  "end_time": 210.3,
  "start_time_formatted": "2:22",
  "published_at": "2024-01-15T00:00:00Z",
  "channel": "Your Channel Name",
  "chunk_index": 0,
  "total_chunks": 12,
  "source_type": "youtube_transcript"
}
```

Pinecone uses **NDJSON** (newline-delimited JSON) for upserts — one JSON object per line, `Content-Type: application/x-ndjson`. The ID field is `_id` (with underscore).

---

## Resume Safety

Before processing each video, check if a local transcript JSON already exists (`data/youtube/transcripts/<videoId>.json`). If it does, skip it. This means the pipeline is fully resumable — if it crashes halfway, just re-run it.

---

## Key Gotchas

| Issue | Fix |
|-------|-----|
| `youtube-transcript` npm package fails silently | Use `yt-dlp` instead |
| VTT files repeat lines (rolling captions) | Deduplicate with a `Set` |
| Videos with no captions | `yt-dlp` exits cleanly, just returns no `.vtt` file — handle the null case |
| Very short "transcripts" | Skip if < 100 chars or < 5 segments |
| YouTube API quota (10,000 units/day) | Cache the video list to disk; only re-fetch when needed |
| `execSync` with string interpolation = security risk | Always use `execFileSync` with an array of arguments |

---

## Required Env Vars

```
YOUTUBE_API_KEY=...   # YouTube Data API v3 key
PINECONE_API_KEY=...  # (if indexing to Pinecone)
```

## Required System Dependency

```bash
brew install yt-dlp   # macOS
# or
pip install yt-dlp    # cross-platform
```
