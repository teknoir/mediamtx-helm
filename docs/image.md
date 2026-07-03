# ⬢ MediaMTX Image

!!! abstract "At a glance"
    **MediaMTX real-time media server** container image — the engine core powering the app.

    | Field | Value |
    |-------|-------|
    | Component | `docker.io.bluenviron.mediamtx` |
    | Repository | `docker.io/bluenviron/mediamtx` |
    | Tag | `latest-ffmpeg` |
    | Type | `image` |

---

## ✦ Why `latest-ffmpeg`

The FFmpeg-enabled variant bundles the FFmpeg toolchain, unlocking:

- On-container transcoding & remuxing.
- Looped file playback into live RTSP streams.
- Format conversion across all supported protocols.

## ▶ Pull & run

```bash
docker pull bluenviron/mediamtx:latest-ffmpeg
docker run --rm -it \
  -p 8554:8554 -p 1935:1935 -p 8888:8888 -p 8889:8889 \
  bluenviron/mediamtx:latest-ffmpeg
```

## 🔌 Default ports

| Protocol | Port |
|----------|------|
| RTSP | `8554` |
| RTMP | `1935` |
| HLS | `8888` |
| WebRTC | `8889` |

!!! info "Consumed by"
    This image is deployed by [`mediamtx-helm`](helm.md) and referenced by the [`mediamtx-app`](app.md).
