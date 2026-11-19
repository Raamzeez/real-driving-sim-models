# real-driving-sim models

The trained self-driving models that real-driving-sim's **Our model** mode drives with, kept here so the game can load them quickly
from GitHub's servers. Each folder is one model: `model.onnx` (the network, run in the browser with onnxruntime-web), `model.onnx.gz` (the same,
gzipped: what the game downloads, since GitHub Pages sends a .gz as it is) and `model.json` (its settings: picture size, camera, plan
times, the metrics it was chosen on).

From 2026-11-19 each folder also has `vision.onnx` (a picture to the 512 numbers the network sees) and `window.onnx` (the memory run
from blank over the last 40 pictures, as training ran it), each with its .gz; `model.json` gives `window: 40`. The game runs those two:
carried on for minutes, the memory went where training never took it, and the car slowed to 10-20 mph after about a minute.
`model.onnx` is unchanged.

- `v0/`: ResNet-18 + GRU, trained on 60 expert drives of 3 minutes on the highway, with nudges. On 6 drives held back: 1.3 cm
  sideways at 1 s, 11 cm at 3 s.
- `v1/`: v0 trained 3 more passes on the same 60 drives, counting each picture more the harder the expert brakes or speeds up
  (1 + 4 for every m/s² past 0.5). On the 6 drives held back, in 2 s clips: it plans −0.78 m/s² where the expert brakes at −1.51
  (v0: −0.35), and lane keeping is as v0's. Driving for minutes it still slows to 10-20 mph after about a minute, as v0 does.
