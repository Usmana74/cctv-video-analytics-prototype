# RTSP Detection + Tracking Service (prototype)

Camera (simulated) → RTSP → OpenCV reader → YOLOv8n → ByteTrack → events → SQLite → FastAPI

Built in a few hours as a learning prototype for CCTV-style video analytics.
Not production-hardened.

## What it does
- Reads an RTSP stream (MediaMTX + ffmpeg simulate a 720p/15fps camera)
- Latest-frame-only reader thread: slow inference drops stale frames instead of building lag
- YOLOv8n detection + ByteTrack tracking (person/vehicle classes)
- Writes a `track_started` event per new track ID to SQLite
- FastAPI: `/health` (stream state, last-frame age, status), `/stats` (FPS, latency, GPU memory), `/events`
- Auto-reconnect when the stream drops

## Measured (Colab T4, 1 vCPU, 720p/15fps stream, 30 s run)
| Metric | Value |
|---|---|
| Throughput | 15.0 FPS (kept up with the stream) |
| Detection + tracking latency | p50 15.3 ms, p95 16.9 ms |
| Frame arrival to result | p50 16.4 ms |
| Peak GPU memory | 76.5 MB |
| Isolated model speed (detect / detect+track) | 7.8 ms / 10.5 ms |
| Reconnect test | Detected stream loss in ~1 s; recovered 4 s after camera returned |

## Limitations (honest list)
- Single stream only; multi-camera capacity not tested
- Simulated camera on 1 CPU core; no real IP camera, NVR, ONVIF, or PoE hardware
- Reconnect tested only for a clean stream close, not a frozen/half-open connection
- Track-ID counts are inflated by ID fragmentation and the looping test clip
- No face recognition, LPR, TensorRT, or Docker yet

## Next steps
ONNX/TensorRT export and benchmark, Docker, PostgreSQL, multi-stream test on real cameras.
