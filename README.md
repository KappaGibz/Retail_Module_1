# Retail Loss Prevention Tracker
 
A real-time shelf monitoring and person tracking system for retail loss prevention, built with OpenCV and MediaPipe. Detects suspicious arm/reach behavior and shelf disturbances — no YOLO or heavy object detection required.
 
---
 
## Features
 
- **Shelf change detection** — MOG2 background subtraction flags pixel-level changes within a drawn shelf ROI
- **Arm dwell detection** — detects an arm held extended and still (concealment gesture pattern)
- **Person tracking overlay** — draws face outline, eyes, arms, and hands using MediaPipe Holistic
- **Composited video recording** — records overlay-embedded AVI clips, auto-stops after 15s with no person
- **CSV alert logging** — timestamped logs for every arm dwell and shelf change event
- **Persistent ROI** — shelf region saved to `settings.json` and reloaded on next run
---
 
## Requirements
 
- Python 3.8+
- OpenCV
- MediaPipe
Install dependencies:
 
```bash
pip install -r requirements.txt
```
 
---
 
## Usage
 
```bash
# Webcam (default)
python Tracker_body_parts_boxes_no_labels_v2_8.py
 
# Video file, no mirror flip
python Tracker_body_parts_boxes_no_labels_v2_8.py --video clip.mp4 --no-flip
 
# Prompt to draw shelf ROI on startup
python Tracker_body_parts_boxes_no_labels_v2_8.py --enable-shelf
 
# Disable CSV alert logging
python Tracker_body_parts_boxes_no_labels_v2_8.py --no-alerts
```
 
### CLI Arguments
 
| Argument | Default | Description |
|---|---|---|
| `--video PATH` | webcam | Path to a video file instead of live camera |
| `--camera INT` | `0` | Webcam index when not using `--video` |
| `--no-flip` | off | Disable horizontal mirror flip (useful for video files) |
| `--enable-shelf` | off | Draw shelf ROI at startup if none is saved |
| `--no-alerts` | off | Disable CSV logging to `logs/` |
 
---
 
## Runtime Keys
 
| Key | Action |
|---|---|
| `Q` | Quit |
| `S` | Draw / redraw shelf ROI |
| `R` | Clear the current shelf change flag |
| `T` | Toggle person tracking and recording on/off |
 
---
 
## File Structure
 
```
├── Tracker_body_parts_boxes_no_labels_v2_8.py  # Main entry point
├── person_tracker.py       # MediaPipe Holistic overlays (face, arms, hands)
├── person_recorder.py      # AVI recording with REC indicator
├── shelf_monitor.py        # MOG2 shelf change detection + ROI management
├── arm_dwell.py            # Arm extension + stillness detection logic
├── alert_logger.py         # CSV alert writer
├── deps.py                 # Runtime dependency checker
├── settings.json           # Persisted shelf ROI across sessions
├── requirements.txt
├── logs/                   # Auto-created; CSV alert files per session
└── recordings/             # Auto-created; AVI clips per recording session
```
 
---
 
## How It Works
 
### Shelf Monitoring
On first run, press `S` to draw a rectangle around the shelf area. This ROI is saved to `settings.json` and reloaded automatically on future runs. Inside the ROI, MOG2 background subtraction continuously models the "empty" shelf state. When pixels deviate significantly (item removed, added, or disturbed), a `shelf_change` alert is logged and a "CHANGE" indicator appears on screen. A picture-in-picture MOG2 mask is shown in the top-right corner.
 
### Arm Dwell Detection
When tracking is active (press `T`), the system measures:
- **Extension ratio** — wrist reach vs. total arm segment length
- **Wrist speed** — normalized to shoulder width
If the arm stays extended (`≥ 0.55`) with low speed (`≤ 0.35`) for more than 1.5 seconds, an `arm_dwell` alert is logged. Alerts repeat every 2 seconds while the condition persists.
 
### Recording
Pressing `T` starts both tracking overlays and video recording simultaneously. The recording includes all drawn overlays (skeleton, face mesh, REC indicator). If no person is detected for 15 consecutive seconds, recording stops automatically.
 
---
 
## Alert Log Format
 
CSV files are written to `logs/alerts_<timestamp>.csv`:
 
```
timestamp_iso, rule, side, metrics, source
2025-01-15T14:23:01.123+00:00, arm_dwell, right, extend=0.78 speed=0.12 hold=3.45s, 0
2025-01-15T14:23:18.456+00:00, shelf_change, —, pixels_changed=1240, 0
```
 
---
 
## Settings
 
`settings.json` stores the last drawn shelf ROI:
 
```json
{
  "shelf_roi": {
    "x": 19,
    "y": 35,
    "w": 617,
    "h": 436
  }
}
```
 
Delete this file or press `S` to redraw the ROI.
