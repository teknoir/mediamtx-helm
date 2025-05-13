# MediaMTX Helm Chart

This chart deploys the MediaMTX to a Kubernetes cluster.

> The implementation of the Helm chart is right now the bare minimum to get it to work.

## Usage in Teknoir platform
Use the HelmChart to deploy the MediaMTX to a Device.

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
    # The path to the videos directory
    videoPath: /opt/teknoir/mediamtx/videos
    # Videos to be installed via a shared mounted host path (volume)
    videos: []
```

## Example values.yaml

```yaml
commands:
  - ffmpeg -re -stream_loop -1 -i /videos/example1.mp4 -r 15 -c copy -f rtsp rtsp://localhost:8554/example1
  - ffmpeg -re -stream_loop -1 -i /videos/example2.mp4 -r 15 -c copy -f rtsp rtsp://localhost:8554/example2

videos:
  - name: retail-demo
    image:  us-docker.pkg.dev/teknoir/gcr.io/retail-demo-mediamtx:latest
    commands:
      - ffmpeg -re -stream_loop -1 -i /videos/example3.mp4 -r 15 -c copy -f rtsp rtsp://localhost:8554/example3
      - ffmpeg -re -stream_loop -1 -i /videos/example4.mp4 -r 15 -c copy -f rtsp rtsp://localhost:8554/example4
```

## Adding the repository

```bash
helm repo add teknoir-mediamtx https://teknoir.github.io/mediamtx-helm/
```

## Installing the chart

```bash
helm install mediamtx teknoir-mediamtx/mediamtx -f values.yaml
```