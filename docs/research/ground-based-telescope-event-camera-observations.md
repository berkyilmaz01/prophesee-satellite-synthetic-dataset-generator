# Ground-Based Telescope Observations of Satellites Using Event Cameras (Prophesee)

## Detailed Practical Reference

This document consolidates specific, practical information on how ground-based telescopes observe satellites and space debris using Prophesee neuromorphic event-based cameras, drawn from published research, dataset documentation, and sensor specifications.

---

## Table of Contents

1. [Telescope Setups Used in Practice](#1-telescope-setups-used-in-practice)
2. [What Satellites Look Like from Ground](#2-what-satellites-look-like-from-ground)
3. [Atmospheric Effects on Event Cameras](#3-atmospheric-effects-on-event-cameras)
4. [Tracking Modes](#4-tracking-modes)
5. [Observation Windows](#5-observation-windows)
6. [The EBSSA Dataset Setup](#6-the-ebssa-dataset-setup)
7. [The Ev-Satellites Dataset Setup](#7-the-ev-satellites-dataset-setup)
8. [Event Rates](#8-event-rates)
9. [Prophesee Sensor Specs Relevant to Astronomy](#9-prophesee-sensor-specs-relevant-to-astronomy)

---

## 1. Telescope Setups Used in Practice

### 1.1 Astrosite Observatory (Western Sydney University / ICNS)

The Astrosite is a world-first neuromorphic-inspired mobile telescope observatory developed by the International Centre for Neuromorphic Systems (ICNS) at Western Sydney University. It won the 2019 Misha Mahowald Prize for Neuromorphic Engineering.

**Physical Structure:**
- Standard 20-foot shipping container with a sliding roof
- Scissor lift supporting a motorized mount with three bore-sighted telescopes
- Fully mobile -- can be deployed on ships, remote locations, or at permanent sites
- Fully automated as of 2024: self-deploys from sunset to sunrise depending on weather

**Telescopes (Original Configuration ~2019):**
- Three relatively small telescopes ranging from 10 to 16 inches aperture
- The original EBSSA dataset used a Meade LX200-series Schmidt-Cassegrain at a remote site in Edinburgh, South Australia

**Telescopes (Current Configuration ~2022-2024, Ev-Satellites Dataset):**
- Planewave L-600 direct-drive altazimuth mount (the "L600 Altazimuth telescope" referenced in the Ev-Satellites paper refers to the mount, not the OTA)
- L-600 mount specs: 300+ lb payload capacity, zero backlash (direct drive), 50 deg/s slew speed, 0.072 arcsec/tick encoder resolution, sub-arcsecond tracking accuracy
- The telescope OTA on this mount yields a pixel scale of 1.669 arcsec/px with the EVK4-HD, implying an effective focal length of ~600 mm (computed: 206.265 x 4.86 um / 1.669 = 600.5 mm)
- This short focal length is intentional for wide-field satellite survey work
- FOV: 35.6 x 20.0 arcminutes with the Gen 4 HD sensor (1280 x 720 pixels)

**Sensors Used:**
- Prophesee Generation 4 Metavision (EVK4-HD with Sony IMX636)
- iniVation DVXplorer
- Earlier work also used ATIS sensors and DAVIS240C sensors

**Nightly Throughput:**
- Captures data from an average of 800-1000 satellites per night

### 1.2 EOS 1.8m Telescope at Mount Stromlo (Canberra, Australia)

A neuromorphic event-based camera was fitted to Electro Optic Systems's 1.8-metre telescope at Mount Stromlo and used to observe 14 satellites in a variety of orbit regimes (LEO through GEO).

**Published Results:**
- Event-rate monitoring was used for satellite characterization
- Unique patterns in event-rate fluctuations allowed determination of satellite rotation rates
- Could constrain the range of possible satellite orientations
- Could differentiate materials responsible for bright specular reflections
- Published in the Journal of Spacecraft and Rockets (AIAA): "Neuromorphic Sensor Event-Rate Monitoring for Satellite Characterization"

### 1.3 Australian Defence Science and Technology (DST) Facility, Edinburgh, South Australia

**Hardware Setup (EBSSA Dataset, ~2019):**
- Robotic electro-optic telescope facility modified to support event-based sensors
- Two identical telescopes for ATIS and DAVIS event-based sensors, alongside a conventional FLI Proline PL4710 CCD camera
- A Meade LX200 was also used as a co-mounted telescope
- Officina Stellare RH200 telescope used for the conventional sensor configuration

### 1.4 AFIT (Air Force Institute of Technology) Setup

- Used COTS event-based cameras (Prophesee Gen 3) for orbit determination research
- With an 85 mm f/1.4 lens, measured detection limits of:
  - HVGA-format EBC: 6.9 visual magnitudes
  - VGA-format EBC: 9.8 visual magnitudes
  - At sky background of ~20.3 mV per square arcsecond

### 1.5 Devasthal Fast Optical Telescope (DFOT), India

- 1300 mm Cassegrain telescope at Aryabhatta Research Institute (ARIES)
- Used with a neuromorphic camera for evaluating high dynamic range astronomical observations
- Demonstrated simultaneous capture of faint and bright celestial sources without saturation
- Detected faint gas cloud structure of the Trapezium cluster during a full moon night

### 1.6 PlaneWave CDK24 + L-600 System (Reference Specs)

The CDK600 system (CDK24 OTA + L-600 mount) is a commercially available platform used for SSA:

| Parameter | Value |
|-----------|-------|
| Aperture | 610 mm (24 inches) |
| Focal ratio | f/6.5 |
| Focal length | 3,974 mm |
| Optical design | Corrected Dall-Kirkham (CDK) |
| Image circle | 70 mm (flat field) |
| Mount type | L-600 direct-drive Alt/Az |
| Slew speed | 50 deg/s |
| Encoder resolution | 0.072 arcsec/tick (18 million counts) |
| Tracking accuracy | Sub-arcsecond over 5 minutes |
| Pointing precision | ~2 arcseconds |
| Payload capacity | 300+ lbs |

**If paired with EVK4-HD (IMX636, 4.86 um pixel) at native focal length:**
- Pixel scale: 206.265 x 4.86 / 3974 = **0.252 arcsec/px**
- FOV: 1280 x 0.252 = 323 arcsec = **5.4 x 3.0 arcminutes**
- This is a very narrow FOV, suitable for targeted tracking but not wide-field survey

### 1.7 PlaneWave CDK14 (Reference Specs)

| Parameter | Value |
|-----------|-------|
| Aperture | 356 mm (14 inches) |
| Focal ratio | f/7.2 |
| Focal length | 2,563 mm |
| Optical design | Corrected Dall-Kirkham (CDK) |
| Image circle | 70 mm |
| Weight | ~48 lbs (OTA) |

**If paired with EVK4-HD at native focal length:**
- Pixel scale: 206.265 x 4.86 / 2563 = **0.391 arcsec/px**
- FOV: ~**8.3 x 4.7 arcminutes**

---

## 2. What Satellites Look Like from Ground

### 2.1 Angular Size (Physical Body)

Satellites and debris are overwhelmingly **unresolved point sources** as seen from ground-based telescopes. Their physical angular sizes are far below the atmospheric seeing limit (~1 arcsecond).

| Object | Altitude | Physical Size | Angular Size | Resolved? |
|--------|----------|---------------|-------------|-----------|
| ISS | ~400 km | 109 m | ~56 arcsec at zenith | Yes (largest orbiting object) |
| Starlink satellite | ~550 km | ~3.2 m | ~1.2 arcsec | Marginally, under excellent seeing |
| Typical LEO satellite | 400-2000 km | 1-10 m | 0.001-5 arcsec | Generally no |
| GEO satellite | ~35,786 km | 10-30 m | 0.06-0.17 milliarcsec | No (extreme point source) |
| Small debris (<10 cm) | Any | <0.1 m | Sub-milliarcsec | Never |

**Key Point:** For event camera observations, essentially all objects except the ISS appear as unresolved point sources. What the sensor detects is the total reflected flux, not any resolved structure.

### 2.2 Apparent Magnitude (Brightness)

| Object Category | Typical Magnitude Range | Visibility |
|----------------|------------------------|------------|
| ISS | up to -6 | Easily naked eye |
| Starlink v1.0 (original) | +4.6 to +5.0 | Naked eye under dark skies |
| Starlink VisorSat | +5.7 to +6.0 | Barely naked eye |
| OneWeb satellites | Fainter than Starlink | Binoculars/telescope |
| NOAA satellites (brightest) | ~+5.5 | Barely naked eye |
| Typical LEO satellite | +3 to +8 | Varies widely |
| GEO satellite (typical) | +11 to +14 | Small telescope (4+ inches) |
| GEO satellite (brightest, favorable geometry) | +5 to +6 | Binoculars |
| GEODSS detection limit (sidereal mode) | +17.7 | Professional ground station |
| GEODSS detection limit (target-track mode) | +18.9 | Professional ground station |
| Small debris at GEO | +18 to +20+ | Meter-class telescopes |

**IAU Recommendation:** Satellites should be no brighter than apparent V magnitude +7.0 for orbits at or below 550 km altitude (this threshold is currently not met by most commercial LEO constellations).

**Event Camera Detection Limits:**
- With 85 mm f/1.4 lens (COTS EBC): ~6.9-9.8 mV (depending on sensor generation)
- Sensitivity can be improved by ~1.6 mV (4.3x dimmer) through optimized bias configurations
- With larger apertures (telescope-mounted), significantly fainter objects are detectable

### 2.3 Angular Velocity (Apparent Motion Across the Sky)

This is the most critical parameter for event camera observations, as angular motion generates the brightness changes that produce events.

| Orbit | Altitude | Angular Velocity at Zenith | Angular Velocity Near Horizon |
|-------|----------|---------------------------|------------------------------|
| **LEO (ISS-like)** | ~400 km | ~4,000 arcsec/s (~1.1 deg/s) | ~235 arcsec/s (~0.065 deg/s) |
| **LEO (typical)** | ~800 km | ~2,000 arcsec/s (~0.55 deg/s) | ~130 arcsec/s (~0.036 deg/s) |
| **LEO range** | 200-2000 km | 200-1500 arcsec/s (survey data) | Variable |
| **MEO (GPS-like)** | ~20,200 km | ~15-30 arcsec/s | ~5-15 arcsec/s |
| **GEO** | ~35,786 km | ~0 arcsec/s (station-keeping) | ~0 arcsec/s |
| **GEO (drifting)** | ~35,786 km | ~0.1-15 arcsec/s | Variable |

**Key Scaling Law:** Angular velocity scales inversely with slant range. At zenith, the slant range equals the altitude, producing the fastest apparent motion. Near the horizon, the range is much larger, so motion appears slower.

**FOV Crossing Times (for Ev-Satellites setup with 35.6' FOV):**
- ISS at zenith (~4000 arcsec/s): crosses 35.6 arcmin FOV in ~0.53 seconds
- LEO satellite at 800 km zenith (~2000 arcsec/s): crosses in ~1.07 seconds
- LEO near horizon (~200 arcsec/s): crosses in ~10.7 seconds

**FOV Crossing Times (for CDK24 native setup with 5.4' FOV):**
- ISS at zenith: crosses in ~0.08 seconds (too fast for useful observation without tracking)
- LEO at 800 km near horizon: crosses in ~1.6 seconds

### 2.4 What a Satellite Looks Like in an Event Camera

In an event camera:
- **Moving objects generate events** because they cause brightness changes at the pixel level as they transit
- **Stationary objects generate no events** (except from scintillation noise)

**Satellite Appearance:**

1. **In sidereal tracking mode (telescope tracks stars):**
   - Stars appear stationary (no events from stars except scintillation-induced)
   - Satellites appear as **streaks of events** -- a line of ON (positive polarity) events at the leading edge and OFF (negative polarity) events at the trailing edge
   - The streak length depends on the satellite's angular velocity and the observation window
   - LEO satellites produce fast, bright streaks; GEO objects produce slow, faint point-like signals

2. **In scan/slew mode (telescope slewing at constant speed):**
   - Stars appear as streaks moving opposite to the slew direction
   - Satellites move in a different direction from stars (opposite to stars)
   - This is the mode used in the Ev-Satellites dataset

3. **In satellite tracking mode (telescope tracks the satellite):**
   - The satellite appears as a point source generating events from brightness fluctuations (glints, tumbling, phase angle changes)
   - Stars streak across the field
   - Event-rate monitoring in this mode reveals satellite rotation rates and material properties

**Event Polarity Pattern:**
- As a bright point source moves across a dark background, the leading edge of its PSF triggers **positive (ON) events** as pixels see increasing brightness
- The trailing edge triggers **negative (OFF) events** as brightness decreases
- This produces a characteristic **dipole pattern**: positive events in the direction of motion, negative events behind

**Accumulated Event Images:**
- When events are accumulated over a time window (e.g., 40 ms), satellites appear as short line segments against a noisy background
- In the Ev-Satellites dataset, a 15-second accumulated image shows stars as longer streaks and satellites as streaks in a different direction

---

## 3. Atmospheric Effects on Event Cameras

### 3.1 Atmospheric Seeing (Turbulence)

**Seeing (Fried Parameter r0):**
- Typical astronomical seeing: 0.5-3.0 arcseconds FWHM
- Good sites: ~0.5-1.0 arcsec
- Average sites: ~1.5-2.5 arcsec
- The seeing disk defines the effective PSF of a ground-based observation

**Impact on Event Cameras:**
- Atmospheric turbulence causes rapid (~100+ Hz) tip/tilt variations in the apparent position of point sources (stars, satellites)
- These positional fluctuations cause brightness changes at each pixel, generating events
- This is fundamentally different from frame cameras where turbulence produces a blurred time-averaged PSF

### 3.2 Stars Generate Events from Scintillation (Twinkling)

**Yes, stars absolutely do generate events in event cameras due to atmospheric scintillation.** This is a critical phenomenon for space imaging.

**Mechanism:**
- Atmospheric scintillation causes brightness fluctuations at rates typically exceeding 100 Hz
- Sirius can undergo nearly 900 brightness variations per second (~1 ms period)
- These fluctuations exceed the contrast threshold of event camera pixels, triggering events

**Doughnut-Shaped PSF (Key Finding):**
- A neuromorphic camera observing a star produces a **doughnut-shaped pattern** after event integration
- Mechanism: atmospheric tip/tilt causes rapid fluctuations in the star's centroid position. Contrast changes are most pronounced at the **edges** of the blurred Gaussian PSF profile, while the uniform central region produces fewer events
- The result is the **spatial derivative** of a conventional Gaussian intensity profile
- The flux obtained from this doughnut profile is proportional to that derived from the Gaussian profile (Yadav et al., arXiv:2503.15883, 2025)

**Implications for Satellite Detection:**
- Star scintillation events are a significant source of "signal" that must be distinguished from satellite events
- In the Ev-Satellites dataset, stars of varying brightness contribute events alongside background noise
- Noise filtering algorithms must preserve faint satellite events while managing star scintillation events

### 3.3 Sky Background

**Impact on Event Cameras:**
- In low-light conditions (dark sky, ~20-22 mag/arcsec^2), background noise is dominated by **photon shot noise** and **dark current leakage**
- In bright conditions (twilight, daytime, moonlit), **photon noise** dominates and event rates from background increase substantially
- The high dynamic range (>120 dB) of event cameras is what enables daytime observation -- the sensor is not saturated by the bright sky, and satellite contrast can still be detected

**Daytime Observation:**
- Event cameras can observe LEO satellites during daytime -- a capability that conventional CCD/CMOS cameras largely cannot achieve
- The asynchronous operation means only brightness *changes* are captured, inherently rejecting the static bright background
- This has been demonstrated by the Astrosite team and others

### 3.4 Event Camera as Atmospheric Turbulence Sensor

A 2025 MDPI paper (Sensors, 25(23), 7276) demonstrated that a Prophesee EVK4-HD (IMX636) can function as a **passive scintillometer**:
- Extracted 19 lightweight features from event data
- Trained XGBoost regressor to estimate refractive-index structure parameter (Cn^2)
- Achieved sub-40% relative error across turbulence measurements
- No external illumination or moving parts needed
- Tested over a 300m horizontal path

---

## 4. Tracking Modes

### 4.1 Sidereal Tracking (Stars Fixed, Satellites Streak)

**How it works:** The telescope mount compensates for Earth's rotation at the sidereal rate (15 arcsec/s), keeping stars stationary in the FOV.

**What the event camera sees:**
- Stars: Mostly stationary, events generated only from scintillation
- Satellites: Streak across the FOV at their relative angular velocity minus the sidereal rate
- Background: Random noise events from dark current and photon noise

**Advantages:**
- Standard mode for conventional optical SSA (e.g., GEODSS)
- Stars provide astrometric calibration references
- Easy to identify satellites as the only fast-moving events

**Disadvantages for event cameras:**
- A perfectly stationary star field generates few events (only from scintillation), so the telescope is mostly idle in terms of generating useful satellite data unless a satellite transits
- Narrow FOV means infrequent satellite crossings

**GEODSS Experience:** Sidereal tracking was the standard mode, detecting objects to magnitude ~17.7. Track-rate mode was later found to be more sensitive (magnitude ~18.9) because the signal is not spread across multiple pixels.

### 4.2 Satellite Tracking / Rate Tracking (Satellite Fixed, Stars Streak)

**How it works:** The mount tracks the satellite's predicted motion (from TLE data or real-time feedback), keeping the satellite centered in the FOV.

**What the event camera sees:**
- Satellite: Appears as a point source with brightness variations from tumbling, glinting, phase angle changes
- Stars: Streak across the FOV in the opposite direction

**Advantages:**
- All satellite flux concentrated on minimal pixels (maximizes SNR)
- Can detect fainter objects (GEODSS: ~1.2 magnitudes fainter than sidereal mode)
- Event-rate temporal analysis reveals satellite characteristics (rotation rate, material properties, shape)
- Used by the Astrosite team for satellite characterization at the EOS 1.8m telescope

**Disadvantages:**
- Requires a priori knowledge of satellite orbit (TLE data)
- Not suitable for uncued detection of unknown objects
- Mount must be capable of tracking at the satellite's angular rate

### 4.3 Leap-Frog / Scan Mode (Most Common for Event Camera SSA)

**How it works:** The telescope slews at a constant speed through the sky. Stars and satellites both produce events as they streak across the sensor.

**What the event camera sees:**
- Stars: Streak in one direction (opposite to the slew)
- Satellites: Streak in a **different direction** from stars (their orbital motion combined with the slew)
- This directional difference is the key discriminant for satellite detection

**This is the mode used for the Ev-Satellites dataset.** Each 30-second recording was obtained by slewing the telescope at a constant speed through the sky.

**Advantages:**
- Generates events from everything in the FOV (no idle pixels)
- Can discover uncued/unknown objects
- Large sky coverage per unit time
- Stars and satellites are distinguishable by their different motion directions
- Ideal for event cameras: maximizes the asynchronous advantage

**Disadvantages:**
- Signal spread across more pixels (lower per-pixel SNR)
- More complex data processing needed to separate sources
- Astrometric calibration requires motion-compensation algorithms

### 4.4 Stare Mode (No Tracking)

**How it works:** The telescope is pointed at a fixed sky position with no tracking. Stars drift through the FOV at the sidereal rate.

**What the event camera sees:**
- All objects (stars, satellites) generate events as they move across the sensor
- Stars move at ~15 arcsec/s (sidereal rate)
- Satellites move at their own angular rate

**Used in early EBSSA work:** The ATIS camera was used "whilst staring without any sidereal tracking."

### 4.5 Summary of Tracking Modes for Event Cameras

| Mode | Stars | Satellites | Best For | Used By |
|------|-------|-----------|----------|---------|
| **Sidereal** | Stationary (scintillation events only) | Fast streaks | Conventional SSA, astrometric reference | GEODSS, traditional |
| **Satellite tracking** | Streaks | Point source (variable) | Characterization, faint detection | Astrosite (EOS 1.8m), GEODSS TRM |
| **Scan/Slew** | Streaks (one direction) | Streaks (different direction) | Wide-area survey, uncued detection | **Ev-Satellites dataset**, Astrosite survey mode |
| **Stare** | Slow streaks (sidereal) | Fast streaks | Simple setup, uncued | Early EBSSA (ATIS camera) |

**Most common for event cameras:** The **scan/slew mode** appears to be the dominant operational mode for event camera SSA, as it maximizes the event camera's advantage (everything moves, generating events) and enables discovery of unknown objects. The Astrosite captures 800-1000 satellites per night using this approach.

---

## 5. Observation Windows

### 5.1 Fundamental Requirement: Satellite Sunlit, Observer in Darkness

For passive optical observation (reflected sunlight):
- The satellite must be **illuminated by the Sun** (above Earth's shadow cone)
- The ground observer must be in **darkness** (or at least twilight) for sufficient contrast

This creates the classic **terminator observation window**: the period around dawn and dusk when the ground station is in darkness but satellites at altitude are still (or already) in sunlight.

### 5.2 LEO Visibility Windows

**Twilight Windows (~1-3 hours after sunset / before sunrise):**
- LEO satellites (200-2000 km) pass into Earth's shadow relatively quickly after sunset at the observer's location
- Typical useful observation window: roughly 1-3 hours after sunset and 1-3 hours before sunrise
- Higher-altitude LEO objects (closer to 2000 km) remain illuminated longer
- Lower orbits (400 km) enter shadow sooner

**Mid-Summer at High Latitudes:**
- During summer months at high latitudes, LEO satellites can remain illuminated throughout the short "night"
- This extends the observation window significantly

**Dawn-Dusk SSO Satellites:**
- Satellites in dawn-dusk sun-synchronous orbits (SSO) ride the terminator permanently
- They are difficult to observe from ground: they typically pass overhead near midnight/noon, so either both observer and satellite are in sunlight, or both in darkness
- Event cameras' daytime capability partially addresses this (see below)

**Typical LEO Pass Duration:**
- A LEO satellite at ~800 km crosses the visible sky in about 5-10 minutes
- The sunlit portion of this pass (the observable window) may be shorter, depending on timing relative to sunset/sunrise

### 5.3 GEO Visibility Windows

**Much Longer Visibility:**
- GEO satellites are at 35,786 km altitude, far above Earth's shadow cone for most of the year
- They can be observed for most of the night (typically 6-10 hours per night)
- GEO objects enter Earth's shadow only around the equinoxes (spring/fall), for periods of up to ~72 minutes

**Best Seasons:**
- Around equinoxes, GEO satellites exhibit brightness changes as they enter/exit eclipse
- Around solstices, GEO objects may be continuously illuminated all night

### 5.4 Event Camera Advantage: Daytime Observation

**The single most transformative capability** of event cameras for SSA is daytime observation:
- Demonstrated by Astrosite and multiple research groups
- The >120 dB dynamic range means the bright daytime sky does not saturate the sensor
- Only brightness changes are captured, so the bright but static sky background is largely rejected
- This extends observation windows from ~2-6 hours (nighttime-only) to potentially ~12-16 hours per day
- For some satellites, there are more daytime passes than twilight passes over a ground station

**Limitation:** Although event cameras can detect satellites during the day, the elevated sky background increases noise event rates, and the achievable limiting magnitude is reduced compared to night observations.

### 5.5 Astrosite Operational Window

The automated Astrosite system operates **from sunset to sunrise**, capturing 800-1000 satellites per night. Daytime operation has been demonstrated but the primary automated survey mode is nighttime.

---

## 6. The EBSSA Dataset Setup

### 6.1 Overview

- **Full name:** Event-Based Space Situational Awareness (EBSSA) Dataset
- **Authors:** Afshar, Nicholson, van Schaik, Cohen (Western Sydney University, 2020)
- **Publication:** arXiv:1911.08730 / IEEE Sensors Journal
- **First publicly available event-based space imaging dataset**
- **Total acquisition time:** Over 8 hours
- **Recordings:** 236 separate recordings
- **Labeled RSOs:** 572 labeled resident space objects
- **Formats:** .mat files (one per recording) and .h5 file (all combined)
- **Labeled section:** 84 recordings + 84 label files
- **Unlabeled section:** 153 recordings

### 6.2 Hardware Configuration

**Location:** Australian Defence Science and Technology Group's research facility, Edinburgh, South Australia. Robotic electro-optic telescope facility modified for event-based sensors.

**Sensors Used:**

| Sensor | Resolution | Pixel Pitch | With 2000mm FL: FOV | With 2000mm FL: Pixel Scale |
|--------|-----------|-------------|---------------------|----------------------------|
| **ATIS** | 304 x 240 | 30 um | 15.66' x 12.36' | 3.10 arcsec/px |
| **DAVIS240C** | 240 x 180 | 18 um | 7.44' x 5.58' | 1.86 arcsec/px |

**Telescope(s):**
- Two identical telescopes for the ATIS and DAVIS sensors
- A Meade LX200 co-mounted alongside the primary telescope
- Conventional reference sensor: Officina Stellare RH200 + FLI Proline PL4710 CCD
- Both ATIS and DAVIS were observed simultaneously (co-collects)

**Meade LX200 Specifications (reference, depending on model):**
- 10-inch model: 254mm aperture, f/10, 2540mm focal length
- 16-inch model: 406mm aperture, f/10, 4060mm focal length
- Schmidt-Cassegrain / Advanced Coma-Free (ACF) optical design
- Fork-mounted altazimuth with computerized GoTo

### 6.3 Data Format

Each recording file contains a TD data structure as a list of events:
```
event = [x, y, p, t]
  x: pixel x-coordinate
  y: pixel y-coordinate
  p: polarity (ON/OFF)
  t: timestamp (microseconds)
```

### 6.4 Key Characteristics of the Data

- Space imaging is characterized by **sparse, simple-featured scenes** with **low SNR** and **very stable event rates**
- The primary challenge is **extraction of simple faint detections from a highly noisy random event stream**
- This is fundamentally different from terrestrial event-based imaging, which has complex feature-rich scenes at relatively high SNR
- Observed objects span LEO through GEO orbital regimes
- Observations were made during both **daytime and nighttime** (terminator) conditions without modification to the camera or optics
- Hot pixel filtering is critical but must be done carefully in these sparse scenes

### 6.5 Data Collection Notes

- Observations at multiple remote sites
- Multiple event-based sensors from multiple providers
- RSOs observed include satellites in various orbit regimes, planets, and stars
- The ATIS camera was used "whilst staring without any sidereal tracking" (stare mode)

---

## 7. The Ev-Satellites Dataset Setup

### 7.1 Overview

- **Full name:** Ev-Satellites (part of the "Noise Filtering Benchmark for Neuromorphic Satellites Observations")
- **Authors:** Sami Arja, Alexandre Marcireau, Nicholas Owen Ralph, et al. (Western Sydney University)
- **Publication:** arXiv:2411.11233 (November 2024)
- **Recordings:** 100 event files, each 30 seconds long
- **Recording period:** June 2022 to June 2024
- **Purpose:** Noise filtering benchmark with high-quality ground truth
- **Code:** https://github.com/samiarja/dvs_sparse_filter

### 7.2 Hardware Configuration

**Observatory:** Astrosite -- 20ft shipping container with sliding roof, scissor lift, motorized mount with three bore-sighted telescopes.

**Event Camera:**
- **Sensor:** Prophesee Gen 4 HD (Sony IMX636)
- **Camera:** EVK4-HD
- **Resolution:** 1280 x 720 pixels
- **Pixel pitch:** 4.86 um

**Telescope/Mount:**
- **Mount:** Planewave L-600 direct-drive altazimuth
- **Telescope OTA:** Mounted on the L-600 (the paper refers to "Planewave L600 Altazimuth telescope")
- **Effective focal length:** ~600 mm (back-calculated from pixel scale)
- **Pixel scale:** 1.669 arcsec/px
- **Field of view:** 35.6' x 20.0' (arcminutes)

**Important Note on Focal Length:** The pixel scale of 1.669 arcsec/px with a 4.86 um pixel pitch implies an effective focal length of ~600 mm (206.265 x 4.86 / 1.669). This is NOT the CDK24 at native focal length (which would give 0.252 arcsec/px). The Astrosite likely uses a shorter focal-length wide-field OTA on the L-600 mount for survey work, optimized for maximum sky coverage.

### 7.3 Data Collection Mode

- **Scan/slew mode:** Telescope slewing at a constant speed through the sky
- Stars appear to move in one direction (opposite to camera motion)
- Satellites move in the opposite direction of the stars
- This directional difference enables ground-truth labeling

### 7.4 Dataset Characteristics

- **100 recordings** selected for higher noise levels (to stress-test noise filtering)
- Each recording includes events from **stars of varying brightness** plus background noise
- **Noise sources vary** due to sky brightness, cloud cover, sensor bias settings, weather conditions
- **Hot pixels** are the most dominant noise type, artificially increasing event rates
- Dataset is publicly available

### 7.5 Automation

- Fully automated since 2024
- Self-deploys from sunset to sunrise
- Weather-dependent operation
- Captures 800-1000 satellites per night

---

## 8. Event Rates

### 8.1 Background Noise Event Rates

| Condition | Typical Event Rate | Notes |
|-----------|-------------------|-------|
| Low-light (dark sky) | ~50,000 EPS | Order-of-magnitude estimate for sparse scenes |
| Difficult low-light | Up to 400,000 EPS | Noise can reach this level under challenging conditions |
| DAVIS sensor (unfiltered, space) | ~30,000 EPS | BSIDAVIS during space observation |
| DAVIS sensor (filtered) | <5,000 EPS | After hot pixel + correlation filtering |
| Hot pixels | Dominant noise source | Can contribute majority of background events in sparse scenes |

**Noise Event Sources (from most to least significant in astronomy):**
1. **Hot pixels** -- Abnormally high-rate pixels; most dominant in sparse scenes
2. **Photon shot noise** -- Dominant in low-brightness conditions (dark sky)
3. **Dark current / leakage current** -- Temperature-dependent, causes spurious events
4. **Readout noise** -- Electronic noise from the sensor circuitry

### 8.2 Satellite Signal Event Rates

Specific per-satellite event rates are not widely published as single numbers because they depend heavily on:
- Satellite brightness (apparent magnitude)
- Angular velocity (determines how fast the PSF sweeps across pixels)
- Telescope aperture and focal length
- Sensor contrast threshold settings (bias)
- Atmospheric conditions

**Qualitative behavior:**
- A bright LEO satellite (magnitude +3 to +5) passing through the FOV at ~1000 arcsec/s will generate a dense, well-defined streak of events easily distinguishable from noise
- A faint GEO satellite (magnitude +12 to +14) produces sparse events that may be comparable to or below the noise floor
- Satellite specular glints can produce brief bursts of very high event rates (brightening by up to 10 magnitudes)

**Event Rate as Characterization Tool:**
- At the EOS 1.8m telescope, event-rate time series were analyzed for 14 satellites across multiple orbit regimes
- Unique patterns in event-rate fluctuations could determine rotation rates of non-stabilized satellites
- Could differentiate material types from specular reflection signatures

### 8.3 Data Bandwidth Comparison

- Event-based output achieves a **36x data reduction** compared to frame-based equivalent for the same observation
- This massive reduction enables:
  - Real-time processing on low-power hardware
  - Viable remote/autonomous stations with limited bandwidth
  - Continuous day-and-night observation without data storage overflow

### 8.4 Sensor Bias Trade-offs

- **Lower contrast threshold** = more sensitive to faint objects, but generates more noise events
- **Higher contrast threshold** = less noise, but faint satellites may be missed
- Optimized bias configurations can improve sensitivity by ~1.6 visual magnitudes (detecting objects ~4.3x dimmer) and track objects moving up to 6.6x faster than default/naive settings
- There is currently no standardized guidance for optimal EBC biases for SSA -- this is an active research area

---

## 9. Prophesee Sensor Specs Relevant to Astronomy

### 9.1 EVK4-HD (Sony IMX636) -- Primary Sensor for Current SSA Work

| Parameter | Specification |
|-----------|--------------|
| **Sensor** | Sony IMX636ES (co-developed with Prophesee) |
| **Architecture** | 3D stacked (photodiode layer on top of CMOS logic) |
| **Resolution** | 1280 x 720 pixels (0.92 MP) |
| **Pixel pitch** | 4.86 um |
| **Optical format** | 1/2.5" (diagonal 7.1 mm) |
| **Dynamic range** | >86 dB (5 lux - 100,000 lux); >120 dB (0.08 lux - 100,000 lux) |
| **Latency** | <100 us (at 1000 lux); microsecond timestamps |
| **Max event throughput** | Up to 1.066 Geps (billion events per second) peak |
| **System bandwidth** | 1.6 Gbps max |
| **Spectral response** | ~400-1000 nm (silicon-based; exact QE curve not publicly available) |
| **Minimum illumination** | 0.08 lux (low-light cut-off for nominal contrast sensitivity) |
| **Interface** | USB 3.0 |
| **Lens mount** | C/CS mount |
| **Weight** | 40 g |
| **Dimensions** | 30 x 30 x 36 mm |
| **Power** | <1.5 W (USB powered) |
| **Output format** | EVT 2.0, EVT 2.1, EVT 3.0 (via Metavision SDK / OpenEB) |
| **Operating temperature** | Specified in datasheet (requires account access) |

### 9.2 Spectral Response / Quantum Efficiency

**The detailed QE curve is not publicly available.** Prophesee and Sony restrict this data to:
- Purchasers of the EVK (access via Prophesee Knowledge Center)
- Authorized distributors (FRAMOS, RESTAR)
- Direct request to Sony Semiconductor

**What is known:**
- Silicon-based sensor: responsive from ~400 nm (violet) to ~1000 nm (near-IR)
- No sensitivity above 1000 nm (silicon cutoff)
- Back-side illuminated (BSI) via Sony's stacking process, suggesting decent QE (likely comparable to other Sony BSI sensors, perhaps 50-70% peak in the 500-600 nm range)
- The logarithmic response to light intensity means the sensor output voltage scales as log(photocurrent), which is fundamentally different from linear CCD/CMOS sensors

**Astronomy Implications:**
- The broad spectral response (400-1000 nm) means the sensor captures reflected sunlight from satellites across the visible and near-IR band
- No color discrimination without external filters (monochrome sensor)
- The unfiltered broad response improves total photon collection but prevents color photometry
- For accurate synthetic data generation, the spectral response must be modeled -- in the absence of the exact curve, using a typical Sony BSI CMOS response shape is a reasonable approximation

### 9.3 Lens Mount & Telescope Compatibility

**C/CS mount (standard for the EVK4-HD):**
- C mount: 1-inch (25.4 mm) thread diameter, 32 TPI, 17.526 mm flange-to-sensor distance
- CS mount: Same thread but 12.5 mm flange distance
- Compatible with C-mount telescope adapters (available from most telescope manufacturers)
- The EVK4-HD can accept "any C/CS mount compatible lens, from 8mm objective lens to microscopes' or telescopes' imaging ports"

**Connecting to a Telescope:**
- Most telescopes output through a standard 2" or 1.25" eyepiece port, or a T-thread (M42x0.75) back port
- A **C-mount to T-thread adapter** (or C-mount to 1.25" adapter) is needed
- The small 1/2.5" sensor format means the image circle from even small telescopes easily covers the sensor
- The ~7.1 mm diagonal sensor means vignetting is unlikely with any standard telescope

### 9.4 Comparison with Astronomy CCD/CMOS Sensors

| Feature | EVK4-HD (IMX636) | Typical Astro CCD (e.g., FLI PL4710) | Note |
|---------|------------------|--------------------------------------|------|
| Pixel pitch | 4.86 um | 13 um (KAF-4710) | Astro sensors have larger pixels for more photon collection |
| Resolution | 1280 x 720 | 2048 x 2048 | Astro sensors much higher resolution |
| Dynamic range | >120 dB | ~60-80 dB | Event camera far superior |
| Read noise | N/A (different paradigm) | ~8-15 e- | Not directly comparable |
| Dark current | Generates leak events | ~0.001-0.01 e-/pixel/s (cooled) | Astronomy sensors are cooled, event cameras typically not |
| Frame rate | Asynchronous (microsecond) | 0.1-30 fps | Event camera infinitely faster |
| Sensitivity | Limited by contrast threshold | Limited by photon count | Astro sensors currently more sensitive for faint objects |
| Data output | Sparse events | Full frames | Event camera 36x less data |
| Cooling | None (in EVK4-HD) | Thermoelectric to -40C or below | Cooling reduces dark current dramatically |

**Key Takeaway:** Current COTS event cameras are **less sensitive** than optimized astronomy cameras for detecting the faintest objects, but offer **far superior temporal resolution, dynamic range, and data efficiency**. They excel in a different part of the SSA problem space: continuous monitoring, fast-moving objects, daytime observation, and autonomous operation.

### 9.5 Earlier Sensors Used in SSA Research

| Sensor | Resolution | Pixel Pitch | Provider | Era |
|--------|-----------|-------------|----------|-----|
| ATIS | 304 x 240 | 30 um | Prophesee (Gen 1) | ~2017-2019 |
| DAVIS240C | 240 x 180 | 18.5 um | iniVation | ~2017-2021 (first in space March 2021) |
| DAVIS346 | 346 x 260 | 18.5 um | iniVation | ~2019-present |
| Prophesee Gen 3 (VGA) | 640 x 480 | 15 um | Prophesee | ~2020-2022 |
| Prophesee Gen 3 (HVGA) | 320 x 240 | 15 um | Prophesee | ~2020-2022 |
| **Prophesee Gen 4 HD (IMX636)** | **1280 x 720** | **4.86 um** | **Prophesee/Sony** | **2022-present** |
| GenX320 | 320 x 320 | N/A | Prophesee | 2024+ (compact, >140 dB DR, <50 mW) |

---

## References

### Astrosite & ICNS
- ICNS Astrosite: https://www.westernsydney.edu.au/icns/astrosite
- Astrosite brochure: https://www.westernsydney.edu.au/__data/assets/pdf_file/0007/1497004/RSCH3638_Astrosite_Brochure_v03.pdf
- Prophesee SSA page: https://www.prophesee.ai/space-situational-awareness-event-based-vision/

### EBSSA Dataset
- Dataset: https://research-data.westernsydney.edu.au/published/ffe96150519311ecb15399911543e199/
- Paper: Afshar et al., "Event-based Object Detection and Tracking for Space Situational Awareness," arXiv:1911.08730 / IEEE Sensors Journal, 2020
- ICNS Space Imaging: https://www.westernsydney.edu.au/icns/resources/reproducible_research3/publication_support_materials2/space_imaging

### Ev-Satellites Dataset
- Paper: Arja et al., "Noise Filtering Benchmark for Neuromorphic Satellites Observations," arXiv:2411.11233, 2024
- Code: https://github.com/samiarja/dvs_sparse_filter

### Satellite Characterization with Event Cameras
- "Neuromorphic Sensor Event-Rate Monitoring for Satellite Characterization," Journal of Spacecraft and Rockets (AIAA): https://arc.aiaa.org/doi/abs/10.2514/1.A35474
- "Astrometric calibration and source characterisation of the latest generation neuromorphic event-based cameras for space imaging," Astrodynamics, 2023: https://link.springer.com/article/10.1007/s42064-023-0168-2

### Detection Limits & Sensor Performance
- Oliver et al., "Commercial-off-the-shelf event-based cameras for space surveillance applications," Applied Optics, 2021: https://opg.optica.org/ao/abstract.cfm?uri=ao-60-25-g144
- "Demystifying Event-based Sensor Biasing to Optimize Signal to Noise for Space Domain Awareness," AMOS 2023: https://ui.adsabs.harvard.edu/abs/2023amos.conf..142M/abstract
- "Analysis of detection limits in Event-Based Cameras for Space Situational Awareness," AMOS 2023: https://ui.adsabs.harvard.edu/abs/2023amos.conf..197W/abstract

### Neuromorphic Cameras in Astronomy
- Yadav et al., "Neuromorphic Cameras in Astronomy: Unveiling the Future of Celestial Imaging Beyond Conventional Limits," arXiv:2503.15883, 2025
- "Event-Based Camera Modeling for Atmospheric Turbulence Prediction," MDPI Sensors 25(23), 2025: https://www.mdpi.com/1424-8220/25/23/7276

### Event Camera Sensor Specifications
- Prophesee EVK4-HD: https://www.prophesee.ai/event-camera-evk4/
- Sony IMX636 sensor: https://docs.prophesee.ai/stable/hw/sensors/imx636.html
- IMX636 product brief: https://www.prophesee.ai/wp-content/uploads/2024/09/IMX636-AAMR-C-Product-Brief-2024.pdf
- Sony IMX636 flyer: https://www.sony-semicon.com/files/62/flyer_industry/IMX636-AAMR-Flyer.pdf

### PlaneWave Telescope Systems
- CDK24 OTA: https://planewave.com/products/cdk24-ota/
- CDK600 System: https://planewave.com/products/cdk600/
- CDK14 OTA: https://planewave.com/products/cdk14-ota/
- L-600 Mount: https://planewave.com/products/l-600-telescope-mount/

### Traditional Space Surveillance (Context)
- GEODSS: https://mostlymissiledefense.com/2012/08/20/space-surveillance-sensors-geodss-ground-based-electro-optical-deep-space-surveillance-system-august-20-2012/
- Geostationary satellite observation: https://www.satobs.org/geosats.html
- Satellite brightness: https://www.satobs.org/magnitude.html

### Satellite Observation Basics
- Observing satellites: https://satelliteobservation.net/2017/04/20/observing-satellites/
- N2YO satellite tracker: https://www.n2yo.com/
- LEO angular velocity calculator: https://www.satnow.com/calculators/leo-satellite-angular-velocity-calculator
- Starlink brightness: https://arxiv.org/pdf/2101.00374

### AFIT Research
- Orbit Determination with Event-Based Cameras: https://scholar.afit.edu/etd/5544/
- Satellite Tracking with Neuromorphic Cameras: https://scholar.afit.edu/cgi/viewcontent.cgi?article=5971&context=etd
