# In-Sensor 3s-Chunk Architecture: A Native Visual Recording & Retrieval Paradigm for the AI Era

*[Read in Chinese (中文版)](README_zh.md)*

**Status:** Request for Comments (RFC) / Conceptual Architecture Whitepaper  
**Focus:** In-Sensor Computing, Computer Vision, Autonomous Driving Data Pipeline, Web-Native Media  

---

## Introduction: Reconstructing the Data Pipeline at the Physical Source

In current visual computing systems, we are trapped in a massive "redundancy trap": sensors blindly capture full pixels $\rightarrow$ massive redundant data floods the bus $\rightarrow$ heavy video decoders (VPU/ISP) consume massive compute to compress it $\rightarrow$ playback or AI retrieval requires heavy compute again to decompress.

**This proposal introduces a brand-new underlying data paradigm: pushing data reduction and feature extraction directly to the physical source where photons turn into electrons (the CMOS/SRAM layer).**

We abandon traditional continuous video stream encapsulation (like H.265/MP4) and adopt a discrete architecture based on **"3-Second Hardware Chunks" (1080p/4K Index Frame + Physical Micro-Delta Stream)**. This design aligns seamlessly with the AI era's demand for "semantic-rich" data, drastically reduces network redundancy in Web-native transmission, and provides robust anti-tampering capabilities because it records *true physical residuals* rather than AI-hallucinated pixels.

---

## 1. Core Architecture: Discrete "3-Second Atomic Chunks"

Completely eliminating the "heavy container, high coupling" nature of traditional video streams, the timeline is reconstructed into independent `.3s` chunk files.

Each 3-second chunk contains strictly two parts:
1. **Index Frame:** A single 1080p or 4K native RGB static frame used for visual anchoring and millisecond-level retrieval.
2. **Physical Delta Stream:** True pixel displacement and feature vectors extracted at the sensor hardware layer for the subsequent 3 seconds (extremely lightweight, a few hundred KBs), completely discarding full intermediate frames.

Under this mechanism, a 1-minute dynamic recording appears in the file system as 20 independent yet continuous atomic chunks.

---

## 2. Hardware Source Layer: The "Three-Step Elimination" in 3D Stacked CMOS

Data slimming must occur before data leaves the chip and enters the bus. We harden three pipelines directly in the logic layer (On-chip SRAM/ASIC) of the 3D stacked CMOS:

### 1. Hardware 3D LUT Color Quantization (Physical Baking)
* **Mechanism:** Stops outputting 12/14-bit RAW data laden with useless dynamic range. The moment photons convert to electrical signals, an on-chip 3D LUT directly maps and quantizes them into 8-bit sRGB or Log curves.
* **Benefit:** "Bakes away" highlight/shadow color gradations unnecessary for human eyes and Web viewing, reducing source volume by ~40%.

### 2. Dynamic Noise Gate (Threshold Truncation)
* **Mechanism:** Sets a baseline physical mask. When pixel fluctuations (such as dark-field thermal noise or subtle micro-flickers on smooth walls) fall below human perception thresholds, hardware circuitry forcibly zeroes them out.
* **Benefit:** Eliminates the core culprit preventing video bitrates from dropping—high-frequency random thermal noise.

### 3. In-SRAM Motion Engine (Block Matching)
* **Mechanism:** The sensor enters Binning mode. SRAM-connected block matching circuits perform pixel-level XOR comparisons between the current frame and the index frame. **Never outputting full frames outward**, it only outputs coordinate displacement vectors $(dx, dy)$ and sparse residuals of valid changes.

---

## 3. Encapsulation & Transmission: Zero-Redundancy Web-Native Packets

Instead of traditional MIPI line-by-line pixel transmission, this architecture uses a custom **sparse packet encapsulation**:

* **RLE (Run-Length Encoding) & Sparse Arrays:** Zeroed-out noise is packed as `[position, count of consecutive zeros, compressed value]`, consuming zero bandwidth in static or redundant regions.
* **Native Semantic Binding:** Encapsulated units are no longer bare pixels, but mathematical vectors carrying spatial motion laws.
* **Anti-Tampering Imprint:** Since it records hardware-level XOR vectors from the CMOS physical layer rather than AI-generated pixels, Deepfakes struggle immensely to bypass physical optical flow constraints when tampering with this format.

---

## 4. Software Rendering Layer: Zero-Decode GPU Pipeline

Bypassing heavy system-level VPU hardware video decoders, achieving zero-latency, stutter-free visual recreation:

* **Zero-Copy DMA to VRAM:** The index image of the `.3s` file is loaded directly into GPU VRAM via DMA as a Base Texture.
* **Mesh Warping via Shaders:** Lightweight motion vectors drive GPU Compute Shaders to perform micro-translations and optical flow interpolations on the image mesh, much like a 3D game engine.
* **Playback Experience:** Users perceive a seamless, continuous dynamic video, while the system background entirely lacks the heavy computational overhead of "video decompression."

---

## 5. Dual-Stream Pipeline (The Ultimate Form for Autonomous Driving & IoT)

This architecture naturally supports splitting dual streams ("for machines" vs. "for humans") on the same sensor bus, resolving the fundamental conflict between machine vision and visual retention:

| Dimension | Channel A: Machine Semantic Stream (For AI) | Channel B: Chunk Visual Stream (For Humans) |
| :--- | :--- | :--- |
| **Data Form** | 3D Spatial Vectors, Occupancy Grids, Semantic Labels | `1080p/4K Index Frame + Physical Delta Stream` |
| **Data Destiny** | **Read-and-Discard:** Raw pixels are instantly wiped from SRAM after on-chip NPU feature extraction | **Discrete Persistence** |
| **Pipeline Overhead** | Zero video encoding; bus transmits only tens of KB of text vectors | Zero VPU decoding; relies on GPU texture stretching |

---

## 6. Why This Architecture is Inevitable in the AI Era

1. **Solving Massive Data Retrieval Bottlenecks ($O(1)$ Complexity)**
   * Current autonomous driving or security footage requires frame-by-frame video decoding to locate specific moments.
   * Under this proposal, the system simply reads the static index frame at the header of the `.3s` chunk (only 2%–5% of total volume) without any video decoding, instantly enabling Vision-Language Model (VLM) classification and filtering.
2. **Semantic Priming, Rejecting "Dumb Data"**
   * Traditional video is meaningless pixel stacking (dumb data). Motion vectors output at the hardware layer act as exceptional AI semantic features out of the box (recording object trajectories and speeds), ready to feed directly into large models for temporal forecasting.
3. **Eliminating Cosmic Redundancy in Web Propagation**
   * The vast majority of shared web videos consist of low-information content. Stripping out useless dynamic range and redundant frames at the source reduces network bandwidth and server storage pressures by orders of magnitude.

---

### Conclusion

Stop using compute power to compensate for sensor greed.

Discard redundancy at the source, extract semantics at the physical layer, and replace continuous long streams with discrete chunks. This is not merely an iteration of image compression technology, but a fundamental protocol re-architecting of visual data as it evolves into the era of large models and hyper-connectivity.

---
> **Disclaimer:** 
> This repository currently serves as an RFC (Request for Comments) and conceptual architecture whitepaper. It defines a new data paradigm rather than providing immediate physical silicon or executable code. We welcome discussions from CMOS engineers, GPU graphics developers, and autonomous driving data scientists to explore the physical implementation of this pipeline.