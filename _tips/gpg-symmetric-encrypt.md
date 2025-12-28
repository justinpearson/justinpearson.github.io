---
title: "gpg - quick symmetric encrypt"
date: 2025-01-24
tags: [gpg, encryption]
---

```bash
gpg --symmetric --verbose --verbose --cipher-algo AES256 --output foo.txt.gpg --no-symkey-cache foo.txt
```
