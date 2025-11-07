# Motion Detector + Mail Sender

A small Python service that detects motion from a camera, saves a snapshot, and emails the image to a recipient. The project is written to run on a Raspberry Pi with the Pi Camera (using picamera2), but it can be developed and tested on Windows using OpenCV.

## Highlights

- Motion detection using frame differencing (OpenCV).
- On motion, captures an image and sends it by email using SMTP.
- Dockerfile and docker-compose.yml provided for containerized deployment on a device that exposes camera devices.

## Repo layout

- `detektor/motion_detektor.py` — main motion detection script.
- `requirements.txt` — Python dependencies.
- `Dockerfile` — image for running the detector.
- `docker-compose.yaml` — example compose configuration (see notes below).

## Quick summary / contract

- Inputs: environment variables (see next section). If running on Windows, the script expects a webcam accessible via OpenCV.
- Outputs: saved JPEG snapshots (in working directory or mounted `photos` folder) and email sent to the configured recipient.
- Error modes: missing environment variables or missing camera libraries will raise exceptions; see Troubleshooting.

## Required environment variables

The script reads the following environment variables at runtime (no defaults are provided in code). Set them before running.

- `SENDER_EMAIL` — email address used to send (SMTP login).
- `EMAIL_PASSWORD` — password or app password for the sender email account.
- `RECIPIENT_EMAIL` — recipient address.
- `DIFF_THRESHOLD` — integer pixel-count threshold used to decide "motion".
- `CAMERA_WIDTH` — camera capture width (int).
- `CAMERA_HEIGHT` — camera capture height (int).

Example .env (do NOT commit credentials):

```
SENDER_EMAIL=your.email@gmail.com
EMAIL_PASSWORD=your_app_password
RECIPIENT_EMAIL=recipient@example.com
DIFF_THRESHOLD=5000
CAMERA_WIDTH=1280
CAMERA_HEIGHT=720
```

## Dependencies

Primary Python packages are listed in `requirements.txt`. Important notes:

- `opencv-python` — cross-platform. Required for frame processing and Windows webcam access.
- `numpy` — image arrays.
- `picamera2` — Raspberry Pi only (not available on Windows). Leave it in `requirements.txt` for Pi deployments, but do not expect it to install or run on Windows.

Install locally in a virtual environment (Windows example):

```powershell
python -m venv .venv; .\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```


## Running on Raspberry Pi

This project was originally written for Raspberry Pi with the official camera and `picamera2`. On the Pi:

- Use Raspberry Pi OS (Bullseye/Bookworm or newer recommended) and install libcamera and picamera2 per Raspberry Pi documentation.
- Ensure the camera is enabled and working.
- Keep `picamera2` in `requirements.txt` and install dependencies in the Dockerfile or on the Pi.

Build and run with Docker-compose (example):

```powershell
# Build and start
docker compose build
docker compose up -d
```

Note: `docker-compose.yaml` in this repo includes example env settings. Two important corrections:

- Do not include spaces around `=` in `environment` values. For example use `DIFF_THRESHOLD=5000` (not `DIFF_THRESHOLD = 5000`).
- The compose file sets `privileged: true` and devices for Pi camera access; this is intended for an embedded/Linux environment with access to `/dev/vchiq`.

## Using the provided Dockerfile

The `Dockerfile` installs system libs required by OpenCV and Python packages from `requirements.txt`. It then runs `python motion_detektor.py` in `/detektor`.

If you mount a `photos` folder (as the compose file does), images saved by the script will be available on the host.

## Troubleshooting

- ModuleNotFoundError: No module named 'picamera2'
  - This will happen on Windows. Either run on a Raspberry Pi that has picamera2 installed, or modify the script to fall back to OpenCV VideoCapture as shown above.

- SMTP/email failures
  - Ensure `SENDER_EMAIL` and `EMAIL_PASSWORD` are correct. For Gmail, use an App Password if two-factor auth is enabled. Check that the outbound SMTP port (465 used in code) is allowed from your environment.

- Environment variables not set
  - The script uses `os.environ.get(...)` and then casts to int for some vars. If a variable is missing, `int(None)` will raise. Make sure all required env variables are set, or add safe defaults in the code.

## Next steps / Improvements

- Add a Windows-specific script with the OpenCV fallback and make it the default when `picamera2` is unavailable.
- Add a small `.env.example` and update `docker-compose.yaml` to reference an env file.
- Add unit tests for the motion-detection function (feed synthetic frames).
