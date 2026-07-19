# SIFT + RANSAC Multi-Object Detection in Cluttered Scenes

A classical computer vision pipeline that finds and localizes multiple known objects (book covers / product templates) inside cluttered, real-world photos — using SIFT feature matching, RANSAC-based homography estimation, and geometric plausibility checks. No deep learning involved: every step is interpretable and hand-tuned.

Given a small set of reference "model" images (e.g. book covers) and a "scene" photo (e.g. a bookshelf), the system finds every instance of every model that appears in the scene, outlines its location, and repeats until no further instances remain.

## Example output

<p align="center">
  <img src="assets/detection_examples/scene_multi_book_detection.png" width="700" alt="Three different books detected simultaneously in one scene">
  <br><em>Three different reference books detected in a single scene (6 total instances) — each color-coded box corresponds to the matching thumbnail below it.</em>
</p>

<p align="center">
  <img src="assets/detection_examples/scene_repeated_instances.png" width="700" alt="Two copies of the same book both correctly detected as separate instances">
  <br><em>Two side-by-side copies of the same book, both correctly detected as separate instances.</em>
</p>

## How it works

The pipeline runs in seven stages for each (model, scene) pair:

1. **Feature extraction (SIFT)** — keypoints and 128-d descriptors are extracted separately from the model and the scene, with slightly different contrast thresholds to compensate for lighting differences between a clean template and a real photo.
2. **Descriptor matching (Brute-Force + kNN, k=2)** — candidate correspondences between model and scene keypoints.
3. **Lowe's ratio test** — ambiguous matches are discarded by requiring the best match to be meaningfully closer than the second-best.
4. **Match filtering** — a detection is only considered if it clears both an absolute and a relative match-count threshold, preventing spurious detections from a handful of accidental matches.
5. **Homography estimation (RANSAC)** — a projective transform from model to scene is fit robustly, discarding outlier correspondences.
6. **Model projection** — the model's four corners are projected into scene coordinates via the homography, producing a candidate quadrilateral.
7. **Geometric validation** — the candidate is rejected unless it is both convex and sufficiently "rectangular" (contour area / bounding-box area above a threshold), filtering out degenerate homographies before they're accepted as detections.

Multiple instances of the same model in one scene are found by repeating the process and masking out already-detected regions, so the same object is never counted twice.

## Key parameters

| Parameter | Value | Why |
|---|---|---|
| `sigma` (model / scene) | 0.4 | Low value to preserve fine detail rather than smoothing it away |
| `contrastThreshold` (model / scene) | 0.025 / 0.024 | Slightly different per image type to balance keypoint density between clean templates and noisier scene photos |
| `lowes_ratio` | 0.75 | Standard balance between match precision and recall |
| `min_good_match` | 15 | Minimum absolute number of matches before a homography is even attempted |
| `min_match_percent` | 0.03 | Minimum matches as a fraction of all candidate matches — guards against "many matches but all noise" |
| `min_rectangularity` | 0.6 | Rejects warped/degenerate homographies whose projected shape doesn't look like a plausible rectangle |

## Data

The pipeline expects two folders:
- `models/` — one clean reference image per object to detect
- `scenes/` — photos in which to search for those objects

## Limitations

- Relies on distinctive local texture — near-featureless or highly reflective object covers are harder to match reliably.
- Heavy occlusion beyond what's shown in the examples can drop the match count below the acceptance thresholds.
- Purely classical (no learned features), so performance depends entirely on hand-tuned parameters rather than learned robustness — a deliberate constraint of the project, not a general limitation of the approach.

## Credits

Developed together with Ali Saleh Mohammadabad as a computer vision course project, University of Bologna.
