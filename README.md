# Voyager Signal Recovery

## Overview

This project reconstructs and decodes a deep‑space telemetry signal captured from the Voyager spacecraft.

Starting from raw wideband IQ recordings, the pipeline progressively processes the signal through several digital signal processing (DSP) stages to recover the original telemetry frames and payload.

The workflow includes the following stages:

1. Wideband signal detection  
2. Carrier drift analysis  
3. Baseband conversion  
4. Residual carrier correction  
5. Matched filtering  
6. Symbol rate detection  
7. Symbol timing recovery  
8. Carrier phase recovery  
9. Bit slicing  
10. Frame synchronization  
11. Bitstream transformation removal  
12. Telemetry frame extraction  
13. Payload recovery  

The final result is the successful extraction of the spacecraft’s telemetry payload from the raw RF recording.

---

# Dataset

The input consists of a series of recorded IQ captures:
````dsn_capture_*.npy````
Each file contains:
2 channels (I/Q)
2,000,000 samples

All captures are concatenated to form one continuous signal stream:
voyager_signal.npy

**Sampling rate:**
fs = 2,000,000 samples/sec


---

# Processing Pipeline

## 1. Concatenation of Captures

All recorded files are merged into a continuous IQ stream.

Output:
````voyager_signal.npy````


This step creates a single continuous signal used for all subsequent processing stages.

---

## 2. Wideband FFT Scan

A wideband FFT scan is performed to detect the spacecraft carrier.

Outputs produced during this stage:

- Spectrogram
- Carrier frequency vs time
- Waterfall plot

Purpose:

- Identify the signal location in the spectrum
- Observe carrier drift across the recording
- Estimate initial carrier frequency

These visualizations help confirm the presence of the Voyager signal and determine the center frequency used for baseband conversion.

---

## 3. Carrier Downconversion

The detected carrier is mixed to baseband using complex mixing.

Mathematically:
````baseband = iq * exp(-j2πf_ct)````


Output:
````voyager_baseband.npy````

This step shifts the signal spectrum so that the carrier is centered at zero frequency.

---

## 4. Residual Carrier Correction

After downconversion, a small remaining frequency offset remains in the signal.

Example residual estimate:

`Residual carrier ≈ -2807 Hz`

This offset is removed using another complex mixing step.

Output:
`voyager_baseband_corrected.npy`


This correction stabilizes the constellation and prepares the signal for demodulation.

---

## 5. Symbol Rate Detection

The modulation rate is estimated using spectral analysis of the signal.

Estimated values:
`Symbol rate ≈ 18.5 ksymbols/sec`
`Bandwidth ≈ 25 kHz`


The symbol rate determines how the signal should be sampled during demodulation.

---

## 6. Root Raised Cosine Filtering

A matched Root Raised Cosine (RRC) filter is applied.

Purpose:

- Remove excess bandwidth
- Improve signal‑to‑noise ratio
- Prepare the signal for symbol timing recovery
- Improve constellation clarity

Matched filtering is an essential step in digital communications receivers.

---

## 7. Symbol Timing Recovery

Symbol timing phase is determined by testing multiple sampling offsets and selecting the one that produces the clearest constellation.

Procedure:

- Try several symbol phase offsets
- Evaluate constellation separation
- Select the best sampling phase

Output:
`Symbol-sampled IQ stream`


This produces samples aligned with symbol boundaries.

---

## 8. Carrier Phase Recovery

Carrier phase ambiguity is corrected using block phase estimation.

Purpose:

- Remove phase rotation
- Align constellation axes
- Stabilize symbol clusters

Resulting constellation:
`Two distinct symbol clusters`

This confirms the expected BPSK modulation.

---

## 9. Bit Slicing

The corrected symbols are converted into bits using a decision boundary.

Output:
`voyager_bits.npy`


Each symbol is mapped to either:
0 or 1


based on the sign of the real component.

---

## 10. Frame Synchronization

The recovered bitstream is searched for the standard CCSDS frame synchronization marker:
`1ACFFC1D`


Initial searches do not detect this marker.

This indicates that the bitstream has undergone an additional transformation.

---

## 11. Bitstream Transformation Discovery

Further analysis reveals that the transmitted bitstream is XOR‑masked with a repeating 32‑bit pattern.

Discovered mask:
`Mask: 0x19000002`


Removing this mask restores the expected synchronization marker.

---

## 12. Frame Extraction

After removing the mask:
`decoded_bits = bits XOR mask`


The CCSDS sync marker becomes visible.

Frames can then be extracted using the structure:
`[ Sync Marker | Header | Telemetry Data | Error Control ]`

Frame boundaries are identified using the synchronization marker.

---

## 13. Payload Recovery

Once frames are detected and aligned, the telemetry payload can be reconstructed.

Recovered data includes:

- spacecraft telemetry packets
- encoded payload information
- frame headers and metadata

---


---

# Visualizations Produced

The analysis produces several diagnostic plots:

- Carrier drift plot
- Waterfall / spectrogram
- Constellation diagrams
- Symbol timing comparison
- Sync correlation plots

These visualizations were critical for debugging and confirming each processing stage.

---

# Key Challenges

## Large Dataset Handling

The signal recordings exceed memory limits.

Solution:

- Memory‑mapped arrays
- Chunk‑wise processing

This allowed the dataset to be processed without loading the entire signal into memory.

---

## Carrier Drift

Long recordings introduce noticeable frequency drift.

Solution:

- Wideband FFT scanning
- Residual carrier estimation

These techniques allowed the carrier to be accurately tracked and corrected.

---

## Unknown Bitstream Transformation

The spacecraft applies an undocumented bit‑level transformation.

Solution:

- Correlation analysis against the CCSDS sync word
- Discovery of XOR mask

Removing the mask restored proper frame synchronization.

---

# Final Result

The complete DSP pipeline successfully:

- Detected the Voyager carrier
- Recovered baseband symbols
- Corrected phase and timing
- Removed the hidden bitstream transformation
- Recovered CCSDS frames
- Extracted the spacecraft payload

---

# Requirements

The project requires the following Python packages:
`numpy
scipy
matplotlib`

## Video Presentation link:
https://drive.google.com/drive/folders/1fuRG0b_79ZQGQDeDOBeFIQQTNZ44rvLI?usp=drive_link


