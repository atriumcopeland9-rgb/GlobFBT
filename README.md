# VR FBT — camera-based full body tracking → VRChat OSC

Single front camera + MediaPipe PoseLandmarker → 8-point body tracking (hip,
chest, both knees, both ankles/feet, both elbows) → sent over OSC to VRChat's
native OSCTrackers input, using this tracker-slot convention:

| Slot | Point       |
|------|-------------|
| 1    | Hip         |
| 2    | Chest       |
| 3    | Left ankle  |
| 4    | Right ankle |
| 5    | Left knee   |
| 6    | Right knee  |
| 7    | Left elbow  |
| 8    | Right elbow |

OSC destination IP/port are editable in-app (defaults to `127.0.0.1:9000`,
VRChat's default OSC-in port) and persist via SharedPreferences.

## Build
See `TERMUX_SETUP.md` for the full from-scratch Termux setup. tl;dr once set up:
```bash
./gradlew assembleDebug
```

## Honest limitations to know going in
- **Rotations are inferred, not measured.** MediaPipe Pose only outputs joint
  *positions*; `Kinematics.kt` derives orientation from bone vectors (thigh
  direction, hip/shoulder line, heel→toe line for feet). This is noisier than
  IMU-based trackers (SlimeVR, Vive/Tundra) and will drift/jitter more,
  especially on axes the camera can't see well (e.g. twist along a limb).
- **Single RGB camera = imperfect depth.** MediaPipe's "world landmarks" give
  metric-scale, hip-relative 3D positions, which is what this app uses, but
  accuracy degrades with occlusion, loose clothing, and poor lighting.
- **Coordinate axes will likely need sign flips.** MediaPipe's landmark space
  and VRChat/Unity's left-handed Y-up space don't line up automatically. If a
  tracked limb looks mirrored or rotated 180° in VRChat, flip the relevant
  axis in `Kinematics.boneRotation()` / `Quat.toEulerDegrees()` — this is a
  one-time calibration you do by eye against a T-pose.
- **No smoothing/filtering yet.** Raw per-frame landmarks are sent as-is.
  Adding a one-euro or exponential-moving-average filter in `onPose()` in
  `MainActivity.kt` is the natural next step if it's too jittery in-game.
- In VRChat, you still need to enable **OSC** and **OSCTrackers** in the
  Action Menu (Options → OSC) and do VRChat's own tracker calibration pose
  the first time trackers appear.

## Where to extend
- `Kinematics.kt` — bone → rotation math, add smoothing here
- `OscSender.kt` — protocol layer, swap in a full OSC library if you outgrow this
- `MainActivity.kt` — swap `slotOrder` if you'd rather map differently, or add
  a calibration/pause button
