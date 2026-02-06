# Synthetic Event Camera Dataset Generation for Space Situational Awareness

## Research Summary

This document consolidates research on generating realistic synthetic datasets using Prophesee event-based cameras for space domain awareness (SDA) / space situational awareness (SSA). The goal is to create training data for models that detect and track satellites and space debris using neuromorphic vision sensors.

---

## Table of Contents

1. [Why Event Cameras for Space](#1-why-event-cameras-for-space)
2. [Event Camera Simulators](#2-event-camera-simulators)
3. [Prophesee-Specific Tools](#3-prophesee-specific-tools)
4. [Space Scene Rendering](#4-space-scene-rendering)
5. [Orbital Mechanics Simulation](#5-orbital-mechanics-simulation)
6. [Star Field Simulation](#6-star-field-simulation)
7. [Event Camera Data Formats](#7-event-camera-data-formats)
8. [Existing Datasets](#8-existing-datasets)
9. [Best Practices for Realism](#9-best-practices-for-realism)
10. [Recommended Pipeline](#10-recommended-pipeline)

---

## 1. Why Event Cameras for Space

Event cameras (neuromorphic / dynamic vision sensors) report per-pixel brightness changes asynchronously with microsecond temporal resolution. Advantages for SSA:

- **~36x data reduction** vs. frame-based equivalents
- **>120 dB dynamic range** -- enables daytime satellite observation
- **No motion blur** -- continuous asynchronous operation
- **Low power** -- 10-400 mW, ideal for remote ground stations and space-based platforms
- **Microsecond latency** -- critical for fast-moving LEO objects

### Key Research Groups

| Group | Contribution |
|-------|-------------|
| **Western Sydney University ICNS** | Astrosite observatory, EBSSA dataset, FIESTA tracking algorithm |
| **Prophesee** | Metavision sensors (EVK4-HD, Gen 4 HD), OpenEB SDK |
| **iniVation** | DAVIS240C -- first neuromorphic sensor in space (March 2021) |
| **AFIT** | Satellite tracking with neuromorphic cameras thesis (2021) |
| **Arete** | Neuromorphic SDA event cameras at thousands of fps |
| **DARPA FENCE** | Fast Event-based Neuromorphic Camera and Electronics program |

---

## 2. Event Camera Simulators

### 2.1 Comparison Table

| Simulator | Year | Input | Noise Model | Temporal Fidelity | 3D Scene | Open Source |
|-----------|------|-------|------------|-------------------|----------|-------------|
| **ESIM** | 2018 | 3D scene (UE/OpenGL) | Gaussian on CT | Adaptive sampling | Yes | Yes |
| **v2e** | 2021 | Video frames | BW, leak, threshold mismatch | Discrete timestamps | No | Yes |
| **DVS-Voltmeter** | 2022 | Video frames | Circuit physics, stochastic | Spread timestamps | No | Yes |
| **V2CE** | 2024 | Video frames | Learned | Continuous timestamps | No | Yes |
| **PECS** | 2024 | 3D scene (UE) | Full optical path + spectrum | High fidelity | Yes | Partial |
| **ADV2E** | 2024 | Video frames | Analog circuit filters | Analog-accurate | No | TBD |
| **DAVIS Sim** | 2017 | 3D scene (Blender) | Basic threshold | Frame-based | Yes | Yes |

### 2.2 ESIM (Recommended for 3D scenes)

- **Paper:** Rebecq, Gehrig, Scaramuzza (CoRL 2018)
- **Code:** https://github.com/uzh-rpg/rpg_esim
- Tightly couples rendering engine and event simulator
- Adaptive rendering -- only samples frames when scene dynamics require it
- Events generated via level-crossing sampling on log-intensity
- Rendering backends: OpenGL and Unreal Engine (via UnrealCV)
- Supports Blender model/trajectory import
- Ground truth: poses, IMU, depth maps, optical flow

### 2.3 v2e (Recommended for video-to-events)

- **Paper:** Hu, Liu, Delbruck (CVPRW 2021)
- **Code:** https://github.com/SensorsINI/v2e
- Converts conventional video into DVS event streams
- Uses deep learning frame interpolation for temporal upsampling
- Realistic DVS pixel model: intensity-dependent bandwidth, per-pixel threshold mismatch, leak events
- Can calibrate parameters against real DVS recordings
- Outputs jAER-compatible `.aedat` format
- **Ideal bridge** between conventional space scene renderers and event data

### 2.4 DVS-Voltmeter (Highest circuit fidelity)

- **Paper:** Lin et al. (ECCV 2022)
- **Code:** https://github.com/Lynn0306/DVS-Voltmeter
- Models actual DVS circuit physics: photocurrent, logarithmic voltage, comparator triggering
- Stochastic process model spreads event timestamps realistically
- Neural networks trained with DVS-Voltmeter generalize better to real events

### 2.5 PECS (State-of-the-art fidelity)

- **Paper:** Han et al. (ECCV 2024)
- **Code:** https://github.com/lanpokn/PECS_trail_version
- Generates events directly from 3D scenes in Unreal Engine
- Full optical path modeling: lens simulation, multispectral rendering, quantum efficiency
- Outperforms ESIM, v2e, DVS-Voltmeter on event fidelity metrics

### 2.6 V2CE (Continuous timestamps)

- **Paper:** Zhang et al. (ICRA 2024)
- **Code:** https://github.com/ucsd-hdsi-dvs/V2CE-Toolbox
- Two-stage: motion-aware event prediction + statistics-based timestamp inference
- Solves the discrete-timestamp problem of v2e

### 2.7 ADV2E (Analog-aware)

- **Paper:** Jiang et al. (arXiv:2411.12250)
- Models low-pass filters in DVS cascode feedback loop
- Prevents events from abruptly vanishing in high-contrast scenes

---

## 3. Prophesee-Specific Tools

### 3.1 OpenEB / Metavision SDK

- **Code:** https://github.com/prophesee-ai/openeb
- **Docs:** https://docs.prophesee.ai/stable/index.html

Key modules for synthetic data:

- **Core ML Module** -- `video_to_event` and `event_to_video` pipelines
  - `GPUSimulator` / `GPUEBSIM`: GPU-accelerated event simulation from image sequences
  - `CameraPoseGenerator`: Defines camera trajectories, generates continuous homographies
  - `TimedVideoStream`: Video iterator with timestamp generation
- **"From Frames to Events" Tutorial:** https://docs.prophesee.ai/stable/tutorials/ml/dataset_creation/video_to_event_simulator.html
  - Generates synthetic event sequences from any image stream
  - Adaptive sampling with motion-based frame interpolation
  - Synthetic events generated on-the-fly during training
- **Pre-trained e2v model** at `sdk/modules/core_ml/models/e2v.ckpt`

### 3.2 Data Format Tools

- `metavision_file_info`: Identifies EVT format of `.raw` files
- File to HDF5 converter: Converts RAW/DAT to compressed HDF5
- RAW file writer: Writes EVT2 format

---

## 4. Space Scene Rendering

### 4.1 Rendering Tools

| Tool | Type | Pros | Cons |
|------|------|------|------|
| **Blender (Cycles)** | Path tracer | Free, open source, Python API, PBR | Slower than realtime |
| **SurRender (Airbus)** | Raytracer | Space-specific, physical units, BRDFs | Commercial |
| **SPIN** | Blender-based | Spacecraft-specific, ground truth poses | Limited to pose estimation |
| **Unreal Engine** | Real-time | Fast, photorealistic, ESIM/PECS integration | Complex setup |
| **BlenderProc** | Blender pipeline | Automated, procedural generation | Learning curve |

### 4.2 Blender for Space Scenes (Recommended)

Blender is the most practical choice for our pipeline:

- **Free and open-source** with full Python API for automation
- **Cycles renderer** provides physically correct path tracing
- Used by ESA (space debris visualization with Orekit), NCSTP dataset (200k images), SPIN simulator
- **S.T.A.R.** (https://github.com/jasmeet0915/S.T.A.R) provides Blender + Python orbital simulation using TLE data
- Supports HDR/OpenEXR output (critical for preserving DVS dynamic range)

### 4.3 Physics-Based Illumination for Space

Must model these illumination sources:

1. **Direct solar illumination** -- Primary light source, varies with orbital position
2. **Earth albedo** -- Sunlight reflected from Earth (Lambertian model), secondary source
3. **Planetshine** -- Light from other planetary bodies
4. **Eclipse transitions** -- Gradual illumination changes as satellites enter/exit Earth's shadow

Reflection phenomena:
- **Specular glints** from solar panels, metallic surfaces, MLI
- **Fresnel effects** -- reflectivity varies with incidence angle
- **BRDF models** -- microfacet-based for realistic material rendering

### 4.4 Satellite 3D Models

Sources for spacecraft CAD models:
- NASA 3D Resources (https://nasa3d.arc.nasa.gov/)
- Celestia Motherlode
- Custom parametric models (CubeSats, generic bus configurations)
- SPARK dataset spacecraft models

---

## 5. Orbital Mechanics Simulation

### 5.1 Core Libraries

| Tool | Language | Key Feature |
|------|----------|-------------|
| **SGP4** | Python/C++ | Standard TLE propagation algorithm |
| **Skyfield** | Python | TLE parsing, propagation, coordinate conversion, star catalogs |
| **Astropy** | Python | Coordinate frame transformations (TEME to ICRS/ITRS) |
| **poliastro** | Python | Interactive astrodynamics, orbit visualization |
| **Orekit** | Java/Python | High-fidelity astrodynamics (ESA operational) |
| **GMAT** | C++ (GUI) | NASA's General Mission Analysis Tool |

### 5.2 Recommended: Skyfield + SGP4

```python
from skyfield.api import load, EarthSatellite

ts = load.timescale()
line1 = '1 25544U 98067A   ...'  # ISS TLE
line2 = '2 25544  51.6442 ...'
satellite = EarthSatellite(line1, line2, 'ISS', ts)
t = ts.now()
geocentric = satellite.at(t)
```

- Skyfield wraps SGP4 with easy coordinate conversions
- Can load Hipparcos star catalog as Pandas dataframe
- TLE data sourced from CelesTrak (https://celestrak.org/)

### 5.3 Observer Simulation

For ground-based telescope simulation:
- Define observer location (lat, lon, elevation)
- Compute satellite apparent position (azimuth, elevation, range)
- Determine visibility windows (satellite illuminated, observer in darkness)
- Account for atmospheric refraction

---

## 6. Star Field Simulation

### 6.1 Star Catalogs

| Catalog | Stars | Precision | Size |
|---------|-------|-----------|------|
| **Hipparcos** | 118,218 | ~1 mas | ~52 MB |
| **Tycho-2** | ~2.5 million | ~25 mas | ~160 MB |

### 6.2 Rendering Pipeline

1. Load catalog via Skyfield: `hipparcos.load_dataframe()`
2. Compute apparent positions for observer location and time
3. Convert stellar magnitudes to pixel intensities (Pogson scale)
4. Apply blackbody color model from B-V magnitude (Flower 1996)
5. Render as Gaussian PSFs matching simulated optical system
6. NASA SVS Tycho Skymap v2.0 provides reference implementation

### 6.3 Key Consideration for Event Cameras

Stars appear as static points -- they should generate **no events** unless the camera is rotating (sidereal tracking). Satellite streaks crossing star fields will generate events. This is a key discriminant for RSO detection algorithms.

---

## 7. Event Camera Data Formats

### 7.1 Fundamental Event Representation

```
event = (x, y, p, t)
  x: pixel x-coordinate (uint16)
  y: pixel y-coordinate (uint16)
  p: polarity (+1 brightness increase, -1 or 0 decrease)
  t: timestamp in microseconds (int64)
```

### 7.2 Prophesee Formats

| Format | Bits/Word | Key Feature | Resolution |
|--------|-----------|-------------|-----------|
| **EVT 2.0** | 32 | Non-vectorized, simple | Up to 2048x2048 |
| **EVT 2.1** | 64 | Vectorized, grouped events | High event rates |
| **EVT 3.0** | 16 | Delta-encoded, stateful decode | Maximum compactness |
| **RAW** | Variable | Container with EVT header | Depends on EVT version |
| **HDF5** | Compound | Compressed, indexed every 2ms | Any |

### 7.3 Tensor Representations for Deep Learning

| Representation | Description | Temporal Info |
|---------------|-------------|---------------|
| **Event Frame / Histogram** | Sum polarities per pixel over time window | Lost |
| **Time Surface (SAE)** | exp(-\|t - T(x,y)\| / tau) per pixel | Partial |
| **Voxel Grid** | 3D histogram (x, y, time_bin) | Discretized |
| **EST (Learned)** | End-to-end differentiable representation | Learned |
| **Point Cloud / Graph** | Treat events as 3D points or graph nodes | Full |

Tools: https://github.com/TimoStoff/event_utils

---

## 8. Existing Datasets

### 8.1 Event-Based Space Datasets

| Dataset | Year | Type | Sensor | Details |
|---------|------|------|--------|---------|
| **SPADES** | 2024 | Synthetic + Real | Prophesee EVK4-HD | Spacecraft pose estimation, Unreal Engine rendering |
| **SEENIC** | 2022 | Synthetic | Neuromorphic | 20 scenes with events and ground truth poses |
| **Ev-Satellites** | 2024 | Real | Prophesee Gen 4 HD | 100 recordings, noise filtering benchmark |
| **EBSSA** | 2020 | Real | DVS | 572 labeled RSOs, 8+ hours, first public event-based SSA dataset |
| **ESA Neuromorphic Pipeline** | -- | Synthetic | Simulated | 500 trajectories with event streams and motion field GT |

### 8.2 Frame-Based Space Datasets

| Dataset | Year | Images | Type | Tasks |
|---------|------|--------|------|-------|
| **SPARK** | 2021 | ~150k RGB + 150k Depth | Synthetic | Detection, classification, segmentation |
| **SPEED+** | 2021 | 60k synthetic + 9.5k HIL | Synthetic + Real | 6DoF pose estimation |
| **SpaceDet** | 2025 | 781.5 GB | Synthetic | RSO detection from space-based sensors |
| **NCSTP** | 2025 | 200,000 | Synthetic (Blender) | Detection, recognition, segmentation |
| **SDTD** | 2025 | 18k videos (62k frames) | Synthetic | Space debris tracking |

---

## 9. Best Practices for Realism

### 9.1 Critical: Contrast Threshold (CT) Modeling

The CT is the most important simulation parameter:

- **Per-pixel variation is essential**: Manufacturing mismatch, typically N(mu=0.18, sigma=0.03)
- **Positive/negative asymmetry**: Leakage current makes positive events trigger more easily
- **Calibration to real data**: Use v2e's calibration algorithm to match synthetic CT distributions to real recordings
- **Reference:** Stoffregen et al. (ECCV 2020) showed ~50% performance drop without proper CT matching

### 9.2 Noise Sources to Model

1. **Shot noise** -- Poisson-distributed from photon arrival randomness
2. **Dark current / leak events** -- Temperature-dependent background events (especially important for space: thermal extremes)
3. **Bandwidth limitations** -- Intensity-dependent; low-light conditions (common in space) significantly reduce bandwidth
4. **Hot pixels** -- Abnormally high event rate at specific coordinates
5. **Refractory period** -- Dead time after event firing
6. **Analog circuit filtering** -- Low-pass filter causes time delays in high-contrast scenes

### 9.3 Space-Specific Considerations

- **Low-light conditions**: Many observations are at low illumination -- model bandwidth limitations carefully
- **High contrast**: Bright satellite against dark sky -- analog filtering effects become significant
- **Long exposures**: Ground-based telescopes may track sidereal rate or satellite rate
- **Atmospheric effects**: For ground-based: turbulence (Fried parameter), scintillation, sky background
- **HDR rendering**: Store frames in OpenEXR float format to preserve DVS dynamic range

### 9.4 Reducing Sim-to-Real Gap

Key strategies from Stoffregen et al. (ECCV 2020):
1. Match CT to target sensor characteristics
2. Include realistic noise distributions
3. Use domain adaptation techniques
4. Validate with Event Quality Score (EQS) metric (CVPR 2025 Workshop)

---

## 10. Recommended Pipeline

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    SYNTHETIC DATA PIPELINE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐       │
│  │   Orbital     │    │  Star Field   │    │  Satellite   │       │
│  │  Mechanics    │    │  Generation   │    │  3D Models   │       │
│  │  (Skyfield)   │    │ (Hipparcos)   │    │  (Blender)   │       │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘       │
│         │                   │                    │               │
│         └───────────┬───────┴────────────────────┘               │
│                     ▼                                            │
│         ┌───────────────────────┐                                │
│         │   Blender Rendering    │                                │
│         │   (Cycles / EEVEE)     │                                │
│         │   - Solar illumination │                                │
│         │   - Earth albedo       │                                │
│         │   - Material BRDFs     │                                │
│         │   - OpenEXR output     │                                │
│         └───────────┬───────────┘                                │
│                     ▼                                            │
│         ┌───────────────────────┐                                │
│         │  Event Camera Sim      │                                │
│         │  (v2e / ESIM / OpenEB) │                                │
│         │  - CT variation        │                                │
│         │  - Noise injection     │                                │
│         │  - Bandwidth model     │                                │
│         └───────────┬───────────┘                                │
│                     ▼                                            │
│         ┌───────────────────────┐                                │
│         │  Output & Annotation   │                                │
│         │  - EVT2 / HDF5 format  │                                │
│         │  - Bounding boxes      │                                │
│         │  - Trajectories        │                                │
│         │  - Object metadata     │                                │
│         └───────────────────────┘                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Stage 1: Scene Definition

- **Orbital trajectories**: Use Skyfield + SGP4 with real TLE data from CelesTrak
- **Observer setup**: Define ground-based telescope location, FOV, mount type
- **Target selection**: Choose satellite types (LEO/MEO/GEO), debris sizes, magnitudes

### Stage 2: Rendering

- **Blender + Cycles** for physically-based rendering
- Import satellite CAD models with realistic materials
- Generate star backgrounds from Hipparcos/Tycho-2
- Model solar illumination, Earth albedo, eclipse transitions
- Output high-framerate OpenEXR sequences (1000+ fps equivalent via interpolation)

### Stage 3: Event Simulation

**Option A -- v2e (Simplest, good baseline):**
- Convert rendered video to events
- Apply realistic noise model with calibrated parameters
- Good for rapid prototyping

**Option B -- OpenEB GPUSimulator (Prophesee-native):**
- Direct integration with Prophesee data ecosystem
- GPU-accelerated, on-the-fly generation during training
- Native RAW/HDF5 output

**Option C -- ESIM (Highest control):**
- Tight rendering-simulation coupling
- Adaptive sampling for temporal accuracy
- Best for ground truth generation

### Stage 4: Annotation & Export

- Ground truth from orbital mechanics (position, velocity, magnitude)
- Bounding boxes / tracks in event stream coordinates
- Object class labels (satellite type, debris, star)
- Export in Prophesee-compatible formats (EVT2 RAW, HDF5)

### Key Dependencies

```
# Python packages
skyfield          # Orbital mechanics + star catalogs
sgp4              # TLE propagation
astropy           # Coordinate transformations
numpy             # Numerical computation
h5py              # HDF5 I/O
opencv-python     # Image processing
torch             # Deep learning (for v2e frame interpolation)

# External tools
blender           # 3D rendering (Python API: bpy)
openeb            # Prophesee SDK (optional, for native format support)
v2e               # Video-to-events conversion

# Optional
poliastro         # Interactive astrodynamics
metavision-sdk    # Full Prophesee SDK (commercial features)
```

---

## References

### Event Camera Simulators
- ESIM: http://proceedings.mlr.press/v87/rebecq18a/rebecq18a.pdf
- v2e: https://github.com/SensorsINI/v2e
- DVS-Voltmeter: https://www.ecva.net/papers/eccv_2022/papers_ECCV/papers/136670571.pdf
- PECS: https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/06110.pdf
- V2CE: https://arxiv.org/abs/2309.08891
- ADV2E: https://arxiv.org/abs/2411.12250

### Prophesee / OpenEB
- OpenEB: https://github.com/prophesee-ai/openeb
- Metavision SDK Docs: https://docs.prophesee.ai/stable/index.html
- Video-to-Event Tutorial: https://docs.prophesee.ai/stable/tutorials/ml/dataset_creation/video_to_event_simulator.html
- Prophesee SSA: https://www.prophesee.ai/space-situational-awareness-event-based-vision/

### Space Rendering & Datasets
- SPIN: https://github.com/vpulab/SPIN
- SurRender: https://arxiv.org/abs/1810.01423
- SPARK: https://ar5iv.labs.arxiv.org/html/2104.05978
- SPEED+: https://arxiv.org/abs/2110.03101
- SPADES: https://orbilu.uni.lu/bitstream/10993/61821/1/ICRA_2024.pdf
- EBSSA: https://research-data.westernsydney.edu.au/published/ffe96150519311ecb15399911543e199/
- Ev-Satellites: https://arxiv.org/abs/2411.11233

### Orbital Mechanics & Star Catalogs
- Skyfield: https://rhodesmill.org/skyfield/earth-satellites.html
- SGP4: https://pypi.org/project/sgp4/
- Hipparcos: https://www.cosmos.esa.int/web/hipparcos/catalogues
- NASA SVS Tycho Skymap: https://svs.gsfc.nasa.gov/3572

### Sim-to-Real Gap
- Stoffregen et al. (ECCV 2020): https://ar5iv.labs.arxiv.org/html/2003.09078
- Event Quality Score: https://www.researchgate.net/publication/390892486

### Event Cameras in Space
- Star Tracking with Event Camera (CVPR 2019): https://arxiv.org/abs/1812.02895
- DARPA FENCE: https://www.darpa.mil/research/programs/fast-event-based-neuromorphic-camera-and-electronics
- AFIT Thesis: https://scholar.afit.edu/etd/4968/
