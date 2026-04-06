# Tingopix

*From the Latin **tingō** (to dye, to colour) and **pix** (picture element).
Colour, at the pixel level.*

**Tingopix** is an open DIY film scanner project for small-gauge film —
8mm, Regular 8, Super 8, and 16mm. It was born out of a simple goal: to
digitize home movies and archival film at a quality level that does justice
to the original material, without the cost of commercial equipment.

The project is rooted in the [Kinograph community](https://forums.kinograph.cc),
where the build has been documented since before the first line of code was
written. It is designed to be reproducible by individuals and small institutions
pursuing the preservation of *cine-doméstico* — home movies and community film
collections that deserve better than consumer-grade telecine.

---

## Philosophy

- **Archival first.** The scanner is a pure data-acquisition bridge. Output is
  linear, unprocessed, and lossless. Debayering, grading, and delivery are
  handled downstream in the software of your choice.
- **Sensel-native.** Tingopix captures and preserves true sensor values before
  any irreversible processing is applied. What is stored is what the sensor
  actually measured — nothing more, nothing less.
- **Open and attributable.** Everything here — hardware, software, and
  documentation — is published for others to study, build, and improve upon,
  with attribution.
- **Honest about trade-offs.** Speed is sacrificed for quality. This is a slow
  scanner. That is a deliberate choice.
- **Gentle on film.** Tension-controlled transport designed to be safe for
  fragile, aging reels that may never run through a projector again.

---

## Archival Context

The following was submitted as a conference session proposal:

> **Domestic Sensel-Native Archiving: A Recipe for Uncooked Scanning**
>
> In the world of film digitization, "raw" image data is inherently "cooked"
> the moment it is captured. This concept, championed by Reto Kromer at the
> No Time To Wait series, provides the context for this session: a practical
> how-to of "uncooked" sensel-native archiving via a DIY 8mm cinefilm scanner.
> By treating the scanner as a pure data-acquisition bridge, we argue that
> sensel-data provides the most honest digital representation of the film's
> content, within the inherent constraints of a given sensor's color filter
> array spectral sensitivity — and demonstrably within the budget of an
> individual builder. This method creates a fixed, immutable datapoint that
> is free of typical unknown "hallucinations" introduced by traditional
> debayering at the point of capture. Our open-source implementation sidesteps
> other hardware constraints by utilizing the full quantization range of the
> sensor's native Bayer mosaic. We present a choice of scanning strategies:
> producing an "Honest" 1/4-resolution Binned TIFF for a ready-to-use result
> based strictly on measured values, or a Sensel-TIFF to preserve the option
> for post-scan debayering choice. This latter approach allows future image
> representations to evolve with algorithm improvements without ever altering
> the original, authoritative sensel-record.

The theoretical foundation for this pipeline is Reto Kromer's lightning talk
*The Photon Path: from Sensel Values to Pixel Values*, presented at
*No Time to Wait 6* (Den Haag, October 2022)
([slides](https://reto.ch/training/2022/2022-10-28/slides.pdf),
[video](https://www.youtube.com/watch?v=G4EGQeHg22g)).
The *sensel* concept was coined by Charles Poynton for the IMAGO Photon Path
Diagram. See [reto.ch](https://reto.ch) for his broader body of work on
raw data preservation.

---

## Projects

### v1 — RPi 4 + IMX477 HQ Camera

> *The first Tingopix scanner. Operational as of 2026.*

Built around a Raspberry Pi 4 and the Sony IMX477 HQ camera module, v1
achieves archival-quality results comparable to commercial scanners costing
many times more — at a build cost in line with other community projects.

#### What makes it different

Most DIY film scanners output a processed RGB image or a standard video file.
Tingopix v1 goes further:

- **Direct Sensel-RAW capture** — 12-bit Bayer data from the IMX477, bypassing
  picamera2's RGB pipeline entirely. No irreversible processing on the scanner.
- **Calibrated color science** — Black Level Correction (BLC), Clipping, and a
  Color Correction Matrix (CCM) using Rolf Henkel's scientific tuning file for
  the RPi HQ sensor, incorporated into `picamera2`.
- **Embedded sprocket metadata** — sprocket location data is captured at a
  separate light level and embedded directly into the rightmost pixels of the
  RAW array, carrying registration data without a separate file.
- **Dual output paths** — an Honest Binned TIFF (1/4 resolution, strictly
  measured values, suitable for HD delivery and upconversion) for immediate
  usable results; and a full Sensel-TIFF for post-scan debayering to TIFF16
  or lossless OpenEXR — the maximum resolution path to professional
  post-production and color correction.
- **Multi-exposure / per-channel capture** — separate captures per RGB channel
  for maximum dynamic range, with or without light intensity control.
- **Vision-based frame registration** — sprocket hole detection via blue-channel
  projection profile on a dedicated ROI, enabling electronic stabilization
  with no physical film gate.
- **Hardware → software architecture** — mechanical complexity is deliberately
  minimized; computation and vision do the work that hardware sensors and
  encoders would otherwise require.

#### Hardware

| Component               | Details                                              |
|-------------------------|------------------------------------------------------|
| Compute                 | Raspberry Pi 4                                       |
| Camera                  | RPi HQ Camera Module (Sony IMX477, 12MP)             |
| Lens                    | Enlarger lens with M42 tube and adapters             |
| Transport & tension     | 3D-printed roller tension system (PICO controlled)   |
| Illumination & control  | 3D-printed integrating sphere; bespoke LED control via PICO |
| Motion controller       | Raspberry Pi Pico (RP2040) — F-Code command system   |
| Film formats            | 8mm, Regular 8, Super 8, 16mm (small reels)          |
| Reel capacity           | Up to 400ft for 8mm                                  |
| Output storage          | USB3 SSD or NAS                                      |

#### Software Stack

| Layer                    | Technology                                          |
|--------------------------|-----------------------------------------------------|
| Language                 | Python                                              |
| Camera control           | picamera2 (RAW-only, unconventional mode)           |
| GUI                      | GTK4, Cairo (waveform display)                      |
| Multiprocessing          | Python multiprocessing, shared memory, CPU pinning  |
| Debayering — preview     | Numba JIT (real-time GUI)                           |
| Debayering — immediate   | Honest Binned (1/4 resolution, measured values)     |
| Debayering — archival    | VNG post-scan (full resolution)                     |
| File output              | tifffile, OpenEXR (12–16 bit, linear)               |
| OS                       | Raspberry Pi OS (Linux)                             |

#### File Output Summary

| Output type                    | Format            | Use case                               |
|--------------------------------|-------------------|----------------------------------------|
| Sensel-TIFF with sprocket data | TIFF16            | Archival master, post-scan debayering  |
| Honest Binned TIFF             | TIFF16            | Immediate use, HD delivery, upconversion |
| Post-scan VNG debayered        | TIFF16 or OpenEXR | DaVinci Resolve, color grading         |

All files are saved as **Linear**. Gamma is selected and applied in the
color grading software. Bit depth: 12–16 bit.

#### Scan Performance

| Capture mode                          | Approx. frames/min |
|---------------------------------------|--------------------|
| Single capture, all channels          | ~25                |
| Per-channel, separate captures        | ~16                |
| Multi-exposure (current, unoptimized) | ~6                 |

#### Build thread

Full technical documentation of the v1 build, including design decisions,
bugs, and workarounds, is in the Kinograph forum thread:
[SnailScanToo or 2](https://forums.kinograph.cc/t/snailscantoo-or-2/2548)

---

## Status

v1 is operational. The codebase is being cleaned up and documented for
public release on this repository. If you are interested in building one,
opening an issue or a discussion here is the best way to accelerate that
process — knowing there are builders waiting is the strongest incentive to
prioritize documentation over development.

---

## Community

- **Kinograph Forums** — [forums.kinograph.cc](https://forums.kinograph.cc) —
  where this project grew up
- **Blog** — [tingopix.blogspot.com](https://tingopix.blogspot.com)
- **Reddit** — [r/tingopix](https://www.reddit.com/r/tingopix)
- **Website** — [tingopix.github.io](https://tingopix.github.io)

---

## Acknowledgements

Special thanks to:

- **Reto Kromer ([AV Preservation by reto.ch](https://reto.ch))** — for his
  foundational work on sensel-native archival principles, which provide the
  theoretical context for Tingopix's pipeline. See the Archival Context
  section above for the specific citation.
- **Rolf Henkel ([cpixip](https://forums.kinograph.cc/u/cpixip))** — for his
  work with the RPi HQ sensor, for sharing his insights into picamera2, and for
  his scientific tuning file, which is used in the Tingopix color processing
  pipeline.
- **Matthew Epler ([matthewepler](https://forums.kinograph.cc/u/matthewepler))** —
  for creating and maintaining the Kinograph forum, which makes projects like
  this possible.
- The Kinograph community — for years of shared knowledge and open collaboration.

---

## License

| What                    | License                                                  |
|-------------------------|----------------------------------------------------------|
| Software / code         | MIT License                                              |
| Hardware designs        | CERN Open Hardware Licence v2 — Permissive (CERN-OHL-P)  |
| Documentation & content | Creative Commons Attribution 4.0 (CC BY 4.0)             |

Attribution: please credit **Tingopix** and link to this repository.

---

*A recipe for uncooked scanning.*
