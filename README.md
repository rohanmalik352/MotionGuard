# MotionGuard — Pose-Based Violence Detection

A command-line tool that watches a video and points out frames where two people are close together and at least one of them is swinging their arms fast. That's the signal this project uses to guess at possible violent interactions. It runs on top of a pretrained pose-estimation model (YOLOv8-pose from Ultralytics), so there's no custom training and no labeled dataset needed anywhere in the pipeline.

**Author:** Rohan Malik

---

## 1. How It Works

1. **Pose estimation** — Every frame gets passed through `YOLOv8-pose`, which finds each person in the frame and returns 17 body keypoints per person (shoulders, elbows, wrists, hips, knees, and so on).

2. **Tracking** — The model assigns each detected person a persistent ID, so the tool can follow the same person from one frame to the next instead of comparing unrelated detections.

3. **Motion score** — For every tracked person, I measure how far their wrists and elbows moved since the *previous* frame, then divide that by their torso length. That keeps the number consistent whether the person is close to the camera or further away.

4. **Proximity score** — For every pair of people in a frame, I measure how close together they're standing, normalized the same way by body size, so it doesn't matter how far anyone is from the lens.

5. **Flagging** — If two people are close together *and* one of them has a **high** motion score, and that holds for several frames in a row, the tool logs that stretch of video as a possible violent event.

6. **Output** — The tool writes out an annotated video (skeletons drawn on everyone, a red "ALERT" border on flagged frames) plus a JSON file logging exactly when each flagged event started and ended.

For the reasoning behind this approach, the assumptions it's making, and what it's genuinely bad at, see [`REPORT.md`](REPORT.md).

---

## 2. Project Structure

```
MotionGuard/
├── detect_violence.py   # main script — runs the detection pipeline
├── utils.py             # motion score / proximity score helper functions
├── requirements.txt     # Python dependencies
├── README.md
├── REPORT.md
└── sample_output/       # example output lands here after a run
```

---

## 3. Environment Setup

### Prerequisites

- Python 3.9 or newer
- pip
- (Recommended) a virtual environment, so this doesn't mess with your other Python projects

### Step-by-step setup

```bash
# 1. Clone the repository
git clone https://github.com/rohanmalik352/MotionGuard.git
cd MotionGuard

# 2. Create and activate a virtual environment
python3 -m venv venv
source venv/bin/activate        # On Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt
```

The first time you run the script, `ultralytics` will automatically download the pretrained `yolov8n-pose.pt` weights (~7 MB) — no manual download needed, but you'll need an internet connection for that first run.

---

## 4. Running the Project

### Basic usage

```bash
python detect_violence.py --source path/to/input_video.mp4
```

This produces:

- `output/annotated_output.mp4` — the original video with skeletons drawn on every detected person, and a red "ALERT" border on any frame that triggered a flag
- `output/events.json` — a log of the start/end time (in seconds) of every flagged event

### Using a webcam instead of a file

```bash
python detect_violence.py --source 0
```

### Full list of options

```bash
python detect_violence.py \
    --source input.mp4 \
    --output output/annotated_output.mp4 \
    --log output/events.json \
    --motion-threshold 1.4 \
    --proximity-threshold 0.35 \
    --consecutive-frames 5 \
    --conf 0.4
```

| Argument                | Default                       | Meaning                                                                 |
|-------------------------|--------------------------------|--------------------------------------------------------------------------|
| `--source`              | *(required)*                   | Path to the input video, or `0` for webcam                              |
| `--output`              | `output/annotated_output.mp4`  | Where to save the annotated video                                        |
| `--log`                 | `output/events.json`           | Where to save the event log                                              |
| `--model`               | `yolov8n-pose.pt`               | Pretrained pose model to use                                             |
| `--motion-threshold`    | `1.4`                           | Higher = needs faster arm movement to count as "agitated"                |
| `--proximity-threshold` | `0.35`                          | Higher = people have to be closer together to count as "interacting"     |
| `--consecutive-frames`  | `5`                             | How many frames in a row the condition must hold before it's logged      |
| `--conf`                | `0.4`                           | YOLO's confidence threshold for detecting a person at all                |

### Example output log (`events.json`)

```json
{
  "source": "input.mp4",
  "total_frames": 450,
  "fps": 30.0,
  "num_events": 2,
  "events": [
    { "start_time_sec": 4.2, "end_time_sec": 6.1 },
    { "start_time_sec": 11.8, "end_time_sec": 13.0 }
  ]
}
```

---

## 5. Tuning Tips

- **Getting too many false alarms?** Raise `--motion-threshold` and/or `--proximity-threshold`.
- **Missing real events?** Lower those same thresholds, or reduce `--consecutive-frames` — though expect more noise if you do.
- The defaults here were picked by watching test footage, not by tuning against a labeled dataset, so they're a starting point, not a guarantee. Your camera angle, distance, and lighting will all affect what works best — see [`REPORT.md`](REPORT.md) for more on that trade-off.

---

## 6. Notes

- This is a **rule-based heuristic**, not a trained violence classifier. It's meant to be simple and explainable, not maximally accurate.
- It can and will trigger on **non-violent** fast, close-contact motion — hugging, dancing, sports, playful roughhousing. That's a known limitation, not a bug.
- Tested on Python 3.10, across macOS, Linux, and Windows.

---

## Acknowledgments

Built on top of [Ultralytics YOLOv8-pose](https://docs.ultralytics.com/tasks/pose/) for pose estimation and tracking, [OpenCV](https://opencv.org/) for video I/O, and [NumPy](https://numpy.org/) for the underlying math. None of those projects are affiliated with this one — just used here as pretrained/off-the-shelf tools.

## Author

**Rohan Malik**
Course: Computer Vision (CSE3010)
Project: MotionGuard — submitted as an evaluated course project.