# VolterraCalculator

A desktop tool for working with nonlinear systems described as **Volterra series** (kernels h0 to h5).
You can create, load and save Volterra systems, combine them (add, subtract, scale, chain, invert), and
inspect their kernels in plots. The heavy kernel math runs on the GPU through OpenCL.

## Architecture

```
src/volterra_calc/
├── gui/             PyQt5 front end (GUI_pyInput.py) — the entry point
├── core/            Volterra math
│   ├── Volterra_In_Out_7.py            VolterraResponse: kernels + input signal → output signal
│   ├── Volterra_Operations_noncausal.py system algebra: add, subtract, multiply, chain, invert
│   ├── Volterra_Downsample.py          resampling and filters
│   └── Volterra_In_Out_6.py            previous VolterraResponse, still used by operations/plots
├── gpu/             opencl_util.py — creates the OpenCL context and queue
├── storage/         Volterra_File_Operations.py — reads/writes kernel sets as CSV
└── visualization/   Volterra_PlotSystem.py — kernel overview and per-order plots

data/
├── Volterra_Kernels/     stored systems: h0<name>.csv … h5<name>.csv
└── Volterra_Input_Data/  input signals (e.g. SPICE.csv)

tests/, benchmarks/       checks and GPU/CPU performance scripts
```

The GUI calls `storage` to load and save systems, `core` to compute on them, and `visualization` to
plot them. Long operations run in a background thread so the window stays responsive. `core` uses
`gpu` for the OpenCL kernels.

A **system** is a set of six kernel files sharing one name. For example, `DUT` is
`h0DUT.csv` to `h5DUT.csv` in `data/Volterra_Kernels/`.

## Getting started

Requirements: Python 3.11+ and an OpenCL driver for your GPU (or a CPU OpenCL runtime).

```bash
python -m venv .venv
.venv\Scripts\activate          # Windows  (Linux/macOS: source .venv/bin/activate)
pip install numpy scipy pandas matplotlib scikit-learn pyopencl PyQt5 qdarkstyle
python src/volterra_calc/gui/GUI_pyInput.py
```

In the window, create a system with **Create System**, or use **Load System** and enter a system name.
**Load System** only looks directly inside `data/Volterra_Kernels/`. To use a bundled set, copy its
files up one level first (e.g. `lmh3401_corr/h*DUT.csv`, then load `DUT`). Then use the operation
buttons (Add, Subtract, Concatenate, Invert, Multiply) or the plot buttons.

To check which OpenCL devices are available:

```bash
python src/volterra_calc/gpu/opencl_util.py
```
