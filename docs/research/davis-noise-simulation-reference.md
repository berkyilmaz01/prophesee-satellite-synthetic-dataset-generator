# DAVIS Sensor Simulation & Realistic Noise Reference

## DAVIS Sensor Specifications

### DAVIS240C

| Parameter | Value |
|-----------|-------|
| Resolution | 240 x 180 |
| Pixel Pitch | 18.5 um |
| DVS Dynamic Range | ~130 dB |
| Contrast Threshold (ON) | ~14% |
| Contrast Threshold (OFF) | ~22% |
| DVS Latency | ~3 us |
| Max Event Rate | 12 Mev/s |

### DAVIS346

| Parameter | Value |
|-----------|-------|
| Resolution | 346 x 260 |
| Pixel Pitch | 18.5 um |
| DVS Dynamic Range | ~120 dB |
| Contrast Sensitivity ON (50% pixels) | 14.3% |
| Contrast Sensitivity OFF (50% pixels) | 22.5% |
| APS Frame Rate | 40 FPS |
| APS Dark Signal | 18,000 e-/s |
| APS Readout Noise | 55 e- |
| Max Event Throughput | 12 Mev/s |
| DVS Temporal Resolution | 1 us |
| IMU | 6-axis, up to 8 kHz |

### DAVIS vs Prophesee (Key Differences for Simulation)

| Feature | DAVIS (iniVation) | Prophesee (IMX636/Gen4) |
|---------|-------------------|-------------------------|
| Architecture | APS + DVS in **same pixel** (simultaneous frames + events) | Pure DVS only |
| Pixel pitch | 18.5 um (more photons/pixel) | 4.86 um (higher resolution) |
| Max resolution | 346x260 | 1280x720 |
| On-chip filtering | **None** -- raw noise preserved | ESP: Anti-Flicker, Trail Filter, Rate Control |
| Noise in output | Noisier raw data (must denoise in software) | Pre-filtered on-chip |
| Data format | AEDAT (.aedat) | EVT (.raw, EVT2/EVT3) |

**Implication**: When simulating DAVIS, model the **full raw noise**. Prophesee sensors have already suppressed some noise on-chip.

---

## Event Camera Simulator Comparison (Noise Focus)

| Noise Source | v2e | ESIM | DVS-Voltmeter | ICNS/IEBCS | PECS | SENPI |
|-------------|-----|------|---------------|------------|------|-------|
| Shot noise (Poisson) | Yes (intensity-dep.) | No | Yes (photon) | Yes (Poisson) | Yes | Yes |
| Dark current / leak events | Yes | No | Yes (Idark) | Yes | Implicit | No |
| Hot pixels | Yes (min thresh clamp) | No | Implicit | Yes | No | No |
| Per-pixel threshold mismatch | Yes (Gaussian) | Yes (Gaussian) | Implicit | Yes | Implicit | Implicit |
| Bandwidth limitation | Yes (cutoff_hz) | No | Yes (circuit RC) | Yes (latency) | Yes | No |
| Refractory period | Yes | No | Implicit | No | Implicit | No |
| Temporal jitter | Yes (photoreceptor) | No | Yes (stochastic) | Yes | Yes (BiLIF) | Yes |

**Winner for realistic DAVIS simulation: v2e** -- most comprehensive noise model with explicit per-parameter control.

---

## v2e: Complete Noise Parameter Reference

### All DVS Noise Flags

```bash
# === Threshold Parameters ===
--pos_thres FLOAT         # ON threshold (log_e intensity). Default: 0.2
--neg_thres FLOAT         # OFF threshold (log_e intensity). Default: 0.2
--sigma_thres FLOAT       # Per-pixel threshold std-dev. Default: 0.03

# === Bandwidth ===
--cutoff_hz FLOAT         # Photoreceptor lowpass 3dB cutoff (Hz). Default: 300
                          # 0 = infinite bandwidth (disable)

# === Noise ===
--leak_rate_hz FLOAT      # Leak event rate per pixel (Hz). Default: 0.01
--leak_jitter_fraction FLOAT  # Jitter on leak interval. Default: 0.1
--shot_noise_rate_hz FLOAT    # Shot noise ON+OFF rate in darkest pixels (Hz). Default: 0.001
--noise_rate_cov_decades FLOAT  # Log-normal CoV of noise rates across pixels. Default: 0.1
--photoreceptor_noise     # Flag: inject Gaussian noise into log-photoreceptor

# === Timing ===
--refractory_period FLOAT # Dead time after event (seconds). Default: 0.0005

# === Presets ===
--dvs_params {clean,noisy,None}

# === Reproducibility ===
--dvs_emulator_seed INT   # Fixed seed (>0). Default: 0 (random)
```

### Presets

**`clean`** (ideal, no noise):
```
pos/neg_thres=0.2, sigma_thres=0.02, cutoff_hz=0, leak=0, shot=0
```

