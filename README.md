# Hospital Reception Monitor

Streamlit and OpenCV tools for monitoring a hospital reception desk. The project
detects people with YOLO, counts confirmed receptionists in the configured
reception zone, and counts visitor entries/exits using two saved crossing lines.

There are two separate Streamlit modes:

- Pink/reference-uniform receptionist counter
- Blue-saree receptionist counter

Both modes use the same reception-zone file and visitor entry/exit line setup.

## Folder Structure

```text
apps/                           <- Streamlit app screens
  streamlit_app.py              <- Pink/reference-uniform receptionist counter
  streamlit_app_blue.py         <- Blue-saree receptionist counter

core/                           <- Main counting logic
  counter_core.py               <- Pink/reference-uniform analysis logic
  counter_core_blue.py          <- Blue-saree analysis logic

tools/                          <- Setup/helper scripts
  two_line_visitor_counter.py   <- Draw/redraw visitor entry and exit lines

config/                         <- Camera calibration settings
  reception_zone.json           <- Four-point reception desk zone
  two_line_counter_lines.json   <- Saved Line 1 and Line 2 setup

assets/                         <- Model and image assets
  yolov8n.pt                    <- YOLOv8 person-detection model (local file)
  receptionist_uniform_ref.png  <- Pink/reference uniform image

data/videos/                    <- Sample/test videos
docs/                           <- Notes and use cases
.streamlit/config.toml          <- Streamlit upload limit
```

The root `streamlit_app.py` and `streamlit_app_blue.py` files are small launchers,
so the old run commands still work after organizing the folders.

## Local Setup

```bash
pip install -r requirements.txt
```

Place the YOLO model file at:

```text
assets/yolov8n.pt
```

This model file is ignored by git, so it must be added locally after cloning.

Run the pink/reference-uniform version:

```bash
streamlit run streamlit_app.py
```

Run the blue-saree version:

```bash
streamlit run streamlit_app_blue.py
```

The app opens at:

```text
http://localhost:8501
```

If one Streamlit app is already running on port `8501`, stop it before starting
the other mode, or run the second app on a different port:

```bash
streamlit run streamlit_app_blue.py --server.port 8502
```

Draw or redraw the visitor entry/exit lines:

```bash
python tools/two_line_visitor_counter.py --redraw
```

## How It Works

- YOLO detects people in each video frame.
- Tracking keeps a stable ID for each detected person.
- `config/reception_zone.json` decides the reception desk area.
- The pink app checks the reference uniform image and lanyard color.
- The blue app checks for blue-saree color in the body area.
- A person must match for multiple frames before being confirmed as receptionist.
- Visitor entries/exits are counted when non-receptionist people cross the two saved lines.
- The app reports peak receptionist count, confirmed receptionist IDs, visitor entries,
  visitor exits, and processed frame count after analysis.

## Main Metrics

- Peak receptionists at desk
- Confirmed receptionist IDs
- Visitor entries
- Visitor exits
- Frames processed

## Calibration Files

- Edit `config/reception_zone.json` when the yellow reception zone is not aligned.
- Run `python tools/two_line_visitor_counter.py --redraw` when the visitor lines are not aligned.
- Replace `assets/receptionist_uniform_ref.png` when the pink/reference uniform changes.

## GitHub Notes

The repository is set up to avoid committing local runtime files and large video/model
assets:

- `.venv/`, `venv/`, `__pycache__/`, and `*.pyc` are ignored.
- `*.mp4` sample videos are ignored.
- `assets/yolov8n.pt` is ignored and should be downloaded or copied locally.
