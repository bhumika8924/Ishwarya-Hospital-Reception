# Hospital Reception Monitor - Beginner Documentation

## 1. Project Overview

This project is a video-based hospital reception monitoring system.

The main goal is to take a reception area video and automatically count:

1. How many receptionists are present at the reception desk.
2. How many visitors enter.
3. How many visitors exit.

The project uses:

- Streamlit for the web interface.
- OpenCV for reading and processing video frames.
- YOLOv8 for detecting people in the video.
- Tracking IDs to follow the same person across multiple frames.
- JSON configuration files to remember the reception desk area and visitor crossing lines.

There are two receptionist detection modes:

1. Pink/reference-uniform mode.
2. Blue-saree mode.

Both modes also count visitor entry and exit using the same two-line crossing logic.

## 2. Problem Statement

In a hospital reception area, management may want to understand:

- Whether enough receptionists are available at the desk.
- How many visitors are coming in.
- How many visitors are going out.
- Whether the reception area becomes crowded during certain periods.

Doing this manually from CCTV footage is difficult because:

- A person has to watch the full video.
- Counting can become inaccurate when many people move at the same time.
- Receptionists and visitors can be confused with each other.
- People can move in both directions, so entry and exit need to be separated.
- The camera angle and reception desk area are fixed, but the app still needs to know where that area is.

So the project solves this by automatically analyzing the video.

## 3. What I Built

I built a Streamlit application that allows the user to upload a video and run counters.

The application:

1. Loads a YOLOv8 model.
2. Reads the uploaded video frame by frame.
3. Detects people in every frame.
4. Tracks each person using an ID.
5. Checks whether a person is inside the configured reception zone.
6. Checks whether the person looks like a receptionist.
7. Counts confirmed receptionists at the desk.
8. Tracks visitors crossing two configured lines.
9. Counts entry when a visitor crosses Line 1 then Line 2.
10. Counts exit when a visitor crosses Line 2 then Line 1.
11. Shows progress and preview frames during processing.
12. Displays final metrics after analysis.

## 4. Why There Are Two Apps

The project has two separate Streamlit apps because receptionist uniform detection is different in two cases.

### Pink/reference-uniform app

File:

```text
apps/streamlit_app.py
```

Launcher:

```text
streamlit_app.py
```

This mode uses a reference image:

```text
assets/receptionist_uniform_ref.png
```

The app compares the detected person's clothing color with the reference image. It also checks for red or pink lanyard/id-card color near the neck area.

### Blue-saree app

File:

```text
apps/streamlit_app_blue.py
```

Launcher:

```text
streamlit_app_blue.py
```

This mode does not use a reference image. Instead, it directly checks whether enough blue color is present in the detected person's body area.

This was added because a blue saree uniform needs a different detection rule from the pink/reference uniform.

## 5. Folder Structure

```text
Ishwaraya_Hospital_Reception-main/
  apps/
    streamlit_app.py
    streamlit_app_blue.py

  core/
    counter_core.py
    counter_core_blue.py

  tools/
    two_line_visitor_counter.py

  config/
    reception_zone.json
    two_line_counter_lines.json

  assets/
    yolov8n.pt
    receptionist_uniform_ref.png

  data/
    videos/

  docs/
    use_case.md
    project_documentation_for_beginners.md

  .streamlit/
    config.toml

  requirements.txt
  README.md
  streamlit_app.py
  streamlit_app_blue.py
```

## 6. Important Files Explained

### `requirements.txt`

This file lists Python packages required by the project:

```text
streamlit
ultralytics
opencv-python
numpy
supervision
```

These libraries are needed for UI, person detection, video processing, and numerical calculations.

### `assets/yolov8n.pt`

This is the YOLOv8 model file.

YOLO is used to detect people in each video frame. In YOLO, class `0` means `person`, so the code only counts detections where the class is `0`.

### `assets/receptionist_uniform_ref.png`

This is the reference uniform image for the pink/reference-uniform app.

The app creates a color histogram from this image and compares it with the clothing area of detected people.

### `config/reception_zone.json`

This file stores four points that define the reception desk area.

Current format:

```json
[
  [549, 57],
  [1098, 77],
  [1094, 1022],
  [524, 1008]
]
```

The four points create a polygon. If a detected person overlaps enough with this polygon, the app treats that person as being inside the reception desk zone.

### `config/two_line_counter_lines.json`

This file stores the two visitor counting lines.

Current format:

```json
{
  "reference_width": 1280,
  "reference_height": 720,
  "lines": [
    [[791, 187], [1106, 211]],
    [[846, 353], [1278, 354]]
  ]
}
```

Line 1 and Line 2 are used to decide entry and exit direction.

### `tools/two_line_visitor_counter.py`

This tool lets the user draw or redraw Line 1 and Line 2 on a video frame.

It saves those lines into:

```text
config/two_line_counter_lines.json
```

### `.streamlit/config.toml`

This file increases the Streamlit upload limit:

```toml
[server]
maxUploadSize = 1024
```

This allows the app to accept large video uploads up to around 1024 MB.

## 7. How To Run The Project

### Step 1: Install dependencies

Open terminal in the project folder and run:

```bash
pip install -r requirements.txt
```

### Step 2: Make sure YOLO model exists

The model file should be here:

```text
assets/yolov8n.pt
```

If this file is missing, the app will stop and show an error.

### Step 3: Run pink/reference-uniform app

```bash
streamlit run streamlit_app.py
```

### Step 4: Run blue-saree app

```bash
streamlit run streamlit_app_blue.py
```

### Step 5: Open the app

Streamlit usually opens this URL:

```text
http://localhost:8501
```

### Step 6: Upload video

Upload a reception video in one of these formats:

```text
mp4, avi, mov, mkv
```

### Step 7: Click Run counters

The app processes the video and shows:

- Frames processed.
- Peak receptionists at desk.
- Confirmed receptionist IDs.
- Visitor entries.
- Visitor exits.

## 8. How The Main App Works

The Streamlit app handles the user interface.

Main responsibilities:

1. Show title and description.
2. Ask user to upload video.
3. Check whether required files exist.
4. Save uploaded video temporarily.
5. Call the core analysis function.
6. Show progress while frames are processed.
7. Show preview frames with overlays.
8. Show final result metrics.
9. Delete the temporary uploaded video file after processing.

The main analysis call for the pink/reference app is:

```python
result = analyze_video(
    video_path=video_path,
    model_path=MODEL_PATH,
    reference_path=REFERENCE_PATH,
    zone_path=ZONE_PATH,
    two_line_path=two_line_path,
    ref_w=ref_w,
    ref_h=ref_h,
    entry_order_option=entry_order_option,
    yolo_conf=yolo_conf,
    yolo_iou=yolo_iou,
    match_threshold=match_threshold,
    min_red=min_red,
    min_overlap=min_overlap,
    analyze_every=analyze_every,
    on_progress=update_progress,
    on_frame=update_frame,
    preview_every=5,
)
```

The blue-saree app calls a similar function but uses `min_blue_pixels` instead of `min_red`.

## 9. How Person Detection Works

The project uses YOLOv8 from the `ultralytics` package.

In each frame:

1. YOLO detects objects.
2. The code checks only person detections.
3. The code ignores very small boxes because small detections can be false or unclear.
4. YOLO tracking gives each detected person a stable `track_id`.

Tracking is important because the same person appears in many frames. Without tracking, the app might count the same person again and again.

## 10. How Receptionist Counting Works

Receptionist counting uses two checks:

1. Is the person inside the reception zone?
2. Does the person match the receptionist uniform rule?

### Reception zone check

The reception desk area is stored in:

```text
config/reception_zone.json
```

The code checks how much the person's bounding box overlaps with this zone.

If the overlap is at least `30%`, the person is considered to be inside the reception zone.

This value comes from:

```python
min_overlap = 0.30
```

### Pink/reference-uniform check

For the pink/reference app:

1. The app reads `assets/receptionist_uniform_ref.png`.
2. It creates a color histogram from the reference image.
3. For every detected person, it crops the torso area.
4. It creates a color histogram for that torso.
5. It compares the person's histogram with the reference histogram.
6. If the score is high enough, it treats the person as a uniform match.

The default threshold is:

```python
match_threshold = 0.55
```

The app also checks the neck area for red or pink lanyard/id-card pixels.

Default minimum red/pink pixels:

```python
min_red = 20
```

So in the pink/reference app, a person can be confirmed as receptionist if:

- The person is inside the reception zone, and
- The uniform color matches or lanyard/id-card color is detected.

### Blue-saree check

For the blue-saree app:

1. The app crops the person's body area.
2. It converts the crop to HSV color format.
3. It creates a mask for blue color.
4. It checks blue color in torso and lower-body regions.
5. If enough blue is found, it treats the person as a blue-saree receptionist.

Default values:

```python
match_threshold = 0.18
min_blue_pixels = 80
```

### Confirmation over multiple frames

The app does not confirm a receptionist from only one frame.

It waits until the same tracked person matches for multiple frames:

```python
CONFIRM_FRAMES_NEEDED = 3
```

This reduces wrong counts caused by a single bad frame.

## 11. How Visitor Entry And Exit Counting Works

Visitor counting uses two lines.

The lines are stored in:

```text
config/two_line_counter_lines.json
```

Default direction:

```python
entry_order_option = "1-2"
```

This means:

- Line 1 then Line 2 means entry.
- Line 2 then Line 1 means exit.

### Visitor counting steps

For each non-receptionist person:

1. Find the center point of the person's bounding box.
2. Remember the previous center point.
3. Check whether movement crossed Line 1 or Line 2.
4. Store the last two crossed lines.
5. If the sequence is `[1, 2]`, count one entry.
6. If the sequence is `[2, 1]`, count one exit.

### Why the code uses cooldown

If a person stands near a line, the center point can touch the same line many times.

To avoid repeated counting, the code uses:

```python
LINE_COOLDOWN_FRAMES = 12
```

This means the same person cannot trigger another line hit immediately.

### Why the code stores person signature

Sometimes YOLO tracking can lose a person and assign a new ID.

To handle this, the code stores a simple color histogram signature for each line event. If a later crossing looks visually similar and happens soon enough, the app can match the two crossings and still count entry or exit.

This helps when tracking IDs are not perfectly stable.

## 12. Final Output Metrics

After analysis, the app shows five metrics:

### Frames processed

Total number of video frames read by the app.

### Peak receptionists at desk

The maximum number of confirmed receptionists visible in the reception zone at the same time.

### Confirmed receptionist IDs

Total number of unique tracked people confirmed as receptionists.

### Visitor entries

Number of non-receptionist people who crossed Line 1 then Line 2.

### Visitor exits

Number of non-receptionist people who crossed Line 2 then Line 1.

## 13. Calibration

Calibration means adjusting the app to match the camera view.

### Reception zone calibration

If the reception desk polygon is not aligned, edit:

```text
config/reception_zone.json
```

It should contain four points around the reception desk area.

### Visitor line calibration

If visitor entry/exit lines are not aligned, run:

```bash
python tools/two_line_visitor_counter.py --redraw
```

The tool opens a video frame and lets the user draw Line 1 and Line 2.

Controls:

- Drag mouse to draw a line.
- Draw Line 1 first.
- Draw Line 2 second.
- Press `S` to save.
- Press `R` to reset.
- Press `ENTER` to start.
- Press `Q` to quit.

### Uniform reference calibration

For the pink/reference app, replace this image if the uniform changes:

```text
assets/receptionist_uniform_ref.png
```

For the blue-saree app, the blue HSV values are hardcoded in:

```text
core/counter_core_blue.py
```

If the shade of blue changes a lot, those values may need adjustment.

## 14. Main Logic Flow

```text
User uploads video
        |
        v
Streamlit saves video temporarily
        |
        v
YOLO detects people frame by frame
        |
        v
Each person gets a tracking ID
        |
        v
Check if person is inside reception zone
        |
        v
Check receptionist uniform rule
        |
        v
Confirm receptionist after multiple matching frames
        |
        v
For non-receptionists, check Line 1 / Line 2 crossing
        |
        v
Count entry or exit based on crossing order
        |
        v
Show final metrics in Streamlit
```

## 15. What Problems Were Solved

### Problem 1: Manual counting is slow

Manual counting requires someone to watch the full video. The app automates this by processing frames and showing final metrics.

### Problem 2: Receptionists and visitors can be confused

The app separates receptionists from visitors using:

- Reception zone location.
- Uniform color.
- Lanyard/id-card color in pink/reference mode.
- Blue-saree color in blue mode.
- Multi-frame confirmation.

### Problem 3: Same person may appear in many frames

The app uses YOLO tracking IDs so the same person is not treated as a new person in every frame.

### Problem 4: Entry and exit directions are different

The app uses two lines instead of one line.

This gives direction:

- Line 1 then Line 2 means entry.
- Line 2 then Line 1 means exit.

### Problem 5: Camera/video resolution may change

The app stores line coordinates with reference width and height, then scales them to the current video frame size.

This helps the same configuration work with videos of different resolutions.

## 16. Current Limitations

This project works based on computer vision rules, so results can change depending on video quality.

Possible limitations:

- Very crowded scenes may reduce tracking accuracy.
- Poor lighting can affect color detection.
- Similar-colored visitor clothing can confuse receptionist detection.
- If the reception zone is not calibrated correctly, receptionist counts may be wrong.
- If visitor lines are not placed correctly, entry/exit counts may be wrong.
- If YOLO loses a person for too long, tracking may become imperfect.
- Large videos can take time to process.

## 17. How To Improve In Future

Possible improvements:

1. Add a UI to draw the reception zone instead of editing JSON manually.
2. Add downloadable CSV reports.
3. Add timestamps for every entry and exit event.
4. Add chart for visitor count over time.
5. Add peak-hour calculation.
6. Add waiting-area chair count.
7. Add doctor cabin presence/absence detection.
8. Add better uniform training using custom model data.
9. Add live CCTV stream support.
10. Add tests for helper functions such as line crossing and zone overlap.

## 18. Simple Explanation For A Beginner

Think of the video as a list of pictures.

The app checks one picture at a time.

In each picture:

1. YOLO finds all people.
2. The tracker gives each person an ID.
3. The app checks who is standing near the reception desk.
4. If that person has the receptionist uniform, the app counts them as receptionist.
5. Other people are treated as visitors.
6. If a visitor crosses the first line and then the second line, it counts as entry.
7. If a visitor crosses the second line and then the first line, it counts as exit.

At the end, the app gives the final numbers.

## 19. Commands Summary

Install dependencies:

```bash
pip install -r requirements.txt
```

Run pink/reference-uniform app:

```bash
streamlit run streamlit_app.py
```

Run blue-saree app:

```bash
streamlit run streamlit_app_blue.py
```

Redraw visitor counting lines:

```bash
python tools/two_line_visitor_counter.py --redraw
```

Run line tool on a specific video:

```bash
python tools/two_line_visitor_counter.py --video data/videos/vdo2.mp4 --redraw
```

## 20. Conclusion

This project converts hospital reception CCTV/video analysis into an automated workflow.

Instead of manually watching a video and counting people, the user can upload a video and get:

- Receptionist count.
- Visitor entry count.
- Visitor exit count.
- Processed frame count.

The solution uses YOLO for person detection, tracking IDs for following people, reception-zone configuration for desk area detection, uniform/color logic for receptionist identification, and two-line crossing logic for visitor direction counting.