**`noisy`** (low-light, heavy noise):
```
pos/neg_thres=0.2, sigma_thres=0.05, cutoff_hz=30, leak=0.1, shot=5.0
```

### Realistic DAVIS346 Parameters for Space Observation

**Night sky, normal conditions:**
```bash
python v2e.py \
  --dvs346 \
  --pos_thres=0.143 --neg_thres=0.225 \
  --sigma_thres=0.03 \
  --cutoff_hz=200 \
  --leak_rate_hz=0.05 \
  --shot_noise_rate_hz=0.01 \
  --refractory_period=0.0005 \
  --leak_jitter_fraction=0.1 \
  --noise_rate_cov_decades=0.1
```

**Low-light / challenging conditions:**
```bash
python v2e.py \
  --dvs346 \
  --pos_thres=0.143 --neg_thres=0.225 \
  --sigma_thres=0.05 \
  --cutoff_hz=15 \
  --leak_rate_hz=0.1 \
  --shot_noise_rate_hz=5.0 \
  --refractory_period=0.001 \
  --photoreceptor_noise \
  --noise_rate_cov_decades=0.3
```

Note: Asymmetric thresholds (0.143/0.225) come from the DAVIS346 datasheet (14.3% ON, 22.5% OFF at 50% pixel response).

---

## Noise Explanation: What Each Source Does

### 1. Per-Pixel Contrast Threshold Mismatch (`sigma_thres`)

Manufacturing variation causes each pixel to have a slightly different threshold.
```
theta(x) ~ N(theta_mean, sigma_thres)
```
Effect: Some pixels fire too easily (potential hot pixels), others are less sensitive. Creates fixed-pattern noise in the event stream.

### 2. Photoreceptor Bandwidth (`cutoff_hz`)

The photoreceptor circuit acts as a lowpass filter. Cutoff frequency is **proportional to light intensity** -- at low light, bandwidth drops dramatically.

Effect at low light: Motion blur in events, reduced event count per edge, increased timing jitter. At `cutoff_hz=30`, the sensor barely responds to fast motion.

### 3. Leak Events (`leak_rate_hz`)

Junction leakage in the reset switch continuously decreases the stored reference voltage. Eventually triggers a spontaneous ON event even without real brightness change.

Effect: Uniform rain of ON events across the sensor. Rate increases with temperature.

### 4. Shot Noise (`shot_noise_rate_hz`)

Random ON/OFF events from electronic noise in the comparator circuit. **Intensity-dependent**: darker pixels have more noise (lower SNR).
```
rate(x,t) = shot_noise_rate_hz * (1 - 0.75 * brightness(x,t))
```
Effect: Random scattered events, worse in dark regions of the image.

### 5. Refractory Period (`refractory_period`)

After firing, a pixel cannot fire again for this duration.

Effect: Maximum event rate is capped at ~1/refractory_period per pixel. Prevents runaway firing. Typical: 0.5ms.

### 6. Hot Pixels

Pixels with abnormally low thresholds that fire constantly. Modeled implicitly by clamping minimum threshold to 0.01 in v2e.

Effect: Fixed spatial pattern of high-rate pixels. Dominant noise source in sparse astronomical scenes.

---

## Frame-Stage Libraries (Before Event Simulation)

### aotools -- Atmospheric Turbulence

```python
import aotools

# Time-evolving Kolmogorov phase screen
phase_screen = aotools.PhaseScreenVonKarman(
    nx_size=512,       # grid size
    pixel_scale=0.01,  # m/pixel
    r0=0.2,            # Fried parameter (m) -- smaller = worse seeing
    L0=25.0            # outer scale (m)
)

# Advance in time (wind-driven evolution)
phase_screen.add_row()

# Generate PSF from phase screen
pupil = aotools.circle(128, 256)
# ... FFT to get instantaneous PSF
```

### GalSim -- Full Astronomical Image Simulation

```python
import galsim

# Multi-layer atmospheric PSF
atm = galsim.Atmosphere(
    screen_size=30.0,           # meters
    altitude=[0, 5, 10],        # km (turbulence layers)
    speed=[10, 20, 30],         # m/s (wind)
    r0_500=0.2                  # Fried parameter at 500nm
)
psf = atm.makePSF(lam=550, exptime=0.001, diam=0.6)

# Draw a star
star = galsim.DeltaFunction(flux=50000)
final = galsim.Convolve(star, psf)
image = final.drawImage(nx=64, ny=64, scale=1.669)  # arcsec/pixel

# Poisson + Gaussian noise
image.addNoise(galsim.PoissonNoise())
image.addNoise(galsim.GaussianNoise(sigma=5.0))
```

### photutils -- PSF Models

```python
from photutils.psf import ImagePSF
import numpy as np

# Moffat PSF (better than Gaussian for astronomical seeing)
y, x = np.mgrid[-25:26, -25:26]
fwhm = 3.0  # pixels
beta = 3.5
alpha = fwhm / (2 * np.sqrt(2**(1/beta) - 1))
psf_data = (1 + (x**2 + y**2) / alpha**2)**(-beta)
psf_data /= psf_data.sum()
psf_model = ImagePSF(psf_data)
```

### scikit-image -- Quick Noise

```python
from skimage.util import random_noise
noisy = random_noise(frame, mode='poisson')  # Poisson photon noise
noisy = random_noise(frame, mode='gaussian', var=0.01)  # Read noise
```

---

## Two-Stage Noise Architecture: Avoiding Double-Counting

### The Rule

| Noise Source | Stage 1 (Frame Gen) | Stage 2 (v2e) | Why |
|-------------|---------------------|---------------|-----|
| Atmospheric turbulence PSF | **YES** (aotools/GalSim) | No | Affects light before sensor |
| Scintillation (twinkling) | **YES** (log-normal on star flux) | No | Atmospheric, not electronic |
| Photon shot noise | **YES** (Poisson on signal) | Use LOW `shot_noise_rate_hz` | Photon noise creates real intensity changes the DVS pixel responds to |
| Sky background | **YES** (Poisson on sky level) | No | Real photons reaching sensor |
| Optical PSF / diffraction | **YES** | No | Optics before sensor |
| Threshold mismatch | No | **YES** (`sigma_thres`) | DVS circuit property |
| Bandwidth limitation | No | **YES** (`cutoff_hz`) | DVS circuit property |
| Leak events | No | **YES** (`leak_rate_hz`) | DVS circuit property |
| DVS electronic noise | No | **YES** (`shot_noise_rate_hz`, small) | Comparator/amplifier noise |
| Refractory period | No | **YES** (`refractory_period`) | DVS circuit property |
| Hot pixels | No | **YES** (implicit in `sigma_thres`) | Manufacturing defect |
| Read noise | No | No | DVS has no traditional readout noise |

### Critical Rule

If your frames already have Poisson photon noise:
- Set `shot_noise_rate_hz` **LOW** (0.001-0.01) -- only electronic contribution
- Do **NOT** enable `--photoreceptor_noise` -- would double-count

If your frames are clean (no noise):
- Set `shot_noise_rate_hz` **HIGH** (1.0-5.0) -- must model everything
- Enable `--photoreceptor_noise`

---

## Additional Simulators Worth Knowing

### ICNS/IEBCS
- https://github.com/neuromorphicsystems/IEBCS
- Extends ESIM with threshold mismatch, leak, temporal noise from calibrated distributions
- From the Western Sydney team that built the Astrosite observatory

### SENPI (2025)
- https://github.com/joeg18/senpi_ebi
- **Fully differentiable** event camera simulator in PyTorch
- Can optimize simulation parameters jointly with downstream neural networks

### DVS-Voltmeter
- https://github.com/Lynn0306/DVS-Voltmeter
- Circuit-level physics: photocurrent -> log voltage -> comparator
- More realistic timestamp distributions than v2e
- Models shot noise from photon arrival, dark current, thermal noise

### Prophesee OpenEB EventSimulator
- Same parameter set as v2e (Cp, Cn, sigma_threshold, cutoff_hz, leak_rate_hz, shot_noise_rate_hz)
- GPU-accelerated, native Prophesee format output
- `metavision_core_ml.video_to_event.simulator.EventSimulator`

---

## References

- v2e paper: https://openaccess.thecvf.com/content/CVPR2021W/EventVision/papers/Hu_v2e_From_Video_Frames_to_Realistic_DVS_Events_CVPRW_2021_paper.pdf
- v2e code: https://github.com/SensorsINI/v2e
- DVS-Voltmeter: https://github.com/Lynn0306/DVS-Voltmeter
- ICNS/IEBCS: https://github.com/neuromorphicsystems/IEBCS
- PECS (ECCV 2024): https://github.com/lanpokn/PECS_trail_version
- SENPI: https://github.com/joeg18/senpi_ebi
- aotools: https://aotools.readthedocs.io/
- GalSim: https://github.com/GalSim-developers/GalSim
- photutils: https://photutils.readthedocs.io/
- DAVIS240 datasheet: https://inivation.com/wp-content/uploads/2019/08/DAVIS240.pdf
- DAVIS346 datasheet: https://inivation.com/wp-content/uploads/2019/08/DAVIS346.pdf
- Prophesee OpenEB video_to_event: https://docs.prophesee.ai/stable/api/python/core_ml/video_to_event.html
- DVS pixel tutorial (CVPR 2023): https://tub-rip.github.io/eventvision2023/papers/2023CVPRW_Shining_light_on_the_DVS_pixel_A_tutorial.pdf
