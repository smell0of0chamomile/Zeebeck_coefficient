# Two-Liquid Model for Seebeck Coefficient in Cuprate Superconductors

## Overview

This project implements a **two-liquid model** for calculating the Seebeck coefficient in cuprate superconductors.

The charge system is described as a mixture of:

1. **Autolocalized carriers** (large bipolarons)
2. **Delocalized Fermi-like carriers**

The Seebeck coefficient is calculated by combining contributions from both subsystems, with optional Hall correction and alternative transport descriptions (metallic or semiconductor-like).

---

# 1. Physical Model

In cuprates, the nominal hole concentration ( p ) (per Cu site) does not directly determine the Fermi surface volume because part of the carriers may form bipolarons.

Therefore, transport quantities must be computed using an **effective carrier density**, accounting for redistribution between the two components.

---

## Effective Carrier Density

Bare 2D carrier density:

[
n_1 = \frac{p}{a^2}
]

where
( a = 3.86 ) Å — CuO(_2) lattice constant.

Fermi-liquid limit density:

[
n_2 = \frac{p + 1}{a^2}
]

Fraction of autolocalized carriers:

[
K_1 = \frac{n_{bip}}{n_1}
]

Effective Hall carrier density:

[
n_H = K_1 n_1 + (1 - K_1)\frac{n_2}{K_2}
]

where ( K_2 ) is a phenomenological factor describing Fermi surface reconstruction and anomalous scattering.

Since in a 2D Fermi liquid:

[
\mu \propto n
]

the chemical potential is corrected via:

[
\Delta \mu \propto (n_H - n_1)
]

This ensures that thermoelectric quantities are calculated using the **effective Fermi-surface carrier density** rather than nominal doping.

---

# 2. Transport Model for Delocalized Carriers

Two alternative descriptions are implemented.

---

## (A) Metallic Regime — Mott Formula

For degenerate carriers:

[
S = \frac{\pi^2}{3}\frac{k_B^2 T}{\mu}
]

This corresponds to the Mott formula for a 2D electron gas.

---

## (B) Semiconductor Regime — Fermi–Dirac Integrals

Numerical Fermi–Dirac integral:

[
F_j(\eta) =
\int_0^{\infty}
\frac{x^j}{e^{x-\eta}+1} dx
]

where

[
\eta = \frac{\mu}{k_B T}
]

The implementation uses an asymptotic exponential tail treatment to avoid overflow during numerical integration.

Semi-classical Seebeck coefficient:

[
S = k_B
\left(
\frac{(r+2)F_{r+1}(\eta)}{(r+1)F_r(\eta)}

* \eta
  \right)
  ]

Scattering parameter:

* ( r = 0 ) — acoustic phonon scattering
* ( r = 1 ) — ionized impurity scattering

---

# 3. Bipolaron Contribution

Bipolaron thermopower is estimated within an electron-gas approximation:

[
S_{bip} \approx \frac{k_B}{2e}
]

The total Seebeck coefficient is calculated as:

[
S = w_{deloc} S_{deloc} + w_{bip} S_{bip}
]

In the superconducting (condensed) state, the Seebeck coefficient is set to zero.

---

# 4. Input File Structure

The input file consists of **blocks**, each corresponding to a fixed doping level ( p ).

Each block contains:

1. First line — integer `n`: number of temperature records
2. Followed by `n` physical records
3. Each record contains **13 numerical values**
4. A single record may span multiple lines

Within each record:

* `nums[0]` → doping ( p )
* `nums[1]` → temperature ( T )
* `nums[9]` → chemical potential
* `nums[12]` → condensate indicator

---

# 5. Temperature Grid Normalization

The reference doping block (typically ( p = 0.05 )) contains 14 temperature points.

Other doping levels may contain fewer temperatures.

To ensure consistent comparison between Seebeck curves:

* Each block is extended to 14 temperature points
* High-temperature region is extrapolated
* Bipolaron density is set to zero in the extended region
  (high-temperature dissociation limit)

This guarantees identical temperature range across all doping levels.

---

# 6. Output

The program:

1. Reads all doping blocks
2. Computes Seebeck coefficient for each (p, T)
3. Applies Hall correction and transport model
4. Plots ( S(T) ) for different doping levels

---

# Purpose

This code is intended for research and model exploration of thermoelectric properties in strongly correlated superconducting systems within a two-liquid framework.
