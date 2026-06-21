# RFSoC-Passive-Distance-Velocity

Lightweight distance and relative radial velocity estimation with a passive RF
receiver, implemented on the AMD RFSoC 4x2.

This repository accompanies the bachelor thesis [*Lightweight Distance and
Relative Radial Velocity Estimation with a Passive RF Receiver*](thesis-url),
carried out as part of the CSE3000 Research Project at Delft University of
Technology in 2026.
A passive receiver estimates its distance and relative radial velocity from a
transmitter, using calibrated RSSI for distance and direct Doppler estimation
for velocity.

## Based on Xilinx RFSoC-MTS

This project is built on top of the [Xilinx RFSoC-MTS overlay][rfsoc-mts], a
PYNQ overlay demonstrating multi-tile synchronization (MTS) on the RFSoC. The
Vivado design and the low-level capture code are derived from RFSoC-MTS: the
build flow and HDL sources under `hardware/` are a **modified version** of their
RFSoC 4x2 design, and `software/mts.py` is a **modified version** of their
Python overlay module. The RFSoC-MTS project is licensed under the MIT License,
and its original copyright notice is retained here (see [LICENSE](LICENSE)).

The original contributions of this thesis are the passive RSSI and Doppler
estimation methods, the clock-offset bracketing scheme for velocity estimation,
the clock-drift characterization, and the experiment and analysis code — all
contained in `software/prototype.ipynb`.

## Overview

The receiver captures a continuous-wave signal from an uncooperative,
unsynchronized transmitter at 2.41 GHz, sampled at 4 GS/s.
All estimation runs in software on the processing system (PS) in
Python. Two methods are implemented and evaluated:

- **Calibrated RSSI ranging** — fits a log-distance path-loss model to estimate
  distance from received signal power.
- **Direct Doppler estimation** — uses Kay's weighted phase-difference estimator
  with a clock-offset bracketing scheme to recover radial velocity despite the
  unsynchronized transmitter and receiver clocks.

## Repository structure

```
.
├── README.md
├── LICENSE
├── hardware/
│   ├── build_mts/          # Vivado build flow (modified from RFSoC-MTS)
│   │   ├── Makefile
│   │   ├── mts.tcl
│   │   ├── mts.xdc
│   │   ├── build_bitstream.tcl
│   │   ├── check_timing.tcl
│   │   └── handoff.tcl
│   ├── ip/
│   │   └── rtl/            # custom HDL sources referenced by the build
│   │       ├── ADCRAMcapture.v
│   │       └── DACRAMstreamer.v
│   ├── prebuilt/           # ready-to-load overlay (no Vivado needed)
│   │   ├── mts.bit
│   │   ├── mts.hwh
│   │   └── ddr4.dtbo
│   └── dts/                # device-tree source and its build
│       ├── Makefile
│       └── ddr4.dts
└── software/
    ├── mts.py              # modified RFSoC-MTS overlay module
    └── prototype.ipynb     # all estimation algorithms and experiments
```

### hardware/build_mts/

The Vivado build flow, modified from the RFSoC-MTS RFSoC 4x2 build. The
`Makefile` recreates the project from `mts.tcl`, applies the constraints in
`mts.xdc`, and runs synthesis, implementation, timing checks, and the hardware
handoff to produce the `.bit` and `.hwh` products. See the
[RFSoC-MTS repository][rfsoc-mts] for the base build flow.

### hardware/ip/rtl/

The custom Verilog sources (`ADCRAMcapture.v`, `DACRAMstreamer.v`) referenced by
the build, derived from RFSoC-MTS.

### hardware/prebuilt/

The prebuilt overlay, provided so the design does not have to be rebuilt:
`mts.bit` (bitstream), `mts.hwh` (hardware description), and `ddr4.dtbo`
(compiled device-tree overlay).

### hardware/dts/

The device-tree source (`ddr4.dts`) and a `Makefile` that compiles it into the
`ddr4.dtbo` overlay shipped in `prebuilt/`.

### software/

- **`mts.py`** — a modified version of the RFSoC-MTS Python module, handling the
  overlay, MTS, and sample capture.
- **`prototype.ipynb`** — the original work of this thesis: baseband
  preprocessing, Kay's frequency estimator, RSSI computation and calibration,
  the clock-offset bracketing for velocity, and all experiments (RSSI distance
  estimation, Doppler velocity estimation, and clock-drift characterization).

## Requirements

- AMD RFSoC 4x2 development board
- A signal source (this work uses a USRP) transmitting a CW tone at 2.41 GHz
- PYNQ image v3.0.1
- Vivado 2022.2 — only required to rebuild the design from source

## Getting started

1. Set up the RFSoC 4x2 with the RFSoC-MTS overlay environment as described in
   the [RFSoC-MTS repository][rfsoc-mts].
2. Copy the overlay files from `hardware/prebuilt/` (`mts.bit`, `mts.hwh`, and
   `ddr4.dtbo`) and the software files (`software/mts.py`,
   `software/prototype.ipynb`) onto the board. Keep `mts.bit` and `mts.hwh`
   together with matching base names.
3. Start the transmitter. On the USRP host, for example:
   ```bash
   /usr/libexec/uhd/examples/tx_waveforms --freq 2.41e9 --rate 1e6 \
       --wave-type SINE --wave-freq 100e3 --gain 30 --ampl 0.5
   ```
4. Open `prototype.ipynb` in the PYNQ Jupyter environment and run the cells.
   The notebook loads the overlay, captures samples, and runs the RSSI, Doppler,
   and clock-drift experiments.

## Rebuilding from source (optional)

The prebuilt overlay in `hardware/prebuilt/` is provided so the design does not
need to be rebuilt. To regenerate it:

- **Bitstream and hardware description:** from `hardware/build_mts/`, run the
  build flow (`make all`), which recreates the Vivado project from `mts.tcl`,
  applies `mts.xdc`, and produces `mts.bit` and `mts.hwh`. The flow follows the
  [RFSoC-MTS repository][rfsoc-mts] for the RFSoC 4x2 board, with the
  modifications described in the thesis. Vivado 2022.2 is required.
- **Device-tree overlay:** from `hardware/dts/`, run `make` to compile
  `ddr4.dts` into `ddr4.dtbo`.

## License

Released under the MIT License (see [LICENSE](LICENSE)). This work is derived
from the Xilinx RFSoC-MTS project; the original Xilinx copyright notice is
retained in the license file as required.

[thesis-link]: TODO
[rfsoc-mts]: https://github.com/Xilinx/RFSoC-MTS
