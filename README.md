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
- `v8/`: v7 whose zoomed view also sees the zoomed picture from 2 s before (`model.json` `zoom.before: 40`; vision.onnx takes `before`,
  and the game keeps the last 41 zoomed views): one picture cannot show how fast a far car is closing, a car growing over 2 s can. But
  trained with the driving it never learned to read it (closing speed 3.0 / 7.9 / 12.6 m/s off at 0-50 / 50-100 / 100-200 m, as v7),
  and it still plans no braking for a stopped car 70 to 200 m ahead (+0.2 to +0.8 m/s² where the expert brakes at −2.3 to −3.2). On the
  held-back drives it drives as v7. Brake test to come.
- `v9/`: v8 whose zoomed view's ResNet was first taught on its own to judge the car ahead (distance and closing speed, from the zoomed
  picture now and 2 s before; train/eyes.py), then put in and held as it is while the rest learned to drive with it. It reads the car
  ahead best of any (closing speed 1.8 / 4.9 / 11.2 m/s off at 0-50 / 50-100 / 100-200 m, v8 3.0 / 7.9 / 12.6) and drives as v8 (12 cm
  sideways at 3 s, speeds up as v8, no braking for nothing), but it still plans no braking for a stopped car 70 to 200 m ahead (+0.15 to
  +0.48 m/s² where the expert brakes at −2.3 to −3.2; v8 up to +0.76). Brake test to come.
- `v10/`: v9's model and recipe (the zoomed view with the picture from 2 s before, its eyes held as eyes.py taught them) trained on
  v9's data plus a new round of drives: stopped cars, wrecks turned 30 to 90°, queues of stopped and braking traffic, pull-aways, at night
  and past bridges, and a later safety driver so the expert brakes hard close behind the car ahead. Data was the one change. It is the
  first to plan braking for a stopped car 70 to 200 m ahead (−0.14 / −0.35 / −0.71 m/s² at 150-200 / 100-150 / 70-100 m, where the
  expert brakes at −2.3 / −2.9 / −3.2 and v9 plans +0.15 / +0.45 / +0.48), though far softer than the expert, and no braking for nothing
  beyond 200 m. On the held-back drives: 14 cm sideways at 3 s, braking −1.56 where the expert brakes at −2.49 (v9 softer), speeding up
  as v9. The weights of its second pass (the third did worse on the checking drives). Brake test to come.
- `v11/`: v10's model and recipe, taught the expert of 2026-11-21 (brisker acceleration, keeping up with traffic, no braking for a
  faster car cutting in, braking that builds as it closes in, braking for a stopped car it sees over the car ahead): its speed is learned
  only from the drives recorded with that expert (4 new rounds: DAgger with v10 driving and the expert taking over late, hard stops,
  sudden stops behind a car that swerves out), its steering from all of them. Three passes, from v10's weights; the last pass's weights.
  Brake test (the game's eased pedals), against v10: it still runs into every stopped or suddenly stopping car (12 of 12, as v10), but
  far slower (1 to 17 mph, 8 of 12 under 8; v10 6 to 29 mph; corrected 2026-11-23: those were read one picture after the hit, already
  stopped by it; at the moment of the hit v11 was doing 11 to 61 mph), braking at up to −4.8 to −5.2 m/s² (v10 −3.3); it starts braking too late
  for a car stopped far ahead (about 70 m in one, where the expert begins at 190 m). Back up to 60 mph in 6 to 10 s once the car ahead
  changes out (v10 10 to 12), 11 to 16 s once it speeds away (v10 mostly never; the expert 7.6). Cut-ins: 4 of 51 crashes (v10 3 of 52),
  the hardest (0.4 to 0.6 s ahead, 10 to 20 mph slower). No braking for nothing on an empty road (−0.2 m/s² at most, v10 −0.4); 3
  drives of 3 minutes without a crash. On the held-back drives: braking −2.29 where the expert brakes at −3.04, speeding up 1.38 (2.08).
- `v12/`: v11's model and recipe trained 2 more passes on everything v11 had plus 450 clips driven by v11 with the expert saying what it
  should have done: a car stopped behind one that swerves out, the same clips with nobody behind it (hold your speed), and faster cars
  cutting in 5 to 25 m ahead (hold your speed). It came from user flags of v11 braking for nothing. Brake test (112 runs, the same for
  expert, v11, v12): 16 crashes (v11 17; the expert 0). When the car ahead swerves onto an empty lane at 65 mph it slows to 34 to 65 mph
  (v11 27 to 59; the expert 57). Faster cut-ins 0 of 16 crashes, as v11, slowing to 45 to 65 mph (v11 43 to 65; the expert 62 to 65).
  Other cut-ins 0 of 48 (v11 1). Worse: it still runs into every stopped or suddenly stopping car (12 of 12, as v11) and harder, at 8 to 61 mph
  (median 41; v11 7 to 61, median 30), braking up to -5.2 m/s² but late. It also crashes into every car cutting in 0.6 s ahead at 20 mph slower
  (4 of 4; v11 3). 3 drives of 3 minutes: 2 with no crash (4 cm off the middle); in the heaviest traffic at dusk it ran into the car ahead after 21 s
  (v11 left its lane there after 34 s).
- `v12b/` and `v12c/` (experiments, 2026-11-23): v12 trained 1 more pass on v12's own data with two training changes, to see what
  they do before v13's new recordings. Each training clip's pictures were made brighter or darker, flatter or harsher, and tinted, one
  look per clip (v12 braked for stopped cars on clips it had trained on and not on new ones: it knew them by their look). Where the
  expert brakes, planning too little braking within 1 s cost 3 times as much as too much. v12c's zoomed-view ResNet also learned;
  v12b's was held as it was, as in v12. On held-out staged clips, its share of the expert's braking for a stopped car at 100-120 /
  80-100 / 60-80 m: v12b 16 / 46 / 61%, v12c 39 / 52 / 77% (v12 13 / 29 / 51%); nothing yet past 120 m. Where the expert plans
  harder than -6 m/s²: v12b -4.5, v12c -4.6 (v12 -3.9). Brake test (186 runs, also in the pickup and the sports car): 31 crashes
  each (v12 35; the expert 0). Stopped cars still 11 of 12 (v12 12), braking from 61 m (v12b) and 78 m (v12c) away, the expert from
  170 m. A car ahead braking to a stop from 45 mph: v12b 1 crash in 6, v12c 2 (v12 3); from 25 mph: v12b 4, v12c 5 (v12 3). Worse:
  more braking for nothing: the car ahead swerving onto an empty lane at 65 mph, down to 22 to 24 mph on straight and right-hand
  roads (v12 30 to 33; the expert 57), and hanging back at 28 to 34 mph behind a car pulling away to 50 (v12 36). Still about 40 cm
  off the middle in the pickup (v12b 34 cm, v12c 43). 3 drives of 3 minutes each without a crash, including the heaviest traffic at
  dusk where v12 crashed after 21 s.
- `v13-p1/` and `v13-p2/` (experiments, 2026-11-25): v12c trained 2 passes on v11-v12's recordings plus v13's (1,190 new: stopped and
  braking traffic, a car cutting in or swerving out, drives in dense traffic, lane drives; v12c driving with the expert taking over once it
  would brake harder than 3-4 m/s²; our car any of 5 body styles; 35% at dusk or night). Training changes: pictures varied (brightness,
  contrast, tint, lens distortion, raindrops on the lens, blur, haze, grain); the plan's first second as hard as the braking the expert
  would command right then; braking too little cost 1.5 times. p1 and p2 are the model after each pass. Held-out stopped-car clips,
  share of the expert's braking at 120-150 / 100-120 / 80-100 m: p1 21 / 51 / 66%, p2 22 / 67 / 77% (v12c on the same clips 4 / 23 / 43%).
  World 1, a highway no model trained on: p1 35 / 34 / 57%, p2 60 / 40 / 59% (v12c 1 / 1 / 22%); where the expert plans harder than
  -6 m/s² there, p1 and p2 plan -3.5 (v12c -0.6). Brake test (186 runs): p1 19 crashes, p2 23 (v12c 30, the expert 0). Still into
  11 of 12 stopped cars, from about 85 m (v12c 75, the expert 150-190), slower at the hit (27-38 mph, v12c 40). The car ahead braking hard
  (6 m/s²) to a stop: p1 1 crash in 6, p2 5 (v12c 6); from 25 mph gently: p1 6, p2 5 (v12c 4). Pickup 5-7 cm off the middle
  (v12c 48). More braking for nothing than v12c: a car ahead swerving out at 65 mph, down to 33 mph (v12c 44, the expert 57); p1 crawls
  at 17-21 mph behind a car slowing to 45 or speeding away.

- `v14-p1/` and `v14-p2/` (experiments, 2026-11-26, published before their tests): v13-p2 trained 2 passes on v13's recordings plus
  v14's: stopped cars with twins of the same scene where the car drives on at 55-80 mph or isn't there, v13-p2 in heavy traffic with the
  expert labelling, and the expert in stop-and-go jams (it now follows close and brakes gently at low speed). Braking too little no
  longer costs extra, and pictures where the expert slows for a car the camera can't show yet are left out. On recorded pictures after
  pass 1: far less braking where the expert holds speed, a little softer braking for stopped cars. Phantom-braking and brake tests to come.

**2026-11-26: vision.onnx of v8 to v14-p2 replaced (same weights, fixed export).** From v8 on, the zoomed view's network starts with a
convolution of 6 channels (the picture now and how it changed over 2 s). onnxruntime-web's WebGPU, which the game uses where the browser
has it, worked that convolution out wrong, so in the game the zoomed view's numbers were not the trained model's: on the same pictures
v14-p2's plans in the game matched its own only 0.2-0.4 (correlation), and it braked for nothing far more. The new files do the same sum
as two 3-channel convolutions added together, which WebGPU gets right (checked: the same numbers on WebGPU, on the CPU and in PyTorch,
for all 11 models). Every brake test and phantom test of v8 to v14 so far drove with the faulty view and has to be run again.

- `v15/` (experiment, 2026-11-27, published before its tests): v14-p2 retaught by a calmer expert (it gives no gas while closing on
  a slower car, matches the speed of the car ahead more gently, and is loose about the exact gap), its old recordings relabelled to
  that pace, plus its own mistakes in stop-and-go and cut-ins with a late safety driver. One fine-tuning pass of 4500 steps. The zoomed
  view's first convolution is split for WebGPU (see the note above).

- `v16a-p1/` (experiment, 2026-11-28, published before its tests): v15 fine-tuned 4500 steps with its zoomed view taken from the same camera through a lens twice as narrow (model.json zoom.tele = 2; the game films that view beside the usual one), world 4's drives (its 40 mph bend held out) and DAgger rounds of v15 on world 4's bends, in heavy traffic and on staged cut-ins, lane changes and stopped cars. Recordings without the narrow view gave the middle half of their zoomed view blown up.

- `v16b-p1/` (experiment, 2026-11-28, published before its tests): v16a-p1 fine-tuned 4000 more steps on the same data with training clips that turn tighter than 500 m drawn 8 times as often (21% of what was drawn).
