# Prophesee Satellite Synthetic Dataset Generator

Synthetic dataset generation pipeline for training space situational awareness (SSA) models using Prophesee event-based cameras (neuromorphic vision sensors).

## Motivation

Real event camera data for space domain awareness is extremely scarce. This project generates realistic synthetic event streams of satellites and space debris as seen by **ground-based telescopes** equipped with Prophesee sensors, enabling training of detection and tracking models.

## Key Insight

From ground-based telescopes, all satellites and debris are **unresolved point sources** (angular size ~1-10 mas, far below the ~1-3" seeing limit). No 3D rendering is needed -- we generate frames programmatically in Python and convert them to events using v2e.

## Pipeline Overview

```
Star Catalog (Hipparcos/Tycho-2)  ──┐
Satellite TLEs (SGP4/Skyfield)    ──┼──> Frame Generator ──> v2e DVS Model ──> Event Stream
Atmosphere (seeing, scintillation) ──┘    (numpy/scipy)       (noise, CT)       (.h5/.aedat)
```

### Stages

1. **Star Field** -- Load Hipparcos/Tycho-2 catalog, convert to pixel positions and fluxes for observer location/time
2. **Satellite Trajectory** -- Propagate real TLEs with SGP4, compute apparent sky track through camera FOV
3. **Frame Generation** -- Place point sources, convolve with atmospheric PSF, add scintillation + sky background + noise
4. **Event Simulation** -- Feed frames to v2e synthetic input with realistic DVS noise model (contrast threshold mismatch, leak events, bandwidth)
5. **Annotation** -- Ground truth from orbital mechanics (positions, velocities, magnitudes, object classes)

## Research

- [General synthetic dataset research](docs/research/synthetic-dataset-generation-research.md) -- Event camera simulators, Prophesee tools, space rendering, existing datasets
- [Ground-based pipeline design](docs/research/ground-based-pipeline-design.md) -- Technical design for the ground-based approach, atmospheric modeling, v2e integration, dataset structure

## Key Dependencies

```
numpy, scipy      # Frame generation, PSF convolution
skyfield, sgp4     # Star catalogs + satellite orbit propagation
astropy            # Coordinate transformations
v2e                # Video-to-events conversion (DVS simulation)
aotools            # Atmospheric turbulence (Kolmogorov phase screens)
h5py               # HDF5 I/O for event data
opencv-python      # Visualization
```

## License

MIT
