# MotionGuard. Pose-Based Violence Detection

MotionGuard is a command-line tool that watches a video and highlights frames where two people stand close together and least one of them swings their arms quickly. This pattern is used by MotionGuard to predict interactions. MotionGuard runs on a pretrained pose-estimation model YOLOv8-pose from Ultralytics so MotionGuard does not require custom training or a labeled dataset.

**Author:** Rohan Malik

---

## 1. How It Works

1. **Pose estimation**. Every frame is processed by YOLOv8-pose. The model finds each person in the frame. Returns 17 keypoints for each person, such as shoulders, elbows, wrists, hips, knees and more.

2. **Tracking**. The model gives every detected person an ID. This allows MotionGuard to follow the person across frames of comparing unrelated detections.

3. **Motion score**. For each tracked person MotionGuard measures how far the wrists and elbows moved since the frame. The distance is divided by the torso length, which keeps the score consistent regardless of how the person's from the camera.

4. **Proximity score**. For every pair of people in a frame MotionGuard measures how close they stand together. The measurement is normalized by body size. The score does not depend on distance from the camera.

5. **Flagging**. When two people are close together. At least one of them has a high motion score for several consecutive frames MotionGuard logs that stretch of video as a possible violent event.

6. **Output**. MotionGuard writes an annotated video that shows skeletons on every person and a red "ALERT" border on flagged frames. MotionGuard also produces a JSON file that logs the start and end times of each flagged event.

For the reasoning behind this approach the assumptions that MotionGuard makes and its limitations see [`REPORT.md`](REPORT.md).

---

## 2. Project Structure

```

MotionGuard/

├── detect_violence.py   # main script. Runs the detection pipeline

├── utils.py             # motion score / proximity score helper functions

├── requirements.txt     # Python dependencies

├── README.md

├── REPORT.md

└── sample_output/       # example output lands after a run

```

---

## 3. Environment Setup

### Prerequisites

- Python 3.9 or newer

- pip

- (Recommended) an environment so this does not mess with Python projects

### Step-by-step setup

```bash

# 1. Clone the repository

git clone https://github.com/rohanmalik352/MotionGuard.git

cd MotionGuard

# 2. Create and activate an environment

python3 -m venv venv

source venv/bin/activate        # On Windows:

# 3. Install dependencies

pip install -r requirements.txt

```

The time you run the script ultralytics will automatically download the pretrained yolov8n-pose.pt weights (~7 MB). No manual download is required,. Motionguard needs an internet connection for that run.

---

## 4. Running the Project

### usage

```bash

python detect_violence.py --source path/to/input_video.mp4

```

This produces:

- `output/annotated_output.mp4`. The original video with skeletons drawn on every detected person and a red "ALERT" border on any frame that triggered a flag

- `output/events.json`. A log of the start/end time (in seconds) of every flagged event

### Using a webcam or a file

```bash

python detect_violence.py --source 0

```

### list of options

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

| `--source`              | *(required)*                   | Path to the input video or `0` for webcam                              |

| `--output`              | `output/annotated_output.mp4`  | Where to save the annotated video                                        |

| `--log`                 | `output/events.json`           | Where to save the event log                                              |

| `--model`               | `yolov8n-pose.pt`               | Pretrained pose model to use                                             |

| `--motion-threshold`    | `1.4`                           | Higher = needs faster arm movement to count as "agitated"

| `--proximity-threshold` | `0.35`                          | Higher = people have to be closer together to count as "interacting"     |

| `--consecutive-frames`  | `5`                             | How many frames in a row the condition must hold before it is logged      |

| `--conf`                | `0.4`                           | YOLO’s confidence threshold for detecting a person all                |

### Example output log (`events.json`)

```json

{

"source": "input.mp4"

"total_frames": 450

"fps": 30.0

"num_events": 2

"events": [

{ "start_time_sec": 4.2 "end_time_sec": 6.1 }

{ "start_time_sec": 11.8 "end_time_sec": 13.0 }

]

}

```

---

## 5. Tuning Tips

- Getting false alarms? Raise `--motion-threshold` and/or `--proximity-threshold`.

- Missing events? Lower the thresholds. Reduce `--consecutive-frames`.. Expect more noise if you do.

- The defaults here were chosen by watching test footage not by tuning against a labeled dataset. Therefore they are a starting point, not a guarantee. Your camera angle, distance and lighting will all affect what works best. For more on that trade-off see [`REPORT.md`](REPORT.md).

---

## 6. Notes

- This is a rule-based heuristic, not a trained violence classifier. MotionGuard is meant to be simple and explainable not maximally accurate.

- It will trigger on -violent fast close-contact motion such as hugging, dancing, sports or playful roughhousing. That is a known limitation, not a bug.

- MotionGuard was tested on Python 3.10 across macOS, Linux and Windows.

---

## Acknowledgments

Built on top of Ultralytics YOLOv8-pose for pose estimation and tracking OpenCV for video I/O. Numpy for the underlying math. None of those projects are affiliated with MotionGuard. They are simply used here as pretrained, off-the-shelf tools.

## Author

**Rohan Malik** Course: Computer Vision (CSE3010) Project: MotionGuard. Submitted as a course project.