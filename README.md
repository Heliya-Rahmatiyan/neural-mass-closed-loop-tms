# Neural Mass Modeling of Single-Pulse & Closed-Loop Phase-Locked TMS Across Epileptic Cortical Regimes

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

> **Author:** Heliya Rahmatiyan  
> **GitHub Repository:** [https://github.com/Heliya-Rahmatiyan](https://github.com/Heliya-Rahmatiyan)  
> **Keywords:** Computational Neuroscience, Wendling Neural Mass Model, Transcranial Magnetic Stimulation (TMS), Critical Slowing Down (CSD), Phase Response Curve (PRC), Closed-Loop Seizure Suppression.

---

## Project Overview

This repository presents an end-to-end computational neuroscience framework designed to model, quantify, and suppress pathological brain dynamics using Transcranial Magnetic Stimulation (TMS). Utilizing the 10-dimensional **Wendling Neural Mass Model (NMM)**, we simulate three distinct cortical regimes: **Healthy Baseline**, **Pre-Seizure Fast Gamma**, and **Epileptic Seizure Spikes**.

The research advances through five sequential notebooks—moving from baseline validation and single-pulse perturbation to Hilbert-based quantification of Critical Slowing Down (CSD), Phase Response Curve (PRC) mapping, and finally a **Causal Real-Time Closed-Loop Phase-Locked TMS System**. Our findings prove that targeting the rising slope (**270°**) of epileptic spikes with closed-loop burst stimulation collapses the limit cycle attractor, achieving an **80.2% steady-state amplitude suppression**.

---

## Table of Contents

- [Key Features](#-key-features)
- [Repository Workflow & Detailed Notebook Analysis](#-repository-workflow--detailed-notebook-analysis)
  - [01. 01_jansen_rit_baseline_tms](#01_jansen_rit_baseline_tms)
  - [02. 02_Wendling_NMM_Fast_Inhibition_TMS](#02_wendling_nmm_fast_inhibition_tms)
  - [03. 03_Quantification of Perturbation Recovery](#03_quantification-of-perturbation-recovery)
  - [04. 04_Phase_Dependency_PRC](#04_phase_dependency_prc)
  - [05. 05_Closed_Loop_Seizure_Suppression](#05_closed_loop_seizure_suppression)
- [Comprehensive Experimental Benchmarks](#-comprehensive-experimental-benchmarks)
- [Installation & Requirements](#-installation--requirements)
- [Usage Guide](#-usage-guide)
- [License](#-license)

---

## Key Features

- **10D Wendling Neural Mass Model:** Realistic simulation of main pyramidal neurons, local excitatory interneurons, slow $GABA_A$ inhibitory interneurons, and fast $GABA_B/GABA_{A,\text{fast}}$ interneurons.
- **Differential Response Isolation ($\Delta\text{EEG}$):** Elimination of baseline stochastic background activity by subtracting unperturbed control runs from perturbed runs (identical random seeds).
- **Critical Slowing Down (CSD) Quantification:** Analytic signal envelope extraction via Hilbert Transform to measure Recovery Time ($\tau_{\text{rec}}$) near bifurcation thresholds.
- **Weak Perturbation PRC Extraction:** Regime-tailored stimulation intensity ($\epsilon \ll \theta_{\text{regime}}$) evaluated across 24 phase target bins ($0^\circ\text{--}360^\circ$).
- **Causal Real-Time Phase Tracking:** Sliding-window bandpass filtering with phase-lag delay compensation for online EEG processing.
- **Closed-Loop Desynchronization Engine:** Comparative evaluation of Baseline, Open-Loop ($8\text{ Hz}$), Closed-Loop Single-Pulse, and Closed-Loop Burst TMS paradigms.

---

## Repository Workflow & Detailed Notebook Analysis

---

### `01_jansen_rit_baseline_tms`
* **Objective & Overview:**  
  Establishes the foundational Jansen-Rit / Wendling Neural Mass architecture. Focuses on parameter tuning, numerical integration via 4th-order Runge-Kutta (RK4), and initial verification of normal cortical baseline activity.
* **Mathematical Foundation:**  
  Models interaction between Pyramidal Cells ($y_1$) and Excitatory/Inhibitory Interneurons using second-order differential equations transforming average presynaptic firing rate $S(v)$ into postsynaptic membrane potentials $v(t)$:
  $$S(v) = \frac{2e_0}{1 + e^{r(v_0 - v)}}$$
* **⚠️ Key Engineering Challenges & Solutions:**
  * **Challenge:** Numerical instability during long-duration stochastic integration and initial transient artifacts.
  * **Solution:** Applied 4th-order Runge-Kutta (RK4) integration with a fine time step ($\Delta t = 0.001\text{ s}$) and discarded the initial $1.0\text{ s}$ transient period to ensure steady-state equilibrium before recording.

---

### `02_Wendling_NMM_Fast_Inhibition_TMS`
* **Objective & Overview:**  
  Extends the baseline model to the 10D Wendling NMM by incorporating fast inhibitory interneurons ($G$). Configures three distinct pathophysiological cortical regimes and applies single-pulse TMS perturbations into the pyramidal population:
  1. **Healthy Baseline:** Balanced excitation and slow/fast inhibition ($A=3.25, G=10$).
  2. **Pre-Seizure Fast Gamma:** Elevated fast inhibitory synaptic gain ($G=165$), producing $40\text{ Hz}$ narrowband oscillations.
  3. **Epileptic Seizure Spikes:** High-amplitude, hypersynchronized epileptic spike train ($A=5.0$).
* **⚠️ Key Engineering Challenges & Solutions:**
  * **Challenge:** Differentiating true external TMS perturbation effects from ongoing background neuronal firing.
  * **Solution:** Introduced identical random seed pairing between control ($\text{EEG}_{\text{ctrl}}$) and perturbed ($\text{EEG}_{\text{pulse}}$) simulations, allowing precise subtraction to extract pure response dynamics.

---

### `03_Quantification of Perturbation Recovery`
* **Objective & Overview:**  
  Quantifies cortical resilience and Critical Slowing Down (CSD) by measuring the Recovery Time ($\tau_{\text{rec}}$) required for the perturbed signal envelope to return to baseline.
* **Mathematical Formulation:**  
  Pure differential EEG is transformed into an analytic signal $z(t)$ via Hilbert Transform:
  $$z(t) = \Delta\text{EEG}(t) + i \cdot \mathcal{H}\{\Delta\text{EEG}(t)\}, \quad \mathcal{H}\{x(t)\} = \frac{1}{\pi} \text{p.v.} \int_{-\infty}^{\infty} \frac{x(\tau)}{t - \tau} d\tau$$
  Instantaneous Envelope: $A(t) = |z(t)| = \sqrt{\Delta\text{EEG}(t)^2 + \mathcal{H}\{\Delta\text{EEG}(t)\}^2}$
* **⚠️ Key Engineering Challenges & Solutions:**
  * **Challenge (Initial Method Failure):** Raw EEG thresholding using baseline variance ($\mu_{\text{base}} + 2.5\sigma_{\text{base}}$) yielded paradoxical results: Healthy recovered in $51.7\text{ ms}$, Fast Gamma incorrectly appeared faster ($34.1\text{ ms}$), and Seizure Spikes measured $70.7\text{ ms}$ instead of $\infty$. High baseline variance in seizure states artificially inflated the recovery threshold.
  * **Solution (Methodological Shift):** Switched to pure differential signal $\Delta\text{EEG}(t) = \text{EEG}_{\text{pulse}}(t) - \text{EEG}_{\text{ctrl}}(t)$ and defined $\tau_{\text{rec}}$ as the time for envelope $A(t)$ to drop permanently below **5% of its peak response**.
* **Key Results:**
  * Healthy Regime: $\tau_{\text{rec}} = \mathbf{334.1\text{ ms}}$ (exponential decay, full resilience).
  * Fast Gamma & Seizure Regimes: $\tau_{\text{rec}} \to \boldsymbol{\infty}$ (sustained response), proving that phase offsets—rather than amplitude attenuation—drive dynamic instability near bifurcation.

---

### `04_Phase_Dependency_PRC`
* **Objective & Overview:**  
  Maps the Phase Response Curve (PRC) to evaluate whether the instantaneous phase of TMS delivery ($\phi_{\text{stim}}$ across $0^\circ\text{--}360^\circ$) dictates the magnitude and direction of phase displacement ($\Delta\phi = \phi_{\text{pulse}} - \phi_{\text{ctrl}}$).
* **Mathematical Formulation:**  
  Infinitesimal phase perturbation relationship:
  $$\frac{d\phi}{dt} = \omega + Z(\phi) \cdot I(t)$$
* **⚠️ Key Engineering Challenges & Solutions:**
  * **Challenge 1 (Type-0 Phase Resetting):** Uniform strong pulses ($1500\text{--}2000\text{ mV}$) wiped out ongoing phase memory, producing flat, uninformative diagonal lines (slope -1). Conversely, ultra-weak pulses ($50\text{ mV}$) failed to trigger non-linear seizure thresholds.
  * **Solution:** Enforced the **Weak Perturbation Condition ($\epsilon \ll \theta_{\text{regime}}$)** tailored to each regime's threshold: Healthy = $150\text{ mV}$, Fast Gamma = $180\text{ mV}$, Seizure Spikes = $350\text{ mV}$.
  * **Challenge 2 (Visual Phase Wrapping Artifacts):** Discontinuities at $\pm 180^\circ$ created artificial vertical boundary lines.
  * **Solution:** Restricted phase displacement wrapping strictly to $[ -180^\circ, +180^\circ ]$ and severed artificial boundary connections in visualization.
  * **Challenge 3 (Hilbert Noise Sensitivity):** Background noise caused jagged PRC curves in healthy baseline.
  * **Solution:** Applied narrow bandpass filtering ($8\text{--}16\text{ Hz}$) prior to Hilbert transformation.
* **Key Results:**
  * Healthy & Fast Gamma: Smooth, symmetric Type-2 sinusoidal PRCs ($\text{Max } \Delta\phi = \mathbf{23.5^\circ}$ and $\mathbf{12.6^\circ}$).
  * Seizure Spikes: **Relaxation Oscillator Behavior**. Phase shift is near zero across $0^\circ\text{--}180^\circ$, but exhibits a massive **$+98.9^\circ$ Phase Advance spike strictly at the $270^\circ$ rising edge**.

---

### `05_Closed_Loop_Seizure_Suppression`
* **Objective & Overview:**  
  Develops a real-time, closed-loop TMS control engine targeting the $270^\circ$ vulnerable rising phase to achieve optimal seizure spike suppression and desynchronization.
* **⚠️ Key Engineering Challenges & Solutions:**
  * **Challenge 1 (Non-Causal Phase Tracking):** Standard Hilbert Transform requires future signal samples, making real-time online tracking impossible.
  * **Solution:** Built a causal online processing pipeline using a sliding time window ($200\text{ ms}$) paired with real-time bandpass filtering ($4\text{--}12\text{ Hz}$).
  * **Challenge 2 (Filter Phase Lag):** Real-time filtering introduced a phase delay, causing pulses triggered at $270^\circ$ to fire late.
  * **Solution:** Implemented phase-lag delay compensation by advancing the internal trigger target to $\approx 255^\circ$, ensuring physical pulse arrival exactly at $270^\circ$.
  * **Challenge 3 (Evaluation Metric Artifacts & Transients):** Measuring full signal RMS created negative efficacy artifacts due to hyperpolarization voltage dips. Furthermore, transient spikes during startup ($t < 1.4\text{ s}$) masked steady-state performance.
  * **Solution:** Evaluated performance strictly in the **Steady-State Regime ($t \ge 1.4\text{ s}$)** using **Peak Spike Height Suppression**.
* **Key Results:**
  * **Baseline Seizure (No Control):** Peak Spike Height = $14.85\text{ mV}$.
  * **Open-Loop ($8\text{ Hz}$ Unsynchronized):** Peak Spike Height = $15.97\text{ mV}$ (**Exacerbation: $-7.5\%$**).
  * **Closed-Loop Single Pulse ($270^\circ$ Target):** Peak Spike Height = $14.78\text{ mV}$ (**Partial Shift: $+0.5\%$**).
  * **Closed-Loop Burst Mode ($270^\circ$ Target):** Peak Spike Height = $2.95\text{ mV}$ (**Attractor Collapse / Suppression: $+\mathbf{80.2\%}$**).

---

## Comprehensive Experimental Benchmarks

### 1. Dynamics & Phase Response Overview Across Regimes

| Cortical Regime | Dominant Dynamics | Recovery Time ($\tau_{\text{rec}}$) | PRC Max Shift ($\Delta\phi_{\text{max}}$) | Vulnerable Target Phase |
| :--- | :---: | :---: | :---: | :---: |
| **1. Healthy Baseline** | Alpha / Beta Oscillations | $334.1\text{ ms}$ | $23.5^\circ$ (Type-2 Sinusoidal) | Non-sensitive |
| **2. Pre-Seizure Fast Gamma** | $40\text{ Hz}$ Narrowband Gamma | $\infty$ (Sustained Shift) | $12.6^\circ$ (Type-2 Sinusoidal) | Uniform / Low |
| **3. Seizure Spikes** | Hypersynchronized Spikes | $\infty$ (Phase Displaced Attractor) | **$+98.9^\circ$ (Phase Advance)** | **$270^\circ$ (Rising Edge)** |

### 2. Steady-State Seizure Control Evaluation ($t \ge 1.4\text{ s}$)

| Control Paradigm | Max Spike Height (mV) | Peak Suppression (%) | Dynamical Outcome |
| :--- | :---: | :---: | :--- |
| **Baseline (Uncontrolled)** | $14.85\text{ mV}$ | Baseline | Trapped in Seizure Limit Cycle Attractor |
| **Open-Loop ($8\text{ Hz}$ Unsynchronized)** | $15.97\text{ mV}$ | **$-7.5\%$** | Entrainment & Worsened Hypersynchrony |
| **Closed-Loop Single Pulse ($270^\circ$)** | $14.78\text{ mV}$ | **$+0.5\%$** | Transient Shift; Attractor Rebounds |
| **Closed-Loop Burst Mode ($270^\circ$)** | **$2.95\text{ mV}$** | **$+\mathbf{80.2\%}$** | **Attractor Collapse & Full Desynchronization** |

---

## Installation & Requirements

1. **Clone the Repository:**
   ```bash
   git clone [https://github.com/Heliya-Rahmatiyan/Neural-Mass-TMS-ClosedLoop.git](https://github.com/Heliya-Rahmatiyan/Neural-Mass-TMS-ClosedLoop.git)
   cd Neural-Mass-TMS-ClosedLoop
   ```

2. **Set Up Environment & Dependencies:**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

### Required Packages
- `python >= 3.9`
- `numpy >= 1.21.0`
- `scipy >= 1.7.0`
- `matplotlib >= 3.4.0`
- `jupyter` / `notebook`

---

## Usage Guide

Execute the Jupyter notebooks in sequential order to replicate the full simulation pipeline and experimental benchmarks:

```bash
jupyter notebook
```

1. `01_jansen_rit_baseline_tms.ipynb`: Initialize RK4 integration and validate baseline parameters.
2. `02_Wendling_NMM_Fast_Inhibition_TMS.ipynb`: Simulate 10D NMM regimes and apply single-pulse TMS.
3. `03_Quantification of Perturbation Recovery.ipynb`: Extract Hilbert analytic envelope and compute $\tau_{\text{rec}}$.
4. `04_Phase_Dependency_PRC.ipynb`: Run 24-bin phase-targeted TMS to construct regime PRCs.
5. `05_Closed_Loop_Seizure_Suppression.ipynb`: Execute online causal phase-locked closed-loop TMS control.

---

## License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.
