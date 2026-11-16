# Umbrella sampling data

PLUMED `COLVAR` files from the umbrella sampling of the VE-cadherin dimer, for every
system. The collective variable `d1` is the distance (nm) between the centres of mass
of the two cadherin chains (atoms 1-1689 and 1690-3378). Each window restrains `d1`
with a harmonic potential with force constant κ = 500 kcal/mol/nm².
The files are the raw PLUMED output, and the `#! FIELDS` header line names the columns.
Times are in ps.

## Systems

| Folder              | System                                     |
|---------------------|--------------------------------------------|
| `no_plastic`        | VE-cadherin + Ca²⁺, no nanoplastic         |
| `PS10`, `PS20`, `PS40` | VE-cadherin + Ca²⁺ + polystyrene chain of 10 / 20 / 40 monomers |
| `no_plastic_no_ca`  | VE-cadherin, no nanoplastic, no Ca²⁺       |
| `2D_BUQ`            | VE-cadherin + Ca²⁺ + polystyrene chain of N monomers (`PS<N>`) |

## Layout

```
data/
├── <system>/
│   ├── <cv>/COLVAR                 grid windows 1.6, 2.1, ..., 6.6 nm
│   └── 1D_BUQ/<cv>/                windows chosen by 1D Bayesian quadrature
│       ├── COLVAR_0-50ns           original 50 ns run
│       └── COLVAR_50-75ns          25 ns extension of that run
└── 2D_BUQ/PS<N>/<cv>/COLVAR        windows chosen by 2D Bayesian quadrature
```

`<cv>` is the umbrella centre in nm.

### Grid windows: `<system>/<cv>/COLVAR`

- 11 windows per system, 1.6-6.6 nm in steps of 0.5 nm.
- Each run is 75 ns. It starts with a short steered pull from 1.6 nm to the window
  centre (column `steer.d1_cntr`), and the rest is umbrella sampling at that centre.
- **Slower pull.** In `no_plastic` 6.1 and 6.6 and in `PS20` 4.6, 5.1 and 5.6, the pull
  is twice as slow, so these runs are 90 ns.
- **Analysis window.** The last 40 ns are used: 35-75 ns of sampling, or 50-90 ns for
  the slower-pull runs.

### 1D BUQ windows: `<system>/1D_BUQ/<cv>/`

- Present for `no_plastic`, `PS10`, `PS20` and `PS40`.
- **Two files per window.** Each window was first run for 50 ns (`COLVAR_0-50ns`) and
  then extended by 25 ns (`COLVAR_50-75ns`). The extension is a separate PLUMED run, so
  its time column restarts at 0. To join the two, add the last time of
  `COLVAR_0-50ns` to the extension's times. The extension has no pull phase.
- **Analysis window.** Over the combined 75 ns, the last 40 ns (35-75 ns) are used.
- **`PS10/1D_BUQ/3.02` and `3.24`** are single continuous 75 ns runs (`COLVAR_0-75ns`).
  They were not run as 50 + 25 ns.
- **`no_plastic/1D_BUQ/3.10` is a rerun of the 3.1 nm grid window.** It used the 1D BUQ
  setup (50 + 25 ns). In the free-energy profile built from 1D BUQ + grid windows, it
  replaces `no_plastic/3.1`. In the grid-only profile, `no_plastic/3.1` is used.

### 2D BUQ windows: `2D_BUQ/PS<N>/<cv>/COLVAR`

- Windows chosen in the two-dimensional space of plastic length N and COM distance.
- Each run is 50 ns, with times running from 100 to 150 ns. It starts with a steered
  pull from 1.6 nm to the window centre at a constant 0.35 nm/ns, so the pull takes
  longer for larger centres: 2.0 ns for 2.30 nm, 11.2 ns for 5.52 nm. The rest is
  umbrella sampling at that centre.
- **Analysis window.** The last 15 ns (35-50 ns) are used.
- **Excluded runs.** Only the runs used in the free-energy analysis are included. These
  runs were excluded:

| Run       | Reason                       |
|-----------|------------------------------|
| PS1 4.89  | plastic moves away           |
| PS3 1.67  | COM distance below 2 nm      |
| PS4 3.03  | plastic position incorrect   |
| PS5 4.18  | plastic moves away           |
| PS8 2.65  | excluded                     |
| PS9 5.25  | chain unfolding              |
| PS16 4.75 | plastic moves away           |
| PS17 3.28 | negative mean force          |
| PS31 5.20 | chain unfolding              |
