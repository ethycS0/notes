# def_pkg

File: `includes/def_pkg.sv` (175 lines).

## Purpose

Top-level ISA/extension configuration and the generic exception type used across
the core.

## Key contents

- Extension enables `:26`–`:42`: `RVF`, `RVD`, `RVA`, `RVV` (disabled by
  `DISABLE_SIMD`), `RVH`.
- Transprecision/float format constants `:44`–`:79`.
- `FP_PRESENT` / `V_PRESENT` `:62`/`:64`; `FLEN` `:67`.
- `ISA_CODE` `:81`.
- Commit-log flag `ENABLE_SPIKE_COMMIT_LOG` `:98`.
- Status read/write masks `:101`–`:150`.
- `exception_t` `:153` — `{cause, tval, valid}`.
- MMU mode constants `MODE_SV39`, `MODE_OFF` `:160`; `PPN4K_WIDTH` `:166`.
- `core_type_t` `:168`.

## Related

[[def_pkg]] · [[drac_pkg]] · [[riscv_pkg]]
