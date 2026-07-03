# ❈ MediaMTX Helm Chart

!!! abstract "At a glance"
    The **Teknoir MediaMTX HelmChart** — deploy MediaMTX to any Kubernetes device in one command.

    | Field | Value |
    |-------|-------|
    | Component | `mediamtx-helm` |
    | Chart | `mediamtx` |
    | Version | `0.0.4` |
    | App version | `1.12.2` |
    | Registry | `teknoir.github.io/mediamtx-helm` |

---

## ⬇ Add & install

```bash
helm repo add teknoir-mediamtx https://teknoir.github.io/mediamtx-helm/
helm install mediamtx teknoir-mediamtx/mediamtx -f values.yaml
```

## ⚙ Key values

| Key | Default | Purpose |
|-----|---------|---------|
| `image.repository` | `bluenviron/mediamtx` | Container image |
| `image.tag` | `latest-ffmpeg` | Image tag (FFmpeg-enabled) |
| `videoPath` | `/opt/teknoir/mediamtx/videos` | Host video directory |
| `videos` | `[]` | Videos mounted via host path |
| `commands` | `[]` | Commands run inside the container |
| `nodePort.rtspPort` | `31554` | RTSP node port |
| `nodePort.grpcPort` | `31334` | gRPC node port |

## ✦ Example

```yaml
commands:
  - ffmpeg -re -stream_loop -1 -i /videos/example1.mp4 -r 15 -c copy -f rtsp rtsp://localhost:8554/example1

videos:
  - name: retail-demo
    image: us-docker.pkg.dev/teknoir/gcr.io/retail-demo-mediamtx:latest
```

!!! note
    Depends on `component:teknoir/ghcr.io.teknoir.mediamtx-app`.
