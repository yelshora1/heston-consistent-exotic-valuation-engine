## Concept and Branding

The underlying conceptual design, payoff visualization structure, and educational framework of Pryce are intellectual creations of **Yousef Elshora (2025)**.
While the source code is available under the Apache 2.0 License, the conceptual framework, architecture, and brand identity of Pryce are protected under copyright and may not be replicated or presented as an original work without express permission.

# Pryce

**Pryce** is a Python-based **exotic options pricing and visualization engine**.  
It models derivatives such as **Asian**, **Barrier**, **Chooser**, and **Compound** options under a **calibrated Heston stochastic volatility framework** — blending quantitative finance with clarity and experimentation.

> Developed by **Yousef Elshora (2025)**

---

## Overview

Pryce is designed as both a **learning tool** and a **research platform** for exploring the dynamics of exotic options.  
Its goal is to make complex payoffs *tangible* — allowing a user to move from vanilla pricing theory to exotic structures without losing mathematical transparency.

The engine:
- Calibrates the **Heston model** to real market implied volatility data (via Yahoo Finance)
- Simulates **stochastic paths** for the underlying asset under risk-neutral dynamics
- Prices exotics through **Monte Carlo** and **closed-form characteristic-function** approaches
- Outputs both **symbolic payoffs** and **numerical fair values**

---

## Implemented Modules

| Category | File | Description |
|-----------|------|-------------|
| Core model | `heston/charfunc.py` | Heston characteristic function |
|  | `heston/vanilla_cf.py` | Closed-form vanilla option pricer |
|  | `heston/paths.py` | Monte Carlo path simulation |
| Calibration | `heston/calibrate_heston.py` | Fits Heston parameters to market IV data |
| Exotics | `exotics/asian.py` | Arithmetic-average Asian call |
|  | `exotics/kibarrier.py` | Up-and-In barrier call |
|  | `exotics/chooser.py` | European chooser option |
|  | `exotics/compound.py` | Call-on-call compound option |
| Data | `data/options_chain.py` | Fetches and filters live option chains |
| Utility | `utils/iv.py` | Black-Scholes + implied vol routines |
| CLI | `run_pryce.py` | Unified command-line runner |

---

## Example Usage

```bash
# Calibrate Heston to AAPL and price a chooser option
python run_pryce.py AAPL 2026-02-20 260 chooser

