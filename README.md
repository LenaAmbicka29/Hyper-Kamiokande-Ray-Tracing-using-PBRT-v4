# Hyper-Kamiokande-Ray-Tracing-using-PBRT-v4-For-Photogrammetry

This repository documents 12 weeks of work for the MITACS Globalink Research Internship undertaken between 08/06/2026-28/08/2026.
# Hyper-Kamiokande Photogrammetry Raytracing

This repository contains a PBRT-v4 based simulation framework for photogrammetry studies in the Hyper-Kamiokande Far Detector.

The project is intended to generate synthetic detector images from known geometry, detect photogrammetry features in those images, assign detections to known LED-mPMTs, and reconstruct their 3D positions using multi-view triangulation.

The main workflow is:

```text
Detector geometry
      ↓
PBRT scene generation
      ↓
Synthetic camera renders
      ↓
Feature detection
      ↓
mPMT ID matching
      ↓
Multi-view triangulation
      ↓
Comparison with known detector coordinates
```

## Project Scope

The current model includes:

- Hyper-K Far Detector cylindrical geometry
- 20-inch PMTs
- Far Detector multi-PMT modules
- LED-mPMTs
- Photogrammetry feature targets
- Blacksheet/support geometry
- Detector water medium
- Eight fixed photogrammetry camera positions
- Fisheye camera support
- Feature detection and ground-truth matching
- Multi-view 3D reconstruction

The main purpose of the simulation is not to reproduce every optical component of Hyper-K exactly, but to provide a controlled environment in which photogrammetry algorithms can be developed and tested against known ground-truth positions.

## mPMT Coordinate Convention

The mPMT positions are read from:

```text
mpmt_position.txt
```

with columns:

```text
id  x(m)  y(m)  z(m)  location  isled
```

where:

- `id` is the mPMT identifier
- `x`, `y`, `z` are the mPMT coordinates in metres relative to the centre of the tank
- `location`
  - `0` = top
  - `1` = barrel
  - `2` = bottom
- `isled`
  - `1` = LED-mPMT
  - `0` = regular mPMT

The local mPMT model uses:

```text
local z = 0
```

as the rear/support-matrix mounting plane.

The local `+z` direction points from the detector wall into the detector volume.

This convention is important for photogrammetry because the coordinate in `mpmt_position.txt` is treated as the nominal mPMT reference position that should ultimately be recovered by triangulation.

## Far Detector mPMT Model

The FD mPMT model is defined in:

```text
fd_mpmt_model.py
```

The current geometry uses the Far Detector PMT layout:

```text
PMT rows                : 1, 6, 12
central PMT position    : 224.678 mm from original baseplate reference
row transverse radii   : 0, 109.487, 199.909 mm
row axial offsets       : 0, -20.8, -75.543 mm
row tilts               : 0, 0.297, 0.593 rad
baseplate diameter      : 590 mm
3-inch PMT radius       : 38.1 mm
```

The outer PMTs tilt radially outward.

The support matrix is imported from:

```text
matrix_pbrt.ply
```

The currently tuned matrix transform is:

```text
matrix scale x = 1.370625
matrix scale y = 1.370625
matrix scale z = 1.485
matrix z offset = -0.150 m
matrix rotation = 30 degrees
```

The complete module is rebased so that the rear of the support matrix lies at:

```text
z = 0
```

This preserves the internal PMT/matrix alignment while making the module reference plane suitable for detector placement and triangulation.

## LED-mPMTs and Photogrammetry Features

LED-mPMTs replace five normal PMT positions with LED-FEB modules:

- One central position
- Four outer-row positions separated by 90 degrees

These five locations form the cross-like feature pattern used by the photogrammetry detector.

The LED-FEB faces are currently shifted forward by:

```text
0.040 m
```

along their individual PMT-slot normals so that the photogrammetry targets are not obscured by the support matrix.

For triangulation studies, a simplified one-sided Lambertian emissive disk can be used instead of the full physical LED optical stack.

This provides a clean directional photogrammetry feature while preserving the known target geometry.

## Camera Layout

The current detector configuration uses eight fixed cameras:

```text
top_cam_ne
top_cam_nw
top_cam_se
top_cam_sw

bottom_cam_ne
bottom_cam_nw
bottom_cam_se
bottom_cam_sw
```

The cameras are placed near the transition between the detector barrel and the top/bottom caps and point toward the detector interior.

The current high-resolution photogrammetry configuration uses:

```text
9576 × 6388 pixels
```

