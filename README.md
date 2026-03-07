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

n1 = p / a^2

where
( a = 3.86 ) Å — CuO(_2) lattice constant.

Fermi-liquid limit density:

n2 = (p + 1) / a^2

Fraction of autolocalized carriers:

K1 = nbip / n1

Effective Hall carrier density:

n_H = K1 * n1 + (1 - K1) * n2 / K2

where ( K_2 ) is a phenomenological factor describing Fermi surface reconstruction and anomalous scattering.

Since in a 2D Fermi liquid:

Δμ ∝ dn

the chemical potential is corrected via:

Delta_mu ∝ (n_H - n1)

This ensures that thermoelectric quantities are calculated using the **effective Fermi-surface carrier density** rather than nominal doping.

---

# 2. Transport Model for Delocalized Carriers

Two alternative descriptions are implemented.

---

## (A) Metallic Regime — Mott Formula

For degenerate carriers:

S = (pi^2 / 3) * (kB^2 * T / mu)
This corresponds to the Mott formula for a 2D electron gas.

---

## (B) Semiconductor Regime — Fermi–Dirac Integrals

Numerical Fermi–Dirac integral:
F_j(eta) = ∫0^∞ x^j / (exp(x - eta) + 1) dx

where

η = μ/kT

The implementation uses an asymptotic exponential tail treatment to avoid overflow during numerical integration.

Semi-classical Seebeck coefficient:

S = kB * ((r + 2) * F_{r+1}(eta) / ((r + 1) * F_r(eta)) - eta)

Scattering parameter:

* ( r = 0 ) — acoustic phonon scattering
* ( r = 1 ) — ionized impurity scattering

---

# 3. Bipolaron Contribution

Bipolaron thermopower is estimated within an electron-gas approximation:

S_bip ≈ kB / (2 * e)

The total Seebeck coefficient is calculated as:


S = w_deloc * S_deloc + w_bip * S_bip


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
* `nums[12]` → condensate bipolaron indicator

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

# 7. Results and Model Comparison

The calculated Seebeck coefficient curves were compared with experimental trends reported in the reference article.

Overall, the model reproduces the qualitative behavior of the Seebeck coefficient for several doping levels. In particular, at **low doping (p ≈ 0.1)** the calculated curves are close to those reported in the literature.

However, deviations appear near the **optimal doping level (p ≈ 0.15)**.
In this regime the calculated Seebeck coefficient increases with temperature for hole-like transport, while experimentally it should decrease. This indicates that the current model does not fully capture the temperature dependence of the system.

A likely reason is the treatment of the **chemical potential**.
In the present implementation of the metallic model the chemical potential is assumed to be constant, while physically it should decrease with increasing temperature. This effect is not yet included and likely contributes to the discrepancy.

Two transport descriptions for delocalized carriers were tested:

1. **Metallic model (Mott formula)**
   Provides qualitatively correct behavior but fails to reproduce the temperature dependence near optimal doping due to the constant chemical potential approximation.

2. **Semiconductor model using Fermi–Dirac integrals**

   Two scattering mechanisms were considered:

   * **r = 0 — acoustic phonon scattering**
     This case produced results closest to theoretical expectations.

   * **r = 1 — ionized impurity scattering**
     This scenario was tested for comparison but showed larger deviations from the expected behavior.

Additionally, the **Hall carrier density correction** was implemented in order to account for Fermi surface reconstruction effects.

When the Hall correction is included, the resulting curves become qualitatively closer to the reference article. However, the Seebeck coefficient decreases too rapidly with temperature, indicating that further refinement of the model is required.

Overall, the results suggest that the two-liquid framework captures the main physical trends but requires a more realistic temperature dependence of the chemical potential and possibly improved weighting of the carrier subsystems.


---

# 8. Purpose

This code is intended for research and model exploration of thermoelectric properties in strongly correlated superconducting systems within a two-liquid framework.

# 9. Reference

The comparison in this project is based on the results reported in:

Alexandrov, A. S., Kabanov, V. V., & Mott, N. F.
**Hall Effect and Resistivity in High-Tc Superconductors**
Physical Review B 48, 557 (1993)

DOI: https://doi.org/10.1103/PhysRevB.48.557

