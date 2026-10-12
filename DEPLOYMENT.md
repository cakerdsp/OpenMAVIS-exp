# Hilti 2022 / M2DGR deployment preparation

Prepared on 2026-10-12. Source checkout and GitHub synchronization preparation
are complete. Linux compilation and replay on the deployment computer have not
been performed; M2DGR adaptation is pending.

- Experiment repository: https://github.com/cakerdsp/OpenMAVIS-exp
- Branch: `codex/hilti22-m2dgr`
- Upstream: https://github.com/MAVIS-SLAM/OpenMAVIS
- Upstream revision: `b13b1c20e84efa4bb63564e26308541af70d03f2`
- Local checkout: `D:\GithubRepo\OpenMAVIS-exp`

## Dataset readiness

| Dataset | Current upstream support | Remaining work |
| --- | --- | --- |
| Hilti 2022 | Official four-fisheye + IMU example, calibration YAML and run instructions. | Convert the bag to the expected EuRoC layout, verify calibration for the selected recording, compile and run a short sequence. |
| M2DGR | No dedicated configuration or data reader in this revision. | Select four cameras with a usable main stereo pair, convert and synchronize images/IMU, derive correctly oriented transforms and camera parameters, then verify initialization and tracking. |

The current multi-camera runner accepts four images, not an arbitrary camera
count. It uses no LiDAR. This is OpenMAVIS, the public reimplementation, rather
than the unreleased original MAVIS implementation.

## Official Hilti example

- YAML: `Examples/Multi-Inertial/HiltiChallenge2022.yaml`
- Runner: `Examples/Multi-Inertial/multi_inertial_euroc.cc`
- Model: `KannalaBrandt8`; images: 720 x 540 at configured 30 Hz.
- Camera slots: camera1 = Hilti cam1, camera2 = cam0, camera3 = cam4,
  camera4 = cam3. IMU input: `mav0/imu0/data.csv`.
- Configured IMU frequency: 400 Hz; ORB feature setting: 500 per image.
- Input images share filenames from the timestamp list. Check associations
  before running; the current example does not resolve asynchronous filenames.

After a successful Linux build and verified dataset conversion:

```bash
./Examples/Multi-Inertial/multi_inertial_euroc \
  Vocabulary/ORBvoc.txt \
  Examples/Multi-Inertial/HiltiChallenge2022.yaml \
  /path/to/euroc_sequence /path/to/timestamps.txt
```

Initialization remains a separate concern. `Tracking.cc` contains a default
acceleration-change gate (`mFastInit` defaults to false in its legacy parameter
reader); the sample YAML does not request `IMU.fastInit`. The local-mapping
insufficient-motion reset is commented out, but multi-keyframe inertial
initialization is still present. Verify the active parameter-loading path and
initialization behavior before changing any option or claiming low-excitation
support.

Build dependencies include Eigen, Pangolin and OpenCV. The checked-in
`CMakeLists.txt` requests OpenCV 4.4; inspect the target computer before deciding
how to satisfy or adjust that requirement. Bundled DBoW2, g2o and Sophus are
built by the upstream script. Build products in the source repository are not
evidence that this computer can run the system.

## Clone and synchronize

From the desired parent directory on either computer:

```bash
git clone --branch codex/hilti22-m2dgr https://github.com/cakerdsp/OpenMAVIS-exp.git
cd OpenMAVIS-exp
git remote add upstream https://github.com/MAVIS-SLAM/OpenMAVIS.git
git rev-parse HEAD
```

Before updating either checkout, inspect `git status --short`. Commit intended
changes and push this branch; on the other machine use `git pull --ff-only`.
Compare `git rev-parse HEAD` on both computers. Keep dataset exports and run
results outside the checkout.

SSH can provide direct deployment control; GitHub is optional for that access.
For computers on different networks, Tailscale can provide a reachable private
address for ordinary OpenSSH. Login and device authorization must be completed
by the user on both computers. Connection details and destination workspace
are not yet known; no remote dependency installation or SLAM run has occurred.

Sources: [OpenMAVIS](https://github.com/MAVIS-SLAM/OpenMAVIS),
[M2DGR](https://github.com/SJTU-ViSYS/M2DGR),
[Ubuntu OpenSSH](https://ubuntu.com/server/docs/how-to/security/openssh-server/),
[Tailscale setup](https://tailscale.com/docs/how-to/quickstart).
