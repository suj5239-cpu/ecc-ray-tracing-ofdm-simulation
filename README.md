# ECC 3D Scene Ray Tracing and OFDM BER Simulation

This project builds an approximate 3D radio environment around **Ewha Campus Complex (ECC)** from OpenStreetMap building footprints and uses Sionna RT to investigate multipath propagation and OFDM-QPSK bit error rate.

The workflow connects geographic data to a communication-link simulation:

**OpenStreetMap → 3D building meshes → ray tracing → channel response → OFDM-QPSK → BER**

## Implementation

The project is implemented in [`Lec7_ECC_scene.ipynb`](Lec7_ECC_scene.ipynb). It includes:

- Conversion of geographic coordinates to approximate local coordinates in meters.
- Extrusion of building footprints into 3D meshes.
- Scene construction with concrete building materials and wet-ground material.
- Ray tracing, a spatial path-gain map, and channel impulse-response visualization.
- Conversion of the ray-traced channel into a 64-subcarrier frequency response.
- BER comparison between AWGN and the normalized ECC channel.

The saved notebook execution generated **73 building meshes**. Building heights use OSM height metadata where available, otherwise floor counts multiplied by 3 m, or a default height of 10 m.

## Example Outputs

### Ray-Traced Path-Gain Map

![Ray-traced path-gain map around ECC](coverage.png)

### Building Footprints and Transmitter/Receiver Locations

![ECC geometry and transmitter/receiver placement](ecc_scene/scene_verification_map.png)

The notebook also contains the channel impulse response, frequency response, and BER comparison plots.

## Link Simulation

The transmitter maps bits to QPSK symbols, arranges them into an OFDM grid, and performs an IFFT with cyclic-prefix insertion.

For channel evaluation, the implementation removes the prefix, returns to the frequency domain, and applies the channel response separately to each subcarrier. It then adds complex Gaussian noise and uses channel-aware one-tap equalization.

For repetition-coded transmission, each information bit is repeated three times. The receiver sums the corresponding equalized decision values before making a bit decision.

Four curves are compared:

- Theoretical uncoded QPSK in AWGN.
- Simulated uncoded QPSK in AWGN.
- Repetition-coded QPSK in AWGN.
- Repetition-coded QPSK over the normalized ECC ray-tracing channel.

In the saved plot, the uncoded AWGN simulation closely follows the theoretical curve. The repetition-coded AWGN result is also close to that curve under the information-bit energy normalization used here. The ECC channel shows higher BER over part of the evaluated range.

## Simulation Settings

| Setting | Value |
| --- | --- |
| Scene center | 37.56101° N, 126.94614° E |
| OSM query distance | 250 m |
| Carrier frequency | 3.5 GHz |
| Antennas | Single-element, vertically polarized isotropic Tx/Rx |
| Tx / Rx height | 20 m / 10 m |
| Point-to-point ray tracing | LoS and specular reflections, maximum depth 5 |
| Radio-map cell size | 5 m × 5 m |
| OFDM grid | 64 subcarriers × 14 symbols |
| Subcarrier spacing | 30 kHz |
| Cyclic prefix | 8 samples |
| Modulation | QPSK |
| Repetition factor | 3 |
| Eb/N0 range | −4 to 12 dB |
| Trials per condition and Eb/N0 point | 2,000 |

## Running the Notebook

The saved execution records **Sionna 2.0.1**, **PyTorch 2.9.1+cu126**, and an **NVIDIA GeForce RTX 4090**, using the `cuda_ad_mono_polarized` Mitsuba variant.

In a compatible Python and CUDA environment, install the dependencies:

```bash
python -m pip install "sionna==2.0.1" osmnx trimesh triangle matplotlib pandas scipy shapely torch jupyterlab
```

Open `Lec7_ECC_scene.ipynb` from the repository root and run the cells in order.

The first sections query OpenStreetMap and regenerate `ecc_scene/scene.xml` and its meshes. Later sections compute propagation paths, render the path-gain map, and evaluate BER.

Internet access is required for the OSM query. Rerunning the scene-generation cells can overwrite the included assets, and updated OSM data may produce a different scene.

## Scope and Assumptions

- Buildings are simplified extrusions on flat ground. The model does not reproduce detailed ECC architecture or terrain and has not been calibrated against field measurements.
- Before BER evaluation, the channel is normalized to unit average subcarrier power. The BER curves therefore examine frequency selectivity rather than absolute received power or site coverage.
- The link simulation assumes a static channel and perfect channel knowledge. Channel estimation and synchronization errors are not modeled.
- Channel effects are applied in the frequency domain. The experiment does not test time-domain intersymbol interference or performance with an insufficient cyclic prefix.
- Repeated bits are combined with equal weights after equalization; the receiver does not implement reliability-weighted LLR combining.

## Data Attribution

Building footprints are retrieved from [OpenStreetMap](https://www.openstreetmap.org/).

© OpenStreetMap contributors. Data is available under the [Open Database License (ODbL)](https://www.openstreetmap.org/copyright).
