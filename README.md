# MotionGuard — Pose-Based Violence Detection

So this is MotionGuard. It's a command-line tool — you feed it a video, and it points out frames where two people are standing close and at least one of them is swinging their arms around fast. That's the whole signal, honestly. Two people close + fast arm movement = maybe something's happening. It's built on top of a pretrained pose model (YOLOv8-pose, from Ultralytics), so no custom training, no labeled dataset, none of that.

**Author:** Rohan Malik

---

## 1. How It Works

Broadly, six steps.

1. **Pose estimation.** Every frame runs through YOLOv8-pose. It finds each person and spits out 17 keypoints per person — shoulders, elbows, wrists, hips, knees, etc.

2. **Tracking.** Each person gets an ID so I can follow them frame to frame instead of accidentally comparing two different people.

3. **Motion score.** For each tracked person, I check how far their wrists and elbows moved since the last frame, then divide that by torso length. Doing it this way means the number doesn't change just because someone's closer to or farther from the camera.

4. **Proximity score.** Same idea but for pairs of people — how close are they standing, normalized by body size so distance from the lens doesn't throw it off.

5. **Flagging.** Two people close together, one with a high motion score, held for a few frames in a row — that gets logged as a possible violent event.

6. **Output.** You get an annotated video (skeletons drawn on, red "ALERT" border when something's flagged) and a JSON file with the timestamps.

More on why I built it this way, what it assumes, and where it breaks down — that's in [`REPORT.md`](REPORT.md).

---

## 2. Project Structure

```
MotionGuard/
├── detect_violence.py   # main script — runs everything
├── utils.py             # motion/proximity score helpers
├── requirements.txt
├── README.md
├── REPORT.md
└── sample_output/       # shows up after you run it once
```

---

## 3. Environment Setup

### You'll need

- Python 3.9+
- pip
- A virtual env, probably, so this doesn't mess with anything else on your machine

### Setup

```bash
# clone it
git clone https://github.com/rohanmalik352/MotionGuard.git
cd MotionGuard

# venv
python3 -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# install stuff
pip install -r requirements.txt
```

First run, ultralytics grabs the yolov8n-pose.pt weights on its own (~7 MB). Nothing you need to do manually, just be online for that first run.

---

## 4. Running It

```bash
python detect_violence.py --source path/to/input_video.mp4
```

You'll get:

- `output/annotated_output.mp4` — skeletons on everyone, red border on flagged frames
- `output/events.json` — when each flagged event started and ended

### Webcam works too

```bash
python detect_violence.py --source 0
```

### Every option

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
| `--source`              | *(required)*                   | Video path, or `0` for webcam                                           |
| `--output`              | `output/annotated_output.mp4`  | Annotated video save location                                            |
| `--log`                 | `output/events.json`           | Event log save location                                                  |
| `--model`               | `yolov8n-pose.pt`               | Which pretrained pose model to use                                       |
| `--motion-threshold`    | `1.4`                           | Higher = needs faster movement to count as "agitated"                    |
| `--proximity-threshold` | `0.35`                          | Higher = people need to be closer to count as "interacting"              |
| `--consecutive-frames`  | `5`                             | How many frames in a row before it logs                                  |
| `--conf`                | `0.4`                           | YOLO's confidence cutoff for detecting a person                          |

### What the log looks like

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

## 5. Tuning

Too many false alarms? Bump up `--motion-threshold` or `--proximity-threshold`, or both. Missing real events instead? Go the other way — lower those, or drop `--consecutive-frames`. Just know that'll bring more noise with it.

Honestly, I picked these defaults by eyeballing test footage, not by tuning against any labeled data, so don't treat them as gospel. Your camera angle, how far away it is, the lighting — all of that changes what actually works. More on this trade-off in [`REPORT.md`](REPORT.md).

---

## 6. Notes

- This isn't a trained violence classifier. It's a rule-based heuristic — simple, explainable, not super accurate. That's on purpose.
- It'll flag stuff that isn't violent too. Hugging, dancing, sports, roughhousing with your friends — all of that can trigger it. That's a real limitation, not something I overlooked.
- I tested it on Python 3.10, on macOS, Linux, and Windows.

---

## Acknowledgments

Runs on Ultralytics YOLOv8-pose for pose estimation and tracking, OpenCV for video I/O, NumPy for the math. None of those projects have anything to do with mine — just tools I used off the shelf.

## Author

**Rohan Malik**
Computer Vision (CSE3010)
MotionGuard — course project.