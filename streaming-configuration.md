---
title: Streaming Configuration
---

### Stream ingest

Codec : BeLive SDK supports H.264 for video and AAC (LC) for audio

#### Resolution/Bitrate/FPS

BeLive SDK provides following recommended settings for SD and HD quality. API for both SD and HD qualities are readily available.

| Parameter             | Quality (SD) (540x960)   | Quality (HD) (720x1280)  |
| --------------------- | ------------------------ | ------------------------ |
| **Bitrate**           | 1000 Kbps                | 1500 Kbps                |
| **FPS**               | 20                       | 24                       |
| **Keyframe Interval** | Adjustable (2.0 default) | Adjustable (2.0 default) |

### Network Requirements

You must have a stable internet connection that can maintain an adequate, constant upload stream. We strongly suggest to allocate 50% more bandwidth than the minimum required. For SD Stream of 1000 Kbps, at-least 1500 kbps (1.5 Mbps) upload speed is required because overhead is added to compensate for the bitrate fluctuations.