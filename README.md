Phone damage check

A browser app that opens your camera and tells you whether a phone looks okay or damaged, in real time. Everything runs in the browser, and no video is uploaded.

Live demo: YOUR-PAGES-LINK (use Chrome or Edge; allow camera access)

This is a student project, not a certified inspection tool.

How it works
A pretrained YOLOv8n (COCO, class cell phone) finds the phone in the frame. It runs every 4th frame.
A YOLOv8n model fine-tuned by me detects crack, scratch and display_defect on the cropped phone.
The page shows OK, Not okay, or "No phone detected". The verdict is smoothed over recent frames (damage in 2 of the last 6) so it does not flicker.
Both models are exported to ONNX and run with ONNX Runtime Web (WebGPU when available, WebAssembly otherwise).
Data

Public datasets from Roboflow Universe only. I downloaded three, then cleaned them:

Mobile Damage Diagnosis turned out to contain the same 1,901 images as the Mobile phone detection (Sengital) set (checked with perceptual hashing), so I dropped it to avoid double counting.
Crackedscreen provided undamaged phones. Its images labelled "broken phone" were dropped, because they would have become wrong "no damage" examples.
Labels were merged into 3 classes: crack, scratch, display_defect.
1,974 images became 1,884 after near-duplicate removal. The split is 1,507 train, 188 validation, 189 test, built with a fixed seed. The exact assignment is in splits.json.

Dataset licenses and sources: see each dataset's page on Roboflow Universe. TODO: add the exact license line for each dataset here.

Results (held-out test split, 189 images)

Model: YOLOv8n, 50 epochs, 640 px, seed 42.

Class	Precision	Recall	mAP50
all	0.776	0.556	0.635
crack	0.755	0.471	0.559
scratch	0.774	0.635	0.667
display_defect	0.798	0.561	0.681

Image-level verdict (NOT OK if any box passes the confidence threshold), at threshold 0.25:

Damaged phones flagged: 137 of 149
Undamaged phones passed: 35 of 40

The undamaged set is small (40 images), so that second number is rough. The ONNX export scored 0.659 mAP50 on the same test split when evaluated one image at a time, close to the PyTorch result.

Speed

On my laptop in Chrome with WebGPU, the page runs at about 66 to 77 ms per frame (13 to 15 frames per second). Without WebGPU, an earlier version that ran both models on the CPU took about 750 to 800 ms per frame. Speed depends on the device and browser. I have not yet tested on phones.

Limitations
Crack recall is the weak point (0.47). The model misses small and faint cracks.
Most training images are product-style photos, and some are mosaic-augmented by the original dataset authors. Dim light and cluttered backgrounds will do worse than the test numbers suggest.
I tried 960 px training. It raised mAP slightly on an earlier split but did not improve the OK / NOT OK verdict, so I shipped the faster 640 model. That comparison was not rerun on the final split.
The page checks one phone at a time.
Run it locally
python -m http.server 8000

Then open http://localhost:8000 in Chrome. The page needs the models/ folder next to index.html.

Files
index.html: the app
models/damage_640.onnx: damage detector
models/phone_coco_640.onnx: pretrained COCO phone detector
splits.json: train / validation / test assignment
