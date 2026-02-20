# Steam ↔ CSFloat Arbitrage (Balance Flow) — Vue 3 + TypeScript

A lightweight calculator to simulate a **two-leg balance flow** between **Steam Wallet** and **CSFloat**.

It follows this model:

**Steam Wallet → (Leg 1) → CSFloat Balance → (Leg 2) → Steam Wallet**

**Net Profit = Final Steam Wallet − Initial Steam Wallet**

This project is intentionally simple: you enter **per-item values** (including fees already baked into the “net” inputs), and the app shows balances, profit, and ROI.

---

## Features

- ✅ Two-leg flow (Steam → CSFloat, CSFloat → Steam)
- ✅ Net Profit + ROI%
- ✅ “Not enough balance” warnings
- ✅ **Card height does not jump** when warnings appear
- ✅ **EN / TR language toggle** (saved in LocalStorage)
- ✅ **Autosave** inputs via LocalStorage (persists after refresh)
- ✅ Number input spinners removed (clean UI)

---

## How the Math Works

### Leg 1 (Steam → CSFloat)
- **Steam Spend** = `qty1 × steamBuyPerItem`
- **CSFloat Gain (net)** = `qty1 × csfloatNetPerItem`

Balances after Leg 1:
- `steam1 = initialSteam − steamSpend`
- `csfloat1 = initialCsfloat + csfloatGain`

### Leg 2 (CSFloat → Steam)
- **CSFloat Spend** = `qty2 × csfloatBuyPerItem`
- **Steam Gain (net)** = `qty2 × steamNetPerItem`

Final balances:
- `steam2 = steam1 + steamGain`
- `csfloat2 = csfloat1 − csfloatSpend`

### Final Result
- **Net Profit** = `steam2 − initialSteam`
- **ROI%** = `netProfit / initialSteam × 100`

> Note: This app measures profit **only in Steam Wallet** (because it compares initial vs final Steam).  
> If you want “total wealth” profit (Steam + CSFloat), you can extend the formula to:
> `(finalSteam + finalCSFloat) − (initialSteam + initialCSFloat)`.

---

## Tech Stack

- **Vue 3**
- **TypeScript**
- **Vite**

---

## Getting Started

### 1) Install dependencies
```bash
npm install
