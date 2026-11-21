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
- `v2/`: v1 trained 3 more passes (counting braking and speeding up as v1 did) on the 60 drives plus braking data: 30 drives in busy and
  heavy traffic with nudges on the pedals (the car pushed up too close behind the car ahead, the expert braking to get its distance
  back) and 180 staged clips (a car stopped in the lane, a slow vehicle, the car ahead braking). Pictures where the expert brakes for a
  car the camera cannot show yet are not trained on. On the 6 drives v1 was checked on: −0.95 m/s² where the expert brakes at −1.51
  (v1: −0.78), no braking where it holds its speed (+0.04, as v1), 11 cm sideways at 3 s (v1: 11.4).
- `v3/`: v2 rewired so that the memory never sees the speed (`model.json` arch `speed-head`: the speed goes in only where the plan is
  made). v2 had learned to brake because the car was already slowing (in every training picture the expert had braked first), not
  because of what it saw: with the speed held at 65 mph its braking on a stopped car went from −1.61 to −0.02 m/s². v3 brakes for what it
  sees, but weakly. Brake test (80 braking runs, same for every model): 26 crashes (v1 31, v2 42); got back to 60 mph in 3 of 12.
- `v4/`: v3 trained on the same data plus its own mistakes (DAgger): 300 staged clips and 20 drives in heavy, reckless traffic driven
  by v3, with the expert saying what it should have done at every moment and taking over before a crash. Brake test: 18 crashes in 80
  (cut-ins 2 of 52, the car ahead braking 5 of 12, slow vehicles 3 of 8; stopped cars still 8 of 8: it brakes hard but late), hardest
  braking at 105% of the expert's; got back to 60 mph in 11 of 12. Drives of 3 minutes: holds 65 mph, 4 to 6 cm off the middle; crashes
  in the heaviest traffic, as v1 does.
- `v5/`: v4 trained on its own mistakes too (DAgger round 2: 160 staged clips aimed at stopped cars, slow vehicles and braking cars,
  and 10 drives in heavy traffic, all driven by v4) and on drives in the outer lanes (10 by the expert, 10 by v4): until then every long
  drive was in the middle lane. Brake test: 18 crashes in 80, as v4 (stopped cars still 8 of 8); back to 60 mph in 11 of 12 (median 8 s,
  v4 10). In the left lane 4 cm off the middle at 62 to 65 mph (v4 once crawled at 23 mph); 3-minute drives on worlds 0 and 1: 5 of 6
  without a crash (v4 4).
- `v6/`: v5 with a second training answer: how far the car ahead is and how fast we close on it (a head that only training uses; the
  files the game runs are the same shape as v5's). It learned the distance (15 to 30 m off at 50 to 200 m) but not the closing speed of
  a car 100 to 200 m off (13 m/s off): in the 256x144 picture a sedan that far is a few pixels wide. It still does not brake for a
  stopped car 70 to 150 m ahead (plans +0.5 m/s² where the expert brakes at −3.3); on the held-back drives it drives as v5 (11 to 13 cm
  sideways at 3 s, a little quicker to speed up, braking a little softer). Brake test to come.
- `v7/`: v6 with a zoomed view: the middle 256x144 of the camera's 640x360 picture at its own detail (`model.json` `zoom`), read by a
  second ResNet-18 that started as a copy of the first; the memory takes both views (`seen: 1024`). A car 100 m ahead is 13 pixels wide in
  it, against 5 in the whole picture. It did not fix stopped cars: the closing speed of a car 100 to 200 m off went from 13.0 to 12.2 m/s
  off, and it still plans no braking for a stopped car 70 to 150 m ahead (+0.6 to +0.8 m/s² where the expert brakes at −3.3). On the
  held-back drives it drives as v6. Twice the download of the others (vision.onnx 89 MB; no model.onnx). Brake test to come.
