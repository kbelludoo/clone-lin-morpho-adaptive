# clone-lin-morpho-adaptive

Experimental **LIN** clone of Morpho Blue `AdaptiveCurveIrm` — the default
borrow-rate model:

`rate = _curve(avgRateAtTarget, err)` with `err` the WAD-normalized distance
from 90% target utilization, `_curve` the two-slope (1/4x … 4x) map, and
`avgRateAtTarget` the trapezoidal integral of `start * wExp(speed * t)`.

**Proofs live in [kbelludoo/lin-open](https://github.com/kbelludoo/lin-open)** — this repo is the LIN copy only.

- **Upstream (hash-pinned, not vendored):** [morpho-org/morpho-blue-irm](https://github.com/morpho-org/morpho-blue-irm) `58ce91fd04840372443a7080a5b32067b17b3d60`
  - `src/adaptive-curve-irm/AdaptiveCurveIrm.sol` sha256 `40ee4764e8eebfee290e07314af38fee7e30323a258d27bdb481e6c00187186d` blob `34cbf8bc7a2489daf1cd4f789d827f4e41967989`
  - License MIT. This repository does **not** vendor the Solidity source.
- **LIN kernel:** [`src/lin_morpho_adaptive_curve.lin`](src/lin_morpho_adaptive_curve.lin) (identical to lin-open)
- **C11 oracle copy:** [`test/oracles/morpho_adaptive_curve_c11.c`](test/oracles/morpho_adaptive_curve_c11.c)
- **Transpilable kernel:** [`test/oracles/morpho_adaptive_curve_kernel.c`](test/oracles/morpho_adaptive_curve_kernel.c)
- **Proofs (lin-open only):** `examples/morpho_adaptive_curve/` and `make morpho-adaptive-curve-proof`
- **Class:** EXPERIMENTAL i64-scale. Not mainnet int256. Not a production IRM replacement.

Named goldens (`WAD=1e18`, `init=1268391679`):

| id | call | value |
|---|---|---|
| TARGET | `_curve(init, 0)` | **1268391679** |
| HIGH | `_curve(init, WAD)` | **5073566716** |
| LOW | `_curve(init, -WAD)` | **317097919** |
| END-1D | 100% util / 86400s endRateAtTarget | **1454044805** |
| AVG-1D | trapezoid avg rate, same inputs | **5438922544** |

`rateAtTarget` and `elapsed` are arguments (Solidity uses a mapping and
`block.timestamp`). Signed 128-bit `wmul` fail-closes if the quotient does
not fit int64; naive i64 `3e18*1e18` wraps. `wExp` fail-closes when `2^q`
would overflow i64 instead of clipping to Solidity's 256-bit upper value.

## Reproduce proofs (in lin-open)

```
make morpho-adaptive-curve-proof
```

## Not claimed

Uint256/int256 / mainnet Morpho Blue market parity.
LIN replacing Morpho, Uniswap, or the EVM. TVL or dollar savings. zk soundness.
Year-scale adaptation (`speed * elapsed` that does not fit int64).
Hidden `rateAtTarget` storage, `block.timestamp`, or `slt`/`sgt` assembly.
