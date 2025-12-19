# Heston Exotic Valuation Engine
Project uploaded to GitHub OCTOBER. Started in my local device around MAY.
A Python pricing engine that calibrates the **Heston stochastic volatility model** to live market option data and uses those parameters to price path-dependent exotic derivatives via Monte Carlo simulation.

## What It Does

1. **Fetch live market data** from Yahoo Finance (option chains + spot prices)
2. **Calibrate Heston model** to fit the implied volatility surface
3. **Price exotic options** using calibrated parameters:
   - **Asian** – payoff based on average spot over time
   - **Barrier (knock-in)** – option activates only if spot hits a barrier
   - **Chooser** – holder chooses call or put at an intermediate date
   - **Compound** – call-on-call (option on an option)

Available as both **CLI** (`run_pryce.py`) and **REST API** (`api.py`).

## Repository Layout

```
.
├── run_pryce.py            # Command-line entry point for calibration + pricing
├── api.py                  # FastAPI service exposing calibration and pricing endpoints
├── heston/                 # Core model components (characteristic fn, calibration, simulation)
├── exotics/                # Monte Carlo payoffs for supported exotic contracts
├── data/options_chain.py   # Yahoo Finance data access and preprocessing utilities
├── utils/iv.py             # Black–Scholes and implied volatility helpers
├── scripts/                # Convenience scripts (e.g. inspect option chains)
└── tests/                  # Pytest-based sanity checks for numerical routines
```

## Prerequisites

| Requirement | Notes |
|-------------|-------|
| Python 3.10+ | Development is tested against Python 3.10 and 3.11. |
| C compiler toolchain | Required by SciPy/NumPy wheels on some platforms. |
| Internet access | Needed at runtime to pull market data from Yahoo Finance. |

The project depends on the following PyPI packages:

```
numpy
scipy
pandas
yfinance
fastapi
uvicorn[standard]
```

Install them into a virtual environment to isolate the toolchain from your system Python.

## Setup & Installation

### 1. Clone & Create Virtual Environment

```bash
git clone https://github.com/yelshora1/heston-exotic-valuation-engine.git
cd heston-exotic-valuation-engine

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
```

### 2. Install Dependencies

```bash
pip install numpy scipy pandas yfinance fastapi uvicorn
```

Optional (for testing):
```bash
pip install pytest
```

## Quick Start

### CLI: Price a Single Option

The command-line tool calibrates the Heston model to market data and prices one exotic option.

```bash
python run_pryce.py <TICKER> <EXPIRY> <STRIKE> <TYPE>
```

**Arguments:**
- `TICKER` – Stock symbol (e.g., `AAPL`)
- `EXPIRY` – Expiration date in `YYYY-MM-DD` format (must be an actual exchange date)
- `STRIKE` – Strike price (in dollars, per share)
- `TYPE` – Exotic type: `asian`, `barrier`, `chooser`, or `compound`

**Example: Price an ATM Asian call on AAPL**

```bash
python run_pryce.py AAPL 2026-03-20 270 asian
```

**Output:**
```
Calibrating Heston to AAPL 2026-03-20 ...

=== Result ===
AAPL Asian option (K=270.0, Expiry=2026-03-20)
Spot / T    : 272.190002 / 0.250170y
Model Price : 10.426306
Std. Error  : 0.081396
Params used : HestonParams(kappa=1.553, theta=0.233, sigma=0.851, rho=-0.395, v0=0.075, r=0.01, q=0.0)
```

**Important:** The price is per share. Multiply by 100 for the actual contract cost. Above: $10.43 per share = **$1,043 per contract** (controls 100 shares).

**JSON output** (for scripting):
```bash
python run_pryce.py AAPL 2026-03-20 270 asian --json
```

### REST API: Run the Web Service

Launch an HTTP API for calibration and pricing:

```bash
uvicorn api:app --reload
```

Then visit `http://localhost:8000/docs` for interactive API documentation (Swagger UI).

**Main endpoints:**

- `GET /api/calibrate?ticker=AAPL&expiry=2026-03-20`  
  Calibrates Heston to the option chain, returns fitted parameters + IV surface comparison

- `GET /api/price?type=asian&ticker=AAPL&expiry=2026-03-20&K=270&paths=10000&steps=126`  
  Prices the exotic using calibrated parameters

See `api.py` for all parameters and response formats.

### Other Exotic Types

Try the other supported payoffs:

```bash
# Barrier (knock-in) option
python run_pryce.py AAPL 2026-03-20 270 barrier

# Chooser option (choice at 25% of time-to-expiry)
python run_pryce.py AAPL 2026-03-20 270 chooser

# Compound call-on-call option
python run_pryce.py AAPL 2026-03-20 270 compound
```

## How It Works

### Workflow

1. **Data Ingestion** – Download option chain from Yahoo Finance, filter by moneyness
2. **Calibration** – Fit Heston parameters to market implied volatilities using numerical optimization
3. **Path Generation** – Simulate correlated asset/variance paths using Euler discretization
4. **Payoff Valuation** – Evaluate exotic payoff on each path and average (Monte Carlo)
5. **Discounting** – Discount to present value

### Monte Carlo Convergence

Standard error is reported with each price. Adjust convergence via:
- `n_paths` – number of simulated paths (more = lower error, slower)
- `n_steps` – time steps per path (more = more accurate, slower)

These are hardcoded in `run_pryce.py` and tunable in `api.py` via query parameters.

## Market Data & Internet Requirements

- **Live data required**: The engine fetches option chains from Yahoo Finance at runtime
- **Valid expirations only**: Use actual exchange expirations (see error message for available dates)
- **Time-to-expiry constraint**: Expiry must be > 10 days for reliable calibration

To run offline, capture the JSON from `data.options_chain.get_chain()` and modify the calibrator to use saved data.

## Testing

```bash
pytest
```

Validates implied-volatility calculations and core Heston path properties.

## Repository Layout

```
.
├── run_pryce.py                    # CLI: calibrate + price one exotic
├── api.py                          # FastAPI service: /api/calibrate, /api/price
├── heston/
│   ├── params.py                   # HestonParams dataclass
│   ├── calibrate_heston.py         # Calibration pipeline
│   ├── vanilla_cf.py               # Heston characteristic function (van Haastrecht–Lord)
│   ├── paths.py                    # Euler discretization for paths
│   └── test_heston.py              # Heston validation tests
├── exotics/
│   ├── asian.py                    # Arithmetic Asian call
│   ├── kibarrier.py                # Knock-in barrier call
│   ├── chooser.py                  # European chooser (call/put choice)
│   └── compound.py                 # Compound call-on-call
├── data/
│   └── options_chain.py            # Yahoo Finance data fetching + filtering
├── utils/
│   └── iv.py                       # Black–Scholes IV conversion
└── tests/                          # Pytest unit tests
```

## Adding New Exotic Payoffs

1. Create a new file in `exotics/` with a `price_<name>_mc()` function
2. Add the routing logic to both `run_pryce.py` and `api.py`
3. Add test cases to `tests/`

All pricers follow the same signature:
```python
def price_<exotic>_mc(S0, K, T, params, n_paths=10000, n_steps=252, seed=None):
    # Simulate paths, compute payoff, return (price, std_error)
    return price, se
```

## Limitations & Notes

- **Heston only** – No alternative volatility models (SABR, local-vol, etc.)
- **Knock-in barrier only** – Knock-out not implemented
- **European exotics** – All options are European (American features not priced)
- **Single currency** – No multi-currency or FX derivatives
- **No dividends** – Dividend yields set to 0 in calibration (but tunable in `HestonParams`)

## License

Apache 2.0
