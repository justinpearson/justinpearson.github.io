---
title: "ffmpeg to normalize / hard-limit audio level of mp3"
date: 2025-01-14
tags: [ffmpeg, audio]
---

```bash
ffmpeg -i input.mp3 -af loudnorm=I=-16:TP=-1.5:LRA=11 output.mp3
```
