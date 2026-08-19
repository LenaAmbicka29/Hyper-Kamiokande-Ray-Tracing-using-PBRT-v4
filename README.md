# Hyper-Kamiokande-Ray-Tracing-using-PBRT-v4-For-Photogrammetry

This repository documents 12 weeks of work for the MITACS Globalink Research Internship undertaken between 08/06/2026-28/08/2026.

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
