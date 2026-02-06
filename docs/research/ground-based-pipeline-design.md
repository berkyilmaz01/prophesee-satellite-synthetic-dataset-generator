# Ground-Based Synthetic Event Camera Dataset: Technical Design

## Key Insight

From ground-based telescopes, **all satellites and debris are unresolved point sources** (angular size ~1-10 milliarcseconds, far below the ~1-3 arcsecond seeing limit). They look identical to stars -- just moving. This means:

- No 3D rendering needed (no Blender)
- A satellite is a bright dot moving across a star field
- The only visual difference from stars is **motion**
- All physics can be modeled programmatically in Python

## Architecture: v2e Synthetic Input

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GROUND-BASED PIPELINE                            │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│  │  Star Catalog │  │  Orbital     │  │  Atmospheric Model       │  │
│  │  (Hipparcos/  │  │  Mechanics   │  │  - Kolmogorov turbulence │  │
│  │   Tycho-2)    │  │  (Skyfield/  │  │  - Scintillation         │  │
│  │              │  │   SGP4+TLE)  │  │  - Sky background        │  │
│  └──────┬───────┘  └──────┬───────┘  └────────────┬─────────────┘  │
│         │                 │                        │                │
│         └────────┬────────┴────────────────────────┘                │
│                  ▼                                                  │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │           satellite_pass.py (v2e synthetic input)          │     │
│  │                                                            │     │
│  │  For each timestep:                                        │     │
│  │    1. Place star fluxes at pixel positions                 │     │
│  │    2. Place satellite flux at current trajectory position  │     │
│  │    3. Convolve with atmospheric PSF (time-varying)         │     │
│  │    4. Add sky background + Poisson + read noise            │     │
│  │    5. Apply scintillation to star intensities              │     │
│  │    6. Return frame + timestamp                             │     │
│  └──────────────────────┬─────────────────────────────────────┘     │
│                         ▼                                          │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │              v2e DVS Pixel Model                           │     │
│  │  - Log-intensity tracking per pixel                        │     │
│  │  - Contrast threshold with per-pixel mismatch              │     │
│  │  - Leak events, shot noise, bandwidth limits               │     │
│  │  - Refractory period                                       │     │
│  └──────────────────────┬─────────────────────────────────────┘     │
│                         ▼                                          │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │              Output: Event Stream + Ground Truth           │     │
│  │  - Events: (x, y, polarity, timestamp) in .aedat / .hdf5  │     │
│  │  - Labels: satellite position, velocity, magnitude         │     │
│  │  - Metadata: telescope params, seeing, observer location   │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## Why Skip Blender?

| Aspect | Blender | Programmatic (Python) |
|--------|---------|----------------------|
| Speed | Minutes/frame (Cycles) | Milliseconds/frame (numpy) |
| Frame rate | Impractical at 1000+ fps | Trivial |
| Stellar photometry | Approximate | Exact (catalog magnitudes) |
| PSF control | Post-processing hack | Native (scipy/photutils) |
| Atmospheric turbulence | Not supported | aotools/GalSim |
| v2e integration | Render video, then convert | Direct synthetic input class |
| Reproducibility | Scene file + settings | Code + config |

Blender only makes sense for **resolved satellites** (ISS at close approach). For all standard SSA targets, programmatic generation is faster, more accurate, and simpler.

---

## Hardware Reference: Real Ground-Based Setups

### Ev-Satellites Setup (State of the Art, 2024)

| Parameter | Value |
|-----------|-------|
| Camera | Prophesee EVK4-HD (Sony IMX636) |
| Resolution | 1280 x 720 |
| Pixel pitch | 4.86 um |
| Mount | Planewave L-600 direct-drive alt-az |
| Telescope | ~600 mm effective focal length |
| Plate scale | 1.669 arcsec/pixel |
| FOV | 35.6' x 20.0' |
| Mode | Scan/slew (stars + satellites both streak) |
| Throughput | 800-1000 satellites/night |

### EBSSA Setup (First Public Dataset, 2020)

| Parameter | Value |
|-----------|-------|
| Camera | ATIS (304x240, 30um) and DAVIS240C (240x180, 18um) |
| Telescope | Various at DST facility, Edinburgh SA |
| Mode | Stare (no tracking) |
| Data | 572 labeled RSOs, 8+ hours |
| Format | .mat and .h5 event tuples |

### AFIT Setup (2021)

| Parameter | Value |
|-----------|-------|
| Camera | Prophesee Gen 3 |
| Lens | 85mm f/1.4 |
| Detection limit | 6.9 mV (HVGA), 9.8 mV (VGA) |

---

## What Satellites Look Like from Ground

### Angular Velocity

| Object / Geometry | Angular Speed |
|-------------------|---------------|
| LEO at zenith (~400 km, ISS) | ~4000 arcsec/s (~1.1 deg/s) |
| LEO at zenith (~800 km) | ~2000 arcsec/s |
| LEO near horizon | 130-235 arcsec/s |
| MEO (~20,000 km) | ~10-50 arcsec/s |
| GEO (~36,000 km) | ~0 arcsec/s (stationary) |

### Apparent Magnitude

| Object | Typical Magnitude |
|--------|------------------|
| ISS | -2 to -4 (very bright) |
| Starlink v2 mini | +4 to +7 |
| Typical LEO debris | +6 to +12 |
| GEO satellite | +10 to +15 |
| Faint GEO debris | +18 to +20 |

### Event Camera Appearance

- **Leading edge**: Positive (ON) events as brightness increases
- **Trailing edge**: Negative (OFF) events as brightness decreases
- Creates a characteristic **dipole pattern** in accumulated events
- Stars generate events from atmospheric **scintillation** (~100-900 Hz twinkling)
- Stars produce **doughnut-shaped PSFs** in event accumulation (contrast changes strongest at PSF edges)

### Event Rates

| Source | Events/second |
|--------|--------------|
| Dark sky background noise | ~50,000 |
| Difficult conditions (bright sky) | ~400,000 |
| Hot pixels | Dominant noise in sparse scenes |
| Bright LEO satellite | Dense, well-defined streaks |
| Faint GEO objects | May be at/below noise floor |

---

## Observation Constraints

### Visibility Window

Satellites are visible when: **satellite in sunlight AND observer in darkness**

- **LEO**: Observable ~1-3 hours after sunset and before sunrise (terminator window)
- **GEO**: Observable 6-10 hours/night (above Earth shadow cone, except near equinoxes)
- **Event camera advantage**: Daytime observation demonstrated (high dynamic range), extending window to ~12-16 hours

### Tracking Modes

| Mode | Stars | Satellite | Best For |
|------|-------|-----------|----------|
| **Scan/slew** | Streak | Streak (different direction) | Survey, high throughput |
| **Sidereal track** | Stationary (scintillation events only) | Streak | Traditional approach |
| **Satellite track** | Streak | Stationary (variable brightness) | Characterization, ~1.2 mag more sensitive |
| **Stare** (no tracking) | Drift | Different drift | Simple setup |

The Ev-Satellites dataset uses **scan/slew** for maximum throughput.

---

## Atmospheric Effects

### Seeing (Image Blurring)

| Site Quality | Fried parameter r0 | Seeing FWHM |
|-------------|--------------------|--------------------|
| Excellent (Paranal) | 20-40 cm | 0.35-0.5 arcsec |
| Good site | 10-20 cm | 0.5-1.0 arcsec |
| Average/urban | 5-7 cm | 1.5-3.5 arcsec |

Seeing FWHM = 0.98 * lambda / r0

### Scintillation

Stars twinkle at >100 Hz due to high-altitude turbulence. This generates events even from "stationary" tracked stars. Must be modeled as a multiplicative log-normal intensity fluctuation.

### Sky Background

| Condition | Surface Brightness (mag/arcsec^2) |
|-----------|-----------------------------------|
| Dark site, no moon | ~22 |
| Dark site, full moon | ~18-19 |
| Suburban | ~18-19 |
| Urban | ~16-17 |

---

## Frame Generation: Implementation Details

### Frame Rate Calculation

For v2e synthetic input, maximum inter-frame motion must be <= 1 pixel:

```
pixel_speed = angular_speed_arcsec_per_sec / plate_scale_arcsec_per_pixel
min_frame_rate = pixel_speed  [fps]
```

**Example (Ev-Satellites-like setup):**
- LEO at zenith: 4000 arcsec/s / 1.669 arcsec/px = 2397 px/s -> need ~2400 fps
- LEO near horizon: 200 arcsec/s / 1.669 arcsec/px = 120 px/s -> need ~120 fps
- Use adaptive dt based on current satellite speed

### Star Field Generation

```python
from skyfield.api import load
from skyfield.data import hipparcos

# Load Hipparcos catalog (118,218 stars)
with load.open(hipparcos.URL) as f:
    stars_df = hipparcos.load_dataframe(f)

# Filter to observable magnitude limit
stars_df = stars_df[stars_df['magnitude'] <= mag_limit]

# Convert RA/Dec -> alt/az for observer at time t
# Then alt/az -> pixel coordinates in camera FOV
# Flux from magnitude: flux = flux_0 * 10^(-0.4 * mag)
```

### Satellite Trajectory

```python
from skyfield.api import load, EarthSatellite

ts = load.timescale()
satellite = EarthSatellite(tle_line1, tle_line2, name, ts)

# Propagate over observation window
times = ts.utc(2024, 6, 15, 22, range(0, 600))  # 10 minutes
positions = satellite.at(times)

# Convert to observer-relative alt/az
observer = load('de421.bsp')['earth'] + observer_location
alt, az, distance = observer.at(times).observe(satellite).apparent().altaz()
```

### PSF Model

```python
import numpy as np
from scipy.signal import fftconvolve

def gaussian_psf(fwhm_pixels, size=None):
    """Seeing-limited Gaussian PSF."""
    sigma = fwhm_pixels / 2.355
    if size is None:
        size = int(6 * sigma) | 1
    y, x = np.mgrid[-size//2:size//2+1, -size//2:size//2+1]
    kernel = np.exp(-(x**2 + y**2) / (2 * sigma**2))
    return kernel / kernel.sum()

def moffat_psf(fwhm_pixels, beta=3.0, size=None):
    """Moffat PSF (more realistic extended wings)."""
    alpha = fwhm_pixels / (2 * np.sqrt(2**(1/beta) - 1))
    if size is None:
        size = int(6 * fwhm_pixels) | 1
    y, x = np.mgrid[-size//2:size//2+1, -size//2:size//2+1]
    kernel = (1 + (x**2 + y**2) / alpha**2)**(-beta)
    return kernel / kernel.sum()
```

### Scintillation Model

```python
def apply_scintillation(star_flux, airmass, aperture_m, wavelength_nm=550):
    """Apply scintillation to star intensities.

    Scintillation index sigma_I^2 ~ D^(-4/3) * sec(z)^3
    where D = aperture diameter, z = zenith angle.
    """
    # Young's approximation for scintillation index
    sigma_I = 0.09 * aperture_m**(-2/3) * airmass**1.75

    # Log-normal fluctuation
    fluctuation = np.random.lognormal(mean=0, sigma=sigma_I)
    return star_flux * fluctuation
```

### Noise Model

```python
def add_noise(frame, sky_background_adu, read_noise_e=5.0, gain=1.0):
    """Add realistic CCD/CMOS noise to frame."""
    # Add sky background
    frame = frame + sky_background_adu

    # Poisson noise (shot noise from photons)
    frame = np.random.poisson(np.clip(frame, 0, None).astype(int)).astype(np.float32)

    # Read noise (Gaussian)
    frame += np.random.normal(0, read_noise_e / gain, frame.shape)

    return np.clip(frame, 0, 255)
```

---

## v2e Integration

### Custom Synthetic Input Class

v2e supports a `--synthetic_input` flag that bypasses video loading and SuperSloMo interpolation entirely. We implement `base_synthetic_input` subclass:

```python
from v2ecore.base_synthetic_input import base_synthetic_input

class SatellitePass(base_synthetic_input):
    """Generates frames of a satellite pass as seen from a ground telescope."""

    def __init__(self, width=1280, height=720, ...):
        super().__init__(width, height, ...)
        # Load star catalog, compute satellite trajectory
        # Set up atmospheric model, PSF kernel
        self.dt = adaptive_dt  # based on satellite speed

    def next_frame(self):
        """Return (frame, timestamp) or None when done."""
        # 1. Place stars at pixel positions with flux values
        # 2. Place satellite at current position with flux
        # 3. Convolve with seeing PSF
        # 4. Apply scintillation to stars
        # 5. Add sky background + noise
        return frame, timestamp
```

### v2e Parameters for Space Scenes

```bash
python v2e.py \
    --synthetic_input=satellite_pass \
    --output_folder=output \
    --pos_thres=0.2 --neg_thres=0.2 \      # Contrast thresholds
    --sigma_thres=0.03 \                      # Per-pixel CT mismatch
    --leak_rate_hz=0.1 \                      # Background leak events
    --shot_noise_rate_hz=0.01 \               # Shot noise rate
    --dvs_aedat2=events.aedat \               # Output events
    --dvs_h5=events.h5 \                      # Alternative HDF5 output
    --dvs_vid=dvs_video.avi                   # Visualization
```

---

## Dataset Structure

```
dataset/
├── sequences/
│   ├── seq_0001/
│   │   ├── events.h5              # Event stream (x, y, p, t)
│   │   ├── events.aedat           # Alternative format
│   │   ├── labels.json            # Ground truth annotations
│   │   └── metadata.json          # Observation parameters
│   ├── seq_0002/
│   └── ...
├── catalog/
│   ├── tle_catalog.txt            # TLE data used
│   └── star_catalog.csv           # Star positions/magnitudes in FOV
└── config/
    ├── telescope.yaml             # Telescope parameters
    ├── atmosphere.yaml            # Atmospheric model parameters
    └── dvs.yaml                   # Event camera simulation parameters
```

### labels.json Format

```json
{
    "sequence_id": "seq_0001",
    "duration_s": 30.0,
    "objects": [
        {
            "id": 0,
            "name": "ISS (ZARYA)",
            "norad_id": 25544,
            "type": "satellite",
            "orbit": "LEO",
            "magnitude": -2.5,
            "trajectory": [
                {"t_us": 0, "x": 640.2, "y": 360.1, "az_deg": 180.5, "alt_deg": 45.2},
                {"t_us": 1000, "x": 641.8, "y": 359.4, "az_deg": 180.6, "alt_deg": 45.3}
            ]
        }
    ],
    "observation": {
        "observer_lat": -35.32,
        "observer_lon": 149.01,
        "observer_alt_m": 770,
        "utc_start": "2024-06-15T12:00:00Z",
        "tracking_mode": "scan",
        "scan_speed_deg_s": 0.5
    }
}
```

---

## Variation Axes for Dataset Diversity

To train robust models, vary these parameters across the dataset:

### Satellite Properties
- **Orbit type**: LEO (200-2000 km), MEO, GEO
- **Magnitude**: -4 to +15 (spanning ISS to faint debris)
- **Angular velocity**: 0 to 4000 arcsec/s
- **TLE source**: Real TLEs from CelesTrak for realistic trajectories
- **Multiple objects**: Simultaneous satellite passes, debris clusters

### Atmospheric Conditions
- **Seeing**: 0.5 to 4.0 arcsec
- **Scintillation strength**: Varies with airmass and turbulence
- **Sky background**: Dark site to urban (mag 16-22 /arcsec^2)
- **Clouds**: Partial obscuration (random opacity patches)

### Telescope Configuration
- **Focal length**: 200mm to 3200mm (wide survey to narrow tracking)
- **Aperture**: 200mm to 600mm
- **Tracking mode**: Sidereal, satellite-rate, scan/slew, stare
- **Scan speed**: 0 to 2 deg/s

### DVS Parameters
- **Contrast threshold**: 0.1 to 0.4 (mean), 0.02 to 0.05 (sigma)
- **Bias settings**: Conservative to aggressive (affects sensitivity by ~1.6 mag)
- **Hot pixel density**: 0 to 0.1% of pixels

---

## Implementation Phases

### Phase 1: Core Frame Generator
- Star field from Hipparcos catalog via Skyfield
- Single satellite trajectory from TLE via SGP4
- Static Gaussian PSF (fixed seeing)
- Poisson + read noise
- Output: numpy frames at adaptive frame rate

### Phase 2: v2e Integration
- Implement `base_synthetic_input` subclass
- Generate events with realistic DVS noise model
- Output in .aedat and .h5 formats
- Validate: events should show dipole satellite streaks + scintillation rings on stars

### Phase 3: Atmospheric Realism
- Time-varying PSF from Kolmogorov phase screens (aotools)
- Scintillation model per star
- Sky background gradient
- Airmass-dependent extinction

### Phase 4: Dataset Generation at Scale
- Batch generation across variation axes
- Ground truth annotation pipeline
- Quality validation (compare event statistics to Ev-Satellites / EBSSA)
- Multiple satellites per sequence
- Various tracking modes

### Phase 5: Validation
- Train a simple detection model on synthetic data
- Test on real Ev-Satellites / EBSSA data
- Measure sim-to-real gap
- Iterate on noise/atmosphere parameters

---

## Key Python Dependencies

```
# Core
numpy>=1.24
scipy>=1.10
h5py>=3.8

# Astronomy
skyfield>=1.46        # Star catalogs + satellite propagation
sgp4>=2.22           # TLE propagation
astropy>=5.3         # Coordinate transforms

# Event camera simulation
v2e                  # Video-to-events (pip install v2e)
# OR
openeb               # Prophesee SDK (optional)

# Atmospheric simulation
aotools>=1.0.6       # Kolmogorov phase screens

# Visualization
opencv-python>=4.8
matplotlib>=3.7

# Optional
photutils>=1.9       # PSF models
numba>=0.57          # JIT acceleration
galsim>=2.4          # Alternative atmosphere simulation
```

---

## References

- Ev-Satellites: https://arxiv.org/abs/2411.11233
- EBSSA: https://research-data.westernsydney.edu.au/published/ffe96150519311ecb15399911543e199/
- v2e: https://github.com/SensorsINI/v2e
- v2e synthetic input: https://github.com/SensorsINI/v2e/blob/master/v2ecore/base_synthetic_input.py
- Skyfield: https://rhodesmill.org/skyfield/
- AOtools: https://aotools.readthedocs.io/
- Prophesee SSA: https://www.prophesee.ai/space-situational-awareness-event-based-vision/
- Reducing Sim-to-Real Gap: https://ar5iv.labs.arxiv.org/html/2003.09078
- AFIT Thesis: https://scholar.afit.edu/etd/4968/
- Astrosite: https://www.westernsydney.edu.au/icns/astrosite
