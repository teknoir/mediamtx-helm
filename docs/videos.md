# ✶ Adding Videos & Streams

!!! abstract "At a glance"
    Ship your own video files into MediaMTX using a **video container image** as a Helm **init container**, then start looping **FFmpeg** streams with a few `commands`.

    | Concept | Where |
    |---------|-------|
    | Video files | Baked into a container image |
    | Delivery | `initContainers` rsync → shared `/videos` volume |
    | Streaming | FFmpeg `commands` in `values.yaml` |
    | Host path | `videoPath` (default `/opt/teknoir/mediamtx/videos`) |

---

## ✦ How it works

```mermaid
graph LR
  VID[⬢ video image] -->|init: rsync| VOL[❒ /videos shared volume]
  VOL --> MTX[▲ mediamtx]
  MTX -->|ffmpeg loop| RTSP[rtsp://.../stream]
```

1. **Video image** — a small container that carries your `.mp4` files and copies them into the shared `/videos` volume on startup.
2. **Init container** — the Helm chart runs each entry of `videos:` as an `initContainer` that mounts `videos-volume` at `/videos`.
3. **Stream commands** — once MediaMTX is up, the chart runs your FFmpeg `commands` to loop each file into a live RTSP path.

---

## ⬢ 1 · Build a video container image

The image only needs to **copy its videos into `/videos`** on start. A minimal `alpine` + `rsync` image does the job:

```dockerfile
FROM alpine:3

RUN apk add --no-cache rsync

# Bundle your video files into the image
COPY ./vid-tmp/ /video/my-demo/

# On start, sync them into the shared /videos volume
CMD ["sh", "-c", "mkdir -p /videos/my-demo/ && \
  rsync --checksum --itemize-changes --delete-before -avhr /video/my-demo/ /videos/my-demo/"]
```

Build & push it to your registry:

```bash
docker buildx build --platform=linux/amd64 --push \
  --tag ghcr.io/teknoir/my-demo-mediamtx:latest .
```

!!! tip "Use a per-image subfolder"
    Copy into a **named subfolder** (e.g. `/videos/my-demo/`). This keeps files from multiple video images from colliding in the shared volume.

---

## ❈ 2 · Add the videos to MediaMTX

Reference the image under `videos:` in the Helm values. Each entry becomes an init container and its `commands` start the streams:

```yaml
---
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: mediamtx
  namespace: default
spec:
  repo: https://teknoir.github.io/mediamtx-helm
  chart: mediamtx
  targetNamespace: default
  valuesContent: |-
    nodePort:
      enabled: true
    videos:
      - name: my-demo
        image: ghcr.io/teknoir/my-demo-mediamtx:latest
        commands:
          - ffmpeg -re -stream_loop -1 -i '/videos/my-demo/clip1.mp4' -r 15 -c copy -f rtsp rtsp://localhost:8554/clip1
          - ffmpeg -re -stream_loop -1 -i '/videos/my-demo/clip2.mp4' -r 15 -c copy -f rtsp rtsp://localhost:8554/clip2
```

| Field | Purpose |
|-------|---------|
| `videos[].name` | Init container name & suggested `/videos` subfolder |
| `videos[].image` | Video container image to pull |
| `videos[].commands` | FFmpeg commands run after the image files are synced |

---

## ▶ 3 · Start streaming the video files

Each stream is an FFmpeg command that **loops a file forever** and republishes it as an RTSP path on MediaMTX:

```bash
ffmpeg -re -stream_loop -1 -i '/videos/my-demo/clip1.mp4' -r 15 -c copy -f rtsp rtsp://localhost:8554/clip1
```

| Flag | Meaning |
|------|---------|
| `-re` | Read input at native (real-time) frame rate |
| `-stream_loop -1` | Loop the file endlessly |
| `-i '/videos/.../clip1.mp4'` | Input file from the shared volume |
| `-r 15` | Output frame rate (fps) |
| `-c copy` | Copy codecs (no re-encode — fast, low CPU) |
| `-f rtsp rtsp://localhost:8554/clip1` | Publish to MediaMTX path `clip1` |

The stream is then playable at:

```
rtsp://<device-ip>:8554/clip1
```

!!! note "Global vs per-video commands"
    You can put FFmpeg lines either under a `videos[].commands` block (grouped with its image) or under a top-level `commands:` list for streams that don't belong to a specific video image. Both are started once MediaMTX is running.

!!! tip "Re-encode when copy fails"
    If `-c copy` produces errors (unsupported codec/format for RTSP), transcode instead, e.g. replace `-c copy` with `-c:v libx264 -preset veryfast -tune zerolatency`.
