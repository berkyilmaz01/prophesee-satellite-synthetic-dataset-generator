# Prophesee Satellite Synthetic Dataset Generator

Synthetic dataset generation pipeline for training space situational awareness (SSA) models using Prophesee event-based cameras (neuromorphic vision sensors).

## Motivation

Real event camera data for space domain awareness is extremely scarce. This project generates realistic synthetic event streams of satellites and space debris as seen by ground-based telescopes equipped with Prophesee sensors, enabling training of detection and tracking models.

## Pipeline Overview

```
Orbital Mechanics (Skyfield/SGP4)  ──┐
Star Field (Hipparcos/Tycho-2)     ──┼──> Blender Rendering ──> Event Camera Sim ──> Dataset
Satellite 3D Models                ──┘    (Cycles / PBR)        (v2e / OpenEB)       (EVT2/HDF5)
```

### Stages

1. **Scene Definition** -- Generate realistic satellite/debris trajectories from TLE data, define observer telescope, select targets
2. **Rendering** -- Physically-based rendering in Blender with solar illumination, Earth albedo, material BRDFs, star backgrounds
3. **Event Simulation** -- Convert rendered frames to event streams using v2e or OpenEB with realistic noise models
4. **Annotation** -- Ground truth labels from orbital mechanics (positions, velocities, bounding boxes, object classes)

## Research

See [docs/research/synthetic-dataset-generation-research.md](docs/research/synthetic-dataset-generation-research.md) for the full research survey covering:

- Event camera simulators (ESIM, v2e, DVS-Voltmeter, PECS, V2CE)
- Prophesee/OpenEB tools for synthetic data
- Space scene rendering approaches
- Orbital mechanics and star field simulation
- Existing datasets (SPADES, EBSSA, Ev-Satellites, SPARK)
- Best practices for sim-to-real transfer

## Key Dependencies

```
skyfield          # Orbital mechanics + star catalogs
sgp4              # TLE propagation
astropy           # Coordinate transformations
numpy             # Numerical computation
h5py              # HDF5 I/O
opencv-python     # Image processing
torch             # Deep learning (v2e frame interpolation)
bpy               # Blender Python API
```

## License

MIT
