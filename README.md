# Handdesk

A browser-based control center driven by your webcam. Control a mouse, type on a virtual keyboard, draw in the air, measure distances between fingers, play gesture games, and scan your body, face and signs. All tracking runs on your device with [MediaPipe Tasks Vision](https://github.com/google-ai-edge/mediapipe). No video is uploaded.

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole app (HTML, CSS, JavaScript) |
| `bridge.py` | Optional local helper that lets your hand control the real mouse and keyboard |
| `requirements.txt` | Python packages for `bridge.py` |
| `README.md` | This documentation |

## Quick start

Browsers allow the camera only on `localhost` or HTTPS.

```bash
python3 -m http.server 8000
```

Open http://localhost:8000 in Chrome or Edge, click **Start hand control**, and allow the camera. The page needs internet access to load the MediaPipe library and models.

## Tabs

Press keys 1 to 6 to switch tabs, bring both hands together and then pull them apart, or hold a fist for 1.2 s to go to the next one. Switching by tab, key or fist plays one of three transitions (sliding panels, colour burst, spinning tiles) with confetti. Switching by clap uses a different set (shockwave, blinds, zoom flash), so you can tell them apart. All are skipped if your system asks for reduced motion, unless you turn on **Always play transitions** in settings.

| Tab | What it does |
|---|---|
| Mouse | Move, click, drag and scroll with your hand. Includes target practice, a drag puck and a scroll list |
| Keyboard | Virtual keyboard. Point at a key and hold still for 1 s, or pinch, to press |
| Draw | Air drawing with 6 colours, brush size, eraser, undo, clear, PNG export and optional sound |
| Gestures | Illustrated gesture guide that highlights the gesture you are making |
| Arcade | Four timed games, XP, levels and badges |
| Body scan | Pose, face parts, expression, clothing colour, attention estimate and sign to text |

## Gestures

| Gesture | Action |
|---|---|
| Point | Move the cursor |
| Pinch (thumb and index) | Click, drag or draw |
| Index and middle finger up, move up or down | Scroll |
| Open palm, hold 1.2 s | Lock or unlock tracking, with a red circle and a Locked / Unlocked box transition |
| Fist, hold 1.2 s | Next tab |
| Bring both hands together, then pull them apart | Next mode with a random clap transition. A loud deep boom, big red shockwave rings at your hands, a red flash, a strong screen shake and XP happen when the hands meet |
| Hold still on a button for 1 s | Presses it without a pinch (Easy click; a ring fills around the cursor) |

## Finger distance meter

The side panel is visible on every tab. It shows the distance between two fingertips in centimetres and in palm lengths, with a dashed line drawn on the camera view. Choose thumb to index, thumb to middle, index to middle, or index tips of two hands.

**Hold 3 s to capture** saves the current value. It works with a mouse, touch, or a pinch held still over the button.

The value is an estimate. It assumes your palm (wrist to middle knuckle) is about 9.5 cm. Moving your hand toward or away from the camera, or tilting it, changes the result. Expect errors of a few centimetres.

## Two-hand features

- Both hands are tracked and drawn in different colours. The first detected hand moves the cursor.
- The side panel shows the finger count per hand.
- In Draw, pinch with your second hand and move your hands apart or together to change brush size.
- Claps are detected when the palms of both hands come together quickly after being apart.

## Sound

- **Sound drawing** (Draw toolbar): pitch follows the height of your stroke and snaps to a pentatonic scale.
- Sound effects for correct answers, claps and fanfares. Turn them off in settings.

## Arcade and gamification

| Game | Goal |
|---|---|
| Sign challenge (10 s) | Show the requested sign. Streaks add bonus points |
| Finger math (10 s) | Show the answer with the fingers of both hands |
| Clap rush (10 s) | Clap as many times as you can |
| Face challenge (10 s) | Make the requested expression |

Every action earns XP, and the level bar is in the header. There are 12 badges, for example First pinch, Surveyor, Two hands, Air artist, Standing ovation, Full scan, Expressive and Signer. Progress is saved in your browser's local storage under `hd_p`.

## Body scan

Models for face and pose load the first time you open the tab (or start the Face challenge), and only run while you use them, to save CPU.

| Feature | How it works |
|---|---|
| Body parts | 33-point pose skeleton. Chips turn green when head, eyes, nose, ears, mouth, shoulders, elbows, wrists, hands, hips, knees and ankles are visible |
| Face parts | Labelled landmarks for eyes, nose, ears, mouth, chin and forehead, with outlines of lips, eyes and face |
| Expression | Rules over MediaPipe face blendshapes: happy, surprised, angry, sad, wink, tongue out, neutral |
| Attention estimate | Head direction (nose position between the eyes) combined with eye openness |
| Clothing colour | Median colour of a patch of your torso (or below your chin if shoulders are out of view), named (black, white, navy, red, and so on) |
| Hair | Colour of a patch above the forehead. Reports "little or none visible" when it matches your skin tone |
| Face shape | Oval, round, square, heart, diamond or oblong, from landmark ratios, smoothed over about 40 frames. Needs you to face the camera |
| Sign to text | Hold a sign for 0.9 s to add a word to the transcript. The Speak button reads it aloud |

### Sign vocabulary

Yes / good (thumbs up), Peace, OK, Rock on, Call me, Hello / stop (open palm), Fist (A), Look / one (point), I love you, and the letter L.

## What this app cannot do

Please read this before relying on any result.

- **Brain and mind:** a webcam cannot see or read your brain or thoughts. The attention estimate only uses head direction and eye openness.
- **Teeth:** individual teeth are not detected. You can see mouth shape and smile strength only.
- **Hair and face shape:** only a colour and a rough ratio-based guess. Hair style, length and texture are not detected, and hats or backgrounds can fool the colour.
- **Clothing:** only a median colour is reported. Garment type (shirt, jacket) is not detected, and lighting changes the colour.
- **Sign language:** real sign languages use motion, both hands, the face and grammar. This app recognises about ten static hand shapes. It is a game and a demo, not a translator.
- **Expressions:** the labels are simple rules and can be wrong. They are not a measure of how someone feels.
- **Distance:** centimetre values are estimates based on average hand size.
- **System control:** a web page cannot move your computer's mouse by itself. Use `bridge.py` for that.

## System bridge (control the real computer)

```bash
pip install -r requirements.txt
python bridge.py
```

Then switch on **Control system mouse** and/or **Type into system** in the side panel. Your hand moves the real cursor, pinch presses the mouse button, two fingers scroll, and the virtual keyboard types into the focused app. Move the mouse into a screen corner to abort (pyautogui failsafe). The bridge listens on `localhost:8765` only.

## How it works

```
webcam -> MediaPipe HandLandmarker (2 hands, always on)
       -> gesture rules (finger extension, pinch ratio, holds)
       -> virtual cursor -> synthetic mouse events -> any element
       -> optional WebSocket -> bridge.py -> pyautogui -> real mouse and keyboard
webcam -> FaceLandmarker + PoseLandmarker (Body scan tab and Face challenge only)
```

- Pinch is the thumb-index distance divided by hand size, with hysteresis to avoid flicker.
- The cursor uses adaptive smoothing: more smoothing when moving slowly, so clicks do not drift.
- The virtual cursor dispatches `mousedown`, `mousemove`, `mouseup` and `click` events, so ordinary page elements react as if a real mouse were used.

## Tuning

In `index.html`, search for these:

| What | Where |
|---|---|
| Reach and pinch sensitivity | Sliders in the side panel (`S.gain`, `S.pin`) |
| Hold times (lock, next tab) | `hold("palm",1.2,...)` and `hold("fist",1.2,...)` |
| Palm size used for cm | `const HAND=9.5` |
| Clap thresholds | `dd<1.5` and `>2.6` in `extras()` |
| Easy click time | `900` in the `S.dw` block inside `handle()`; switch it off in the side panel |
| Expression rules | `FS.expr=` line in `body()` |
| Face shape thresholds | `sh=` line in `hairShape()` |
| Boom sound | `boom()` |
| Sign rules | `sign()` and `wordOf()` |

## Troubleshooting

- **Camera does not start:** use `http://localhost` or HTTPS and allow the camera.
- **Models do not load:** check your internet connection. Models come from `storage.googleapis.com`, libraries from `cdn.jsdelivr.net`.
- **Low frame rate:** close other tabs, use Chrome or Edge, and stay out of the Body scan tab when you do not need it.
- **Bridge not found:** make sure `python bridge.py` is running before switching the setting on.
- **Signs are misread:** face your palm to the camera, use front lighting, and keep your hand 40 to 60 cm away.

## Privacy

Video stays in your browser. Only the MediaPipe code and model files are downloaded. XP and badges are stored locally. The bridge does not send data anywhere.

## Credits

Built with [MediaPipe Tasks Vision](https://github.com/google-ai-edge/mediapipe), [pyautogui](https://github.com/asweigart/pyautogui) and [websockets](https://github.com/python-websockets/websockets).
