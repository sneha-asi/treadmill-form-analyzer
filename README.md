# Treadmill Form Analyzer

**An AI running-form coach for the treadmill.** Point a camera at the treadmill (or drop in a video) and AI — [MediaPipe](https://developers.google.com/mediapipe) pose tracking + [Gemini](https://ai.google.dev/) — flags bad form and injury risk while the video plays.

Two years ago, when I started regular cardio, my trainer told me:
> "You know, treadmills are banned in some places because of injuries, but everyone still uses them!"

That stuck with me. Treadmill running is everywhere, but so are the injuries — often from small mistakes nobody notices.

I love running outdoors, but I realized the same form mistakes happen inside, and on a treadmill they're even riskier. Inspired by [Farza's gemini-bball](https://github.com/farzaa/gemini-bball), I thought: *why not use AI to help people spot treadmill mistakes before they get hurt?*

So I built this:

- **Drop in your treadmill video** (or use a live webcam)
- **MediaPipe** tracks your pose and computes joint angles on every frame
- **Gemini** reads each frame + those metrics and flags bad form
- Issues and an **injury-risk level** (low / medium / high) are overlaid on the video and refreshed after each Gemini response

## How it works

`treadmill.py` runs a single loop:

1. Read a frame (webcam `0` or a video file).
2. [MediaPipe Pose](https://developers.google.com/mediapipe) finds body landmarks; helper functions compute knee/elbow angles, lean, and hip alignment.
3. Every couple of seconds, the frame + computed metrics are sent to **Gemini** (`gemini-1.5-flash` by default, configurable via `GEMINI_MODEL`), which returns structured JSON: `overstriding`, `heel_striking`, `leaning_forward_excessively`, `holding_rails`, `poor_arm_swing`, `hip_drop`, plus `overall_feedback` and `risk_level`.
4. Feedback and a colored risk border are drawn on the video. The Gemini call is synchronous, so the video pauses briefly each time a frame is analyzed; this is not true real-time feedback. Press **`q`** to quit.

## Setup

```bash
pip install -r requirements.txt

# get a key from https://aistudio.google.com/app/apikey
export GOOGLE_API_KEY="your-key-here"
```

## Run

```bash
# analyze a video file
python treadmill.py path/to/your_treadmill_video.mp4

# or set it via an environment variable
VIDEO_SOURCE=path/to/your_treadmill_video.mp4 python treadmill.py

# use the webcam (camera index 0)
python treadmill.py 0
```

The command-line argument takes precedence over `VIDEO_SOURCE`; if neither is set, the script falls back to the placeholder `your_treadmill_video.mp4`.

## Notes

- Requires a real `GOOGLE_API_KEY` — the script never stores or hardcodes one.
- `GEMINI_MODEL` sets the Gemini model (default `gemini-1.5-flash`). That model may be retired; if requests fail, set `GEMINI_MODEL` to a current model from [Google's model list](https://ai.google.dev/gemini-api/docs/models).
- The script uses the `google-generativeai` SDK, which Google has deprecated in favour of `google-genai`. It has not been migrated yet.
- Angle thresholds and the Gemini query interval (`GEMINI_QUERY_INTERVAL`) are tunable constants near the top of the file.
- Built as a hacky, practical experiment — contributions and better metrics welcome.

## Credits

Inspired by [farzaa/gemini-bball](https://github.com/farzaa/gemini-bball). MIT licensed; see [LICENSE](LICENSE).
