---
name: flint
description: "The basic usage of flint, optionally interval arithmetic multiprecision arithmetic library"
---

- Never use Python float. Use `flint.arb` (real), `flint.acb` (complex). Use `flint.arf` (real) if only real upper bound / lower bound is needed. There is no `flint.acf` (complex), use `flint.acb` even if only complex upper bound / lower bound is needed.
- Do not try to reinvent existing simple algorithms as much as possible. However, rootfinding method is not implemented in python-flint, so you may implement it yourself.
- For interval-related methods or python-flint specific methods, use with `np.vectorize`.
  - `flint.arb` has some special interval-related methods
    - `mid()`, `rad()`
    - `upper()`, `lower()`
    - `abs_upper()`, `abs_lower()`
    - `union()`, `intersection()`
    - `contains()`, `overlaps()`
  - `flint.acb` has less special methods.
    - `mid()`, `rad()`
    - `abs_upper()`, `abs_lower()`
    - `union()` (No `intersection()`!)
    - `contains()`, `overlaps()`
- If you have used flint-specific methods in the function, the arguments type should be `numpy_flint_arb.flarray` instead of `array_api.latest.Array`.
- If you really need reference, read https://python-flint.readthedocs.io/en/latest/arb.html and https://python-flint.readthedocs.io/en/latest/acb.html how to use `flint.arb` and `flint.acb`.
