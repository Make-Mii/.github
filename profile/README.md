# Make Mii!

<div align="center">
  <img src="https://raw.githubusercontent.com/Make-Mii/wii-ai/main/assets/logo.png" alt="Make Mii Logo" width="300" />

  <p><strong>Real-time facial capture to playable Nintendo Wii Mii avatars</strong></p>

  <p>
    <a href="https://hacknotts.com">Built for HackNotts '26</a>
  </p>
</div>

---

## Overview

**Make Mii!** is an end-to-end computer vision and Nintendo Wii homebrew ecosystem that transforms your real-life face into a canonical Nintendo Mii avatar in real time. Using a live webcam, the system isolates facial landmarks and hair contours, matches physical characteristics against canonical Nintendo Mii asset geometries using OpenAI's CLIP, extracts perceptual color tones in CIELAB space, and beams the generated Mii over UDP to bare-metal Wii hardware to be rendered live and permanently saved into the console's Mii Plaza database.

---

## Architecture Pipeline

<table align="center" width="100%">
  <thead>
    <tr>
      <th align="center" width="22%">1. Capture &amp; Segmentation</th>
      <th align="center" width="26%">2. AI Feature Matching</th>
      <th align="center" width="26%">3. Payload Serialization</th>
      <th align="center" width="26%">4. Wii Hardware Target</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="center" valign="top">
        <p><b>Webcam Stream</b></p>
        <p><code>VideoCapture.py</code></p>
        <p>↓</p>
        <p><b>Google MediaPipe</b></p>
        <p>Face landmarks &amp; hair masks</p>
        <p>↓</p>
        <p><b>Live Stream Controls</b></p>
        <p>Real-time sliders for Brightness, Contrast, Gamma &amp; Saturation</p>
      </td>
      <td align="center" valign="top">
        <p><b>OpenAI CLIP (ViT-B/32)</b></p>
        <p>CLAHE texture filtering &amp; cosine similarity matching against indexed Mii sprites</p>
        <p>↓</p>
        <p><b>CIELAB Delta-E</b></p>
        <p>Weighted perceptual color quantization for skin, hair, and eyes</p>
      </td>
      <td align="center" valign="top">
        <p><b>Binary Injection</b></p>
        <p><code>mii_payload.py</code></p>
        <p>Big-endian bitfield injection into <code>template.mii</code></p>
        <p>↓</p>
        <p><b>UDP Broadcast</b></p>
        <p>74-byte binary datagram sent over Wi-Fi to port <code>8080</code></p>
      </td>
      <td align="center" valign="top">
        <p><b>Make Mii! Homebrew</b></p>
        <p><code>wii-make-mii.dol</code></p>
        <p>Hardware-accelerated GX rendering on Wii</p>
        <p>↓</p>
        <p><b>Wii NAND Persistence</b></p>
        <p>Saves directly to <code>/shared2/menu/FaceLib/RFL_DB.dat</code></p>
      </td>
    </tr>
  </tbody>
</table>

---

## Organization Repositories

The project is modularized into two dedicated repositories across the organization:

| Repository | Technology Stack | Primary Responsibility |
|---|---|---|
| [**wii-ai**](https://github.com/Make-Mii/wii-ai) | Python 3.10, PyTorch, MediaPipe, OpenAI CLIP, OpenCV | The CV/AI computer vision backend. Captures the webcam stream, performs landmark/hair segmentation, matches geometry and colors, and transmits the resulting 74-byte payload. |
| [**wii-make-mii**](https://github.com/Make-Mii/wii-make-mii) | C/C++, devkitPPC, libogc, GX / GRRLIB | The native Nintendo Wii homebrew application. Listens on UDP port 8080, renders incoming Mii attributes on-screen, and writes the Mii record to the console's NAND database. |

---

## Quick Start Guide

### 1. Launch the Wii Application
1. Compile [**wii-make-mii**](https://github.com/Make-Mii/wii-make-mii) with `devkitPPC` or copy the prebuilt `wii-make-mii.dol` binary to `apps/wii-make-mii/boot.dol` on an SD card or USB drive.
2. Launch **Make Mii!** from the Homebrew Channel.
3. Note the Wii's local IP address displayed on the status bar (listening on port **8080**).

### 2. Run the AI Processing Backend
1. Clone and set up [**wii-ai**](https://github.com/Make-Mii/wii-ai):
<pre><code>git clone https://github.com/Make-Mii/wii-ai.git
cd wii-ai
conda create -n wii-ai python=3.10 -y
conda activate wii-ai
pip install -r requirements.txt</code></pre>

2. Set `WII_IP` in `src/main.py` to match the IP address shown on your Wii screen.
3. Start the capture and segmentation loop:
<pre><code>python src/main.py</code></pre>

4. Use the OpenCV trackbar sliders (`Brightness`, `Contrast`, `Gamma`, `Saturation`) on the `Stream` window to calibrate for room lighting conditions.

### 3. Save to Console
* Face the camera; the matched Mii updates dynamically on the Wii display.
* Press **[A]** on the primary Wiimote to save the Mii into the console's Mii Plaza database (`RFL_DB.dat`).
* Open the **Mii Channel** or launch **Wii Sports**—your avatar is now playable across all Wii software!

---

## Network Protocol Specification

* **Transport Layer:** UDP datagrams targeting port `8080`.
* **Packet Size:** Exactly **74 bytes**.
* **Memory Layout:** Packed big-endian Nintendo Mii binary format (`RawMiiData`), conforming to the official `RFL_DB.dat` record layout. Packets differing from 74 bytes are rejected by the console.

---

## Wii Remote Controls

| Wiimote Button | Functionality |
|---|---|
| **A** | Commit and save the active Mii to the Wii Plaza database (`RFL_DB.dat`) |
| **B** | Freeze or resume the live incoming video stream |
| **HOME** | Safely exit the homebrew application and return to the Homebrew Channel |

---

## NAND Safety Warning

Saving a Mii writes directly to `/shared2/menu/FaceLib/RFL_DB.dat` within the Nintendo Wii's internal NAND flash storage. Creating a BootMii NAND backup prior to testing is strongly recommended.
