---
title: "ffmpeg - how to tell if video is interlaced"
date: 2025-04-22
tags: [ffmpeg, video]
---

```bash
ffmpeg -filter:v idet -frames:v 500 -an -f rawvideo -y /dev/null -i input.mkv
```

Example output:

```
[Parsed_idet_0 @ ...] Repeated Fields: Neither: 480 Top: 15 Bottom: 5
[Parsed_idet_0 @ ...] Single frame detection: TFF: 20 BFF: 10 Progressive: 470 Undetermined: 0
```

Key fields:

- TFF / BFF: Top-/Bottom-field first = interlaced
- Progressive: Most frames are not interlaced
- Undetermined: Couldn't detect

How to interpret:

- If TFF + BFF > 0 → your video is interlaced
- If Progressive ≈ total frames → video is progressive
