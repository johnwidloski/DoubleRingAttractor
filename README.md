# Double Ring Attractor GUI

*A browser-based simulation tool for exploring how two coupled ring attractors integrate velocity (path integration)*

---

## Run it

Open the following URL in any modern browser:

https://johnwidloski.github.io/DoubleRingAttractor/DoubleRingAttractor.html

Press **Start** to begin the simulation. No installation or server required.

---

## Overview

This tool implements a **double** ring attractor: two ring networks (**A** and **B**), each containing excitatory (E) and inhibitory (I) populations arranged on a circular topology. The two rings are recurrently coupled and jointly track a moving external "place" input. A velocity signal drives the two rings with equal-and-opposite gain, so that the shared activity bump moves at a rate proportional to velocity — the network **integrates velocity into position** (path integration), the mechanism thought to underlie head-direction and grid-cell tuning.

This is the two-ring companion to the single-ring [Ring Attractor GUI](https://github.com/johnwidloski/RingAttractorBuild). As there, connectivity is computed efficiently via FFT-based circular convolution, the simulation runs continuously in a background worker, and all parameters can be adjusted in real time.

---

## Network architecture

Four populations in total — E and I on each of the two rings (`r_EA`, `r_IA`, `r_EB`, `r_IB`) — connected by two families of Gaussian weight kernels:

- **Within-ring weights** (applied identically on each ring): `w_EE` (E→E), `w_IE` (E→I), `w_EI` (I→E), `w_II` (I→I).
- **Cross-ring weights**: the same four kernel types coupling ring A ↔ ring B (A→B and B→A), which lock the two bumps together into a single coherent representation.

Each kernel is a Gaussian profile parameterized by **Amplitude** and **Width**, and can be **Invert**ed (peak → trough) or **Binarize**d (Heaviside threshold). All synaptic interactions are evaluated as circular convolutions in the Fourier domain.

---

## Network dynamics

Total synaptic input to each population combines the recurrent (attractor) drive, the cross-ring drive, a uniform bias, and the external place input. For the excitatory population of ring A:

```
G_EA = m_int · (g_EE_A + g_EI_A + cx_EE_A + cx_EI_A + β_E − adaptation)
     + m_place · (placeInput · gainA)
```

with analogous expressions for `G_IA`, `G_EB`, `G_IB` (the inhibitory populations receive no place input). Firing rates are obtained through a **rectified-linear** transfer function, and the populations follow a **leaky integrator** with synaptic time constant `τ_s`:

```
r += s − r · dt / τ_s
```

In **rate mode**, `s = f · dt` (deterministic). In **spiking mode**, `s` is drawn from an inhomogeneous **Poisson** process, `s ~ Poisson(f · dt)`, producing stochastic dynamics.

---

## Velocity integration (path integration)

A moving **place input** — a Gaussian blob whose position advances as `pos ← pos + velocity · dt` around the ring — provides the external teaching signal. The **velocity gain** `β_vel` scales this place drive *asymmetrically* between the two rings:

```
gainA = max(0, 1 + β_vel · velocity)
gainB = max(0, 1 − β_vel · velocity)
```

When the animal (place input) moves in the positive direction, ring A is driven more strongly and ring B more weakly (and vice-versa). Because the rings are cross-coupled, this asymmetry pushes the shared bump along at a rate set by `β_vel`, so bump position becomes the running **integral of velocity**. With `β_vel = 0` the two rings are driven equally and the bump simply follows the place input without integrating.

The velocity gain applies **only** to the place input — the recurrent and cross-ring connectivity are unchanged — so `β_vel` cleanly controls the integration gain independent of the attractor's stability.

---

## Parameters

| Parameter | Description |
|---|---|
| Start | Begin / pause the simulation |
| Spiking | Toggles between deterministic rate dynamics and Poisson spiking |
| Neurons | Number of neurons per population (per ring, per E/I) |
| Dim (1D / 2D) | Switches between a 1D ring and a 2D toroidal sheet |
| Speed | Playback speed — simulation time advanced per rendered frame (does not change `dt`) |
| Within-ring Weights | The four within-ring kernels (E→E, E→I, I→E, I→I), each with Amplitude / Width / Invert / Binarize |
| Cross-ring Weights | The four A↔B coupling kernels, same controls |
| Velocity (ring/s) | Speed of the moving place input, in rings per second |
| Gain (β_vel) | Velocity gain: scales the place drive to ring A by (1 + β_vel·v) and ring B by (1 − β_vel·v) |
| Uniform Input (β_E, β_I) | Spatially uniform drive to the E and I populations |
| Time constant (s) | Synaptic/integration time constant `τ_s` |
| Tracking | Plot of the decoded bump center (circular center-of-mass) against the place-input position |
| Input Traces | Show the input/drive traces |
| Window (s) | Time window shown in the tracking / trace plots |
| Min mean rate | Minimum population rate for the bump center-of-mass to be considered valid |
| Min concentration | Minimum bump concentration (sharpness) for a valid center-of-mass estimate |
| Interval (s) | Sampling interval for the tracking / center-of-mass plot |
| Presets / Save | Load and publish shared parameter presets |

The panels labeled **A: E**, **A: I**, **B: E**, **B: I** show the live firing-rate profiles of the four populations.

---

## Presets

Presets capture the full parameter set and can be named and saved. They are **shared**: the page loads a curated `presets.json` from this repository on startup, so everyone who opens the page sees the same set of presets.

Saving is **read-only for visitors** — anyone can load the shared presets, but only the repository owner can publish new ones. With a GitHub fine-grained personal access token configured (Contents: Read and write, scoped to this repo), clicking **Save** commits the updated `presets.json` to the repo via the GitHub API, and it goes live for everyone within about a minute. The token is stored only in your browser's `localStorage` — never committed and never present in the page source.
