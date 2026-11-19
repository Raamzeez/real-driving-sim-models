# real-driving-sim models

The trained self-driving models that real-driving-sim's **Our model** mode drives with, kept here so the game can load them quickly
from GitHub's servers. Each folder is one model: `model.onnx` (the network, run in the browser with onnxruntime-web), `model.onnx.gz` (the same,
gzipped: what the game downloads, since GitHub Pages sends a .gz as it is) and `model.json` (its settings: picture size, camera, plan
times, the metrics it was chosen on).

- `v0/`: ResNet-18 + GRU, trained on 60 expert drives of 3 minutes on the highway, with nudges. On 6 drives held back: 1.3 cm
  sideways at 1 s, 11 cm at 3 s.
- `v1/`: v0 trained 3 more passes on the same 60 drives, counting each picture more the harder the expert brakes or speeds up
  (1 + 4 for every m/s² past 0.5). On the 6 drives held back, in 2 s clips: it plans −0.78 m/s² where the expert brakes at −1.51
  (v0: −0.35), and lane keeping is as v0's. Driving for minutes it still slows to 10-20 mph after about a minute, as v0 does.
