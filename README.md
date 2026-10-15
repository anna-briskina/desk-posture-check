# Desk Posture Check

Analyse your posture from a video, or live from your webcam while you work, right in your browser.

**[Try it live](https://anna-briskina.github.io/desk-posture-check/)**

## What it does

Desk Posture Check runs a pose-estimation model on your video or webcam and tells you, moment by moment, whether you're sitting well or slouching. It has two modes:

- **Video** (default): upload a recording of yourself at your desk and get a full analysis as it plays.
- **Webcam**: keep the tab open while you work and get a reminder when you slouch, even when the tab is in the background.

Both modes include:

- **Live skeleton overlay** with the measured angles drawn in green, amber or red
- **"Sit up" alerts**, with an optional beep
- **Session report**: a posture score out of 100, the split between good, fair and poor time, your longest slouch, and a list of every alert
- **Timeline** of each metric; hover to read values (in video mode, click to seek)
- **CSV export** of every analysed moment
- **Adjustable thresholds**; changing them re-scores everything analysed so far

Your video and webcam feed are processed on your own device and never uploaded to a server. The model runs locally in the browser (WebAssembly + GPU).

## What it measures

The camera angle is detected automatically, or you can set it yourself under **View**.

| View | Metric | How it's measured | Fair above | Poor above |
|---|---|---|---|---|
| **Side** (recommended) | Head forward | Angle of the ear ahead of the shoulder, from vertical | 25° | 40° |
| | Back lean | Angle of the shoulder ahead of the hip, from vertical | 10° | 20° |
| **Front** (laptop webcam angle) | Head drop | How much the ear-to-shoulder height shrinks compared with your most upright moment | 12% | 22% |
| | Shoulder tilt | One shoulder higher than the other | 4° | 8° |
| | Head tilt | Head leaning toward one shoulder | 6° | 12° |

Leaning back never counts against you. The posture score counts good time fully and fair time as half.

These defaults are sensible starting points, not clinical values. MediaPipe's shoulder point sits slightly in front of the real spine, so an upright "head forward" angle usually reads around 10–20°.

## How to use it

Open the [live page](https://anna-briskina.github.io/desk-posture-check/) in Chrome or Edge.

### Video mode

1. Click **Upload video** or drag a video onto the page.
2. Press play. The analysis runs as the video plays; scrub or skip and it picks up from there.
3. Alerts appear after 5 s of poor posture. Click any alert in the list to jump to that moment.

### Webcam mode

1. Switch to **Webcam** and click **Start webcam**, then allow camera access.
2. Sit up straight for the first few seconds so it learns your upright pose. You can also click **Use this frame as upright** at any time.
3. Allow notifications when asked. If you slouch for 30 s, you get a "Sit up" reminder: as a banner and optional beep when the tab is visible, or as a desktop notification and a changed tab title when it isn't. The reminder repeats every couple of minutes while you stay slouched.
4. Click **Stop webcam** to pause. Your session report is kept, and **Resume webcam** continues the same session.

If you have more than one camera, a **Camera** list appears under the video so you can switch, for example to a phone or external camera placed at your side.

### Camera tips

- **Side view works best.** Put the camera at desk height, 1–2 m to your side, with your ear, shoulder and hip in frame.
- **Front view** works from a laptop webcam. If a video starts while you're slouching, pause on a good frame and click **Use this frame as upright**.
- Good lighting and a plain background help the model track you.

## How it works

1. [MediaPipe Pose Landmarker](https://ai.google.dev/edge/mediapipe/solutions/vision/pose_landmarker) finds 33 body landmarks in each frame.
2. The side of the body facing the camera is chosen by landmark visibility, and the direction you face is taken from the nose.
3. Angles are computed in image space and smoothed with a time-aware moving average.
4. Each metric is graded against its thresholds; the worst one sets the overall status for that moment.
5. In webcam mode the page checks about 15 times a second while visible. Browsers pause animation in background tabs, so a small worker timer keeps checking about once a second while the tab is hidden.

Everything lives in a single `index.html` with no build step. The model files load from Google's CDN the first time the page opens.

## Run it locally

Download `index.html` and open it in Chrome or Edge. That's it: no install and no build step. It needs an internet connection the first time, to load the pose model.

Webcam access and notifications need a secure page. They work on GitHub Pages (https); when opening the file directly, your browser may block them.

## Limitations

- One person at a time.
- Angles come from a 2D view, so a camera placed at an angle (not square to your side or front) skews the numbers.
- Desks and chairs can hide your hips; when they do, back lean is skipped and only head position is scored.
- Background reminders need the tab to stay open. Some browsers slow background tabs further after a long time or in battery-saving modes.
- This is a self-awareness tool, not medical advice.

## Built with

- [MediaPipe Tasks Vision](https://www.npmjs.com/package/@mediapipe/tasks-vision) for pose estimation
- Plain HTML, CSS and JavaScript