with the custom fisheye camera model.

## Rendering

A typical photogrammetry scene is generated with the main Hyper-K renderer and then rendered with PBRT-v4.

Example PBRT execution:

```bash
~/pbrt-v4-build/pbrt --gpu top_cam_ne_disk.pbrt
```

Typical final-render settings include:

```text
pixelsamples = 1024
maxdepth = 32
sampler = zsobol
randomization = fastowen
pixel filter = mitchell
```

The renderer can disable normal detector and environment lighting when producing photogrammetry-only datasets so that the LED targets remain easy to isolate.

## Feature Detection

The image-processing stage identifies the five-point LED-mPMT feature pattern.

The general process is:

```text
image
  ↓
intensity extraction
  ↓
thresholding
  ↓
connected-component detection
  ↓
blob filtering
  ↓
cross candidate construction
  ↓
geometric filtering
  ↓
detected LED-mPMT centres
```

Detection outputs can include:

```text
annotated PNG
processed image
binary image
JSON feature data
```

High-resolution images can produce many reflection-induced blobs, so thresholding and blob filtering should be tuned before running the full cross search.

## mPMT ID Matching

Detected features are matched to known LED-mPMTs by reprojecting their expected 3D positions into each camera image.

The matcher uses:

- Known detector geometry
- The exact PBRT camera parameters
- Projected mPMT/feature positions
- Pixel-space assignment

The output is a CSV containing detected feature coordinates and their matched mPMT IDs.

This allows detections from different cameras to be associated with the same physical module.

## Triangulation

For each matched LED-mPMT, detections from two or more camera views are converted into world-space rays.

The reconstructed 3D point is obtained by minimizing the perpendicular distance to all contributing rays.

The photogrammetry target position is reconstructed first.

Because the target location relative to the mPMT reference plane is known from the model geometry, the reconstructed feature coordinate can then be transformed back into a reconstructed mPMT coordinate.

The final quantity of interest is therefore:

```text
reconstructed mPMT XYZ
        -
nominal mPMT XYZ
```

which gives the 3D reconstruction error.

## Geometry Perturbation Studies

A useful validation test is to deliberately modify the detector geometry and test whether the photogrammetry pipeline can recover the introduced displacement.

Example perturbations include:

```text
global position shift
barrel radial displacement
vertical displacement
sinusoidal detector distortion
random per-mPMT displacement
```

The intended comparison is:

```text
injected displacement
        vs
reconstructed displacement
```

This provides a direct test of the sensitivity of the photogrammetry system to detector-position changes.

## Important Approximations

The current model includes several simplified components.

These include:

- Approximate acrylic dome geometry
- Approximate internal optical materials
- Simplified 3-inch PMT optical response
- Simplified photocathode material
- Simplified LED source models
- Approximate support-matrix scaling
- Idealized camera positioning

These approximations should be considered when interpreting absolute optical realism.

The photogrammetry geometry and coordinate transformations should remain explicit and internally consistent even when optical components are simplified.

## Repository Structure

A typical working directory contains:

```text
fd_mpmt_model.py
fd_led_feb_model.py
HYPER_K_RENDERER_PHOTOGRAMMETRY.py
matrix_pbrt.ply
mpmt_position.txt
hyperk_20inch_pmts.npz
coefficients.xlsx

detect_crosses_*.py
mpmt_id_matcher_*.py
triangulate_mpmts_*.py
```

Generated PBRT scenes, rendered images, detector JSON files, matched CSV files and triangulation outputs should generally be kept separate from the source files.

## Software

The project uses:

```text
Python
NumPy
SciPy
OpenCV
PBRT-v4
```

Optional GPU rendering uses PBRT's OptiX backend where supported.

## Current Pipeline Status

The current pipeline supports:

```text
geometry generation        ✓
8-camera rendering          ✓
LED feature rendering       ✓
feature detection           ✓
ground-truth ID matching    ✓
multi-view triangulation    ✓
3D error analysis           ✓
geometry perturbation tests in progress
```

## Notes for Future Development

Important areas for further development include:

- Final verification of physical mPMT dimensions
- Improved LED-FEB optical modelling
- Improved feature detection performance on full-resolution images
- Further validation of the fisheye camera model
- Comparison with measured lens calibration data
- Automated rendering of all eight camera views
- Automated end-to-end reconstruction
- Detector deformation and displacement recovery studies
- Comparison of reconstruction accuracy against camera coverage and viewing angle
- Improved documentation of measured parameters versus model approximations
