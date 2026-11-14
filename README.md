# Alpha — Quant Cockpit for Alpha Xpedition

A single-page decision-support engine for the **Alpha Xpedition** trading challenge. It turns the published rules of the game — scoring, margin, spillover, envelopes, dice, the IPO and the options session — into live calculators, so every trade can be checked against the math before it is placed.

> **Advisory only.** The competition allows manual trading only; scripts, bots, browser automation and AI agents operating the desk are prohibited. This page never touches the trading terminal. Every order is placed by hand.

## Run it

No build, no dependencies. Open `index.html` in any browser.

To host it, enable **GitHub Pages** on this repo (Settings → Pages → deploy from `main`, root folder) and it will be served at `https://sreesh-codes.github.io/Alpha/`.

Inputs are saved in your browser's local storage on that machine only.

## Modules

| Tab | What it does |
|---|---|
| **Desk & risk** | Position book, live net equity, return, drawdown (ratcheting peak), margin ratio, score and final score, stress test, hard-stop rules |
| **Spillover** | Propagates a headline move through the published coefficients — same nation 25%, same sector elsewhere 15%, nation in tension −20%, 3% cap — ranked, with the effect on your book and score |
| **Impact & fade** | Per-asset round-trip cost and the minimum deviation worth fading under mean reversion; square-root impact model fitted from logged fills; order-slicing bounds |
| **Events & envelopes** | Bayesian posterior from published odds plus envelope evidence; drawdown-capped sizing; Accord fine exposure; 120-second watch timer; reroll vote advisor |
| **Options** | Implied-vol solver from a house quote vs a mixture-of-lognormals jump model priced from the published odds, across all six PTRV series and straddles |
| **IPO** | Comparables valuation with haircut, winner's-curse shading, 3-bid ladder, cash-lock check, uniform-price clearing simulator with pro-rata at the margin |
| **Black swan** | Book P&L and score under each d6 face (with spillover), worst case, and a minimax single-instrument hedge solver |
| **Playbook** | Doctrine, session cards, envelope and reroll protocol, honours worth targeting, desk roles, open questions for the organisers |

## The math

**Score.** `R = N / 1,000,000 − 1`, `S = R / max(D, 5%)`, `S_final = S − 0.3(n + p)|S|`, where `N` is net equity (equity minus bailout injections), `D` the worst fall from a running peak, `n` bailouts and `p` bot penalties.

The 5% floor is the central insight: below it, drawdown costs nothing (`S = 20R`). Above it, one more point of drawdown needs `ΔR/R ≥ ΔD/D` to break even — the first point past the floor is the most expensive.

**Margin.** `E = cash + Σ q·P`, `G = Σ |q·P|`, `m = E/G`. Warning below 0.35, margin call below 0.25. For a directional book, an adverse move of `x` on gross gives `m' = m − x`, so the distance to a call is `m − 0.25` of gross notional.

**Fade threshold.** With reversion half-life `τ` and hold `t`, the fraction of a deviation that reverts is `1 − 2^(−t/τ)`, so the minimum deviation worth fading is `d* = round-trip cost / (1 − 2^(−t/τ))`.

**Impact.** `slippage = k · (notional / depth)^α`, with `depth = 5 × max order` (one order may be at most 20% of depth). `k` and `α` are fitted by log-log least squares from observed fills.

**Envelopes.** Each envelope names the true outcome with probability `θ`, otherwise one of the other `k − 1` outcomes uniformly. Posterior `∝ p_i · Π L(E_j | T = i)`. Tradeable edge is measured against the prior, not zero: `edge = Σ (posterior_i − prior_i) · m_i`.

**Options.** Black–Scholes with `r = 0` and volatility entered as total standard deviation to expiry (no annualisation). The house's "one smooth volatility" is solved by bisection from a single quote; the jump model is `V = Σ p_i · BS(S₀(1 + m_i), K, σ_resid)`.

## Inputs to verify before relying on it

- **Sector and volatility class** are confirmed for ZAP only; the other assets are inferred from their names. Spillover depends entirely on this map — check it against the board.
- **Bond prices** are placeholders at 100.
- **Impact parameters** are uncalibrated defaults until three fills are logged.
- **Envelope truth rate θ** defaults to 0.6 and is unknown — keep a tally through the day.
- **Option jump magnitudes, black swan shocks and IPO comparables** are placeholder reads, not facts.
