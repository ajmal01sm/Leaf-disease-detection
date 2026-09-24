# Leaf Disease Detection — ESP32-CAM + Teachable Machine

Three pieces, working together:

```
[ESP32-CAM]  --WiFi-->  [index.html on your PC]  --runs-->  [Teachable Machine model]
 captures              shows live feed,                     classifies the image,
 images only           connects, fetches a                  shows Healthy/Diseased
                        capture, runs the model              + confidence %
```

The ESP32 is "dumb" — it only serves camera images over WiFi. All the AI
inference happens **in your browser**, using TensorFlow.js, when you open
`index.html`. Nothing needs Python, Flask, or a backend server for the model.

---

## Files

- `arduino/esp32cam_leaf_detector.ino` — flash this onto the ESP32-CAM
- `webpage/index.html` — open this in your browser (double-click it, or
  serve it with any local server — both work)

---

## Step 1 — Train and export your model (Teachable Machine)

1. Go to https://teachablemachine.withgoogle.com/ → **Get Started** → **Image Project** → **Standard image model**.
2. Create two classes (or more): e.g. `Healthy` and `Diseased`. Upload/webcam-capture leaf photos for each class — the more varied (lighting, angle, leaf type) the better.
3. Click **Train Model**.
4. Click **Export Model** → tab **Tensorflow.js** → choose:
   - **Upload (shareable link)** — easiest. Click it, Google hosts your model for free, and gives you a URL like:
     `https://teachablemachine.withgoogle.com/models/AbC123XyZ/`
   - *or* **Download my model** — gives you a `.zip` with `model.json`, `metadata.json`, `weights.bin`. Unzip it into a folder (e.g. `my_model/`) placed next to `index.html`.

## Step 2 — Attach the model to the webpage

Open `webpage/index.html` in a text editor and find this block near the top of the `<script>` section:

```js
const MODEL_URL = "https://teachablemachine.withgoogle.com/models/XXXXXXXXX/";
```

- If you used **Upload (shareable link)**: paste your real URL in place of the placeholder (keep the trailing `/`).
- If you used **Download**: change it to a relative path, e.g.:
  ```js
  const MODEL_URL = "./my_model/";
  ```

Also check these two lines just below it, and make sure they match the **exact class names** you typed into Teachable Machine (case-sensitive):

```js
const HEALTHY_LABELS  = ["Healthy", "healthy"];
const DISEASED_LABELS = ["Diseased", "diseased", "Disease", "Unhealthy"];
```

If you trained more classes (e.g. specific diseases like "Leaf Blight", "Powdery Mildew"), just add those exact strings into `DISEASED_LABELS`.

## Step 3 — Flash the ESP32-CAM (Arduino IDE)

Open `arduino/esp32cam_leaf_detector.ino` — full wiring + board-setup instructions are in the comment block at the top of the file. Quick version:

1. Install the ESP32 board package in Arduino IDE (Boards Manager → search "esp32").
2. Select **Board:** "AI Thinker ESP32-CAM", **Partition Scheme:** "Huge APP".
3. Edit these two lines near the top of the sketch with your real WiFi details:
   ```cpp
   const char* WIFI_SSID     = "YOUR_WIFI_NAME";
   const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";
   ```
4. Wire a USB-to-TTL adapter to the board, bridge GPIO0 to GND, press RESET, then click **Upload**.
5. After upload, disconnect GPIO0 from GND, press RESET again.
6. Open **Serial Monitor** (115200 baud) — it will print the board's IP address once connected to WiFi, e.g.:
   ```
   Camera Ready! Use this IP in the webpage:
      192.168.1.45
   ```

## Step 4 — Run it

1. Open `webpage/index.html` in your browser (Chrome/Edge recommended — just double-click the file, no server required).
2. Type the ESP32's IP address (from Serial Monitor) into the **ESP32-CAM IP address** box and click **Connect**. The status dot turns green when it's reachable.
3. Click **Start Live View** to see the camera feed.
4. Point the camera at a leaf, click **Capture & Analyze**.
5. The captured frame appears in the viewport, and within a second the **Diagnosis** panel shows Healthy/Diseased with a confidence percentage. Results are only marked "confident" at ≥92% — below that they show as "Uncertain" so you know to recapture.

---

## Customizing the portfolio text

Every editable section is marked `<!-- EDIT: ... -->` directly in `index.html`:
- Header brand name / nav links
- Hero headline, tagline, byline (your name, college, hardware/model details)
- "About This Build" heading, description paragraph, and the spec list (capture/model/inference/threshold rows)

## Troubleshooting

- **Status stays red / "Disconnected":** Confirm your PC and the ESP32-CAM are on the *same* WiFi network (not different bands/guest networks), and the IP typed matches Serial Monitor exactly.
- **CORS error in browser console:** Already handled — the sketch sends `Access-Control-Allow-Origin: *` on every response. If you still see CORS errors, double check you flashed the latest version of the `.ino`.
- **Camera init failed (Serial Monitor):** Usually means wrong board selected, insufficient power (use a solid 5V/2A supply, not a weak USB port), or a loose camera ribbon cable.
- **Model won't load:** Open the browser console (F12) — if it shows a 404 for `model.json`, your `MODEL_URL` path is wrong, or (if using a downloaded model folder) it isn't sitting next to `index.html`.
