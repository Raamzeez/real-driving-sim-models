# real-driving-sim models

The trained self-driving models that real-driving-sim's **Our model** mode drives with, kept here so the game can load them quickly
from GitHub's servers. Each folder is one model: `model.onnx` (the network, run in the browser with onnxruntime-web) and `model.json`
(its settings: picture size, camera, plan times, the metrics it was chosen on).

- `v0/`: ResNet-18 + GRU, trained on 60 expert drives of 3 minutes on the highway, with nudges. On 6 drives held back: 1.3 cm
  sideways at 1 s, 11 cm at 3 s.
