---
name: numpy-flint-arb
description: "The package to use flint's arb and acb types as numpy arrays"
---

Import `numpy_flint_arb.np` instead of `numpy`:

```python
from numpy_flint_arb import np

A = np.random.normal(size=(2, 2))
b = np.random.normal(size=(2,))
x = np.linalg.solve(A, b)
b_approx = A @ x
assert np.all(np.contains(b_approx, b))
```

### `asarray()` and Input Check

To avoid mixing ordinary floats like `float` or `np.float`, `flarray` for `arb`, `acb` only accepts integers, `arb` or `acb` and `flarray` for `arf` only accepts integers and `arf, arb`.

To relax this, `allow_input()` may be used:

```python
import pytest
from numpy_flint_arb import allow_input
from flint import arb, arf

# arb array
with pytest.raises(Exception):
    np.asarray(0.5, dtype=arb)
with pytest.raises(Exception):
    with allow_input(float=True):
        np.asarray(0.5, dtype=arb)
with allow_input(interval=True, float=True):
    np.asarray(0.5, dtype=arb)

# arf array
with pytest.raises(Exception):
    np.asarray(0.5, dtype=arf)
with allow_input(float=True):
    np.asarray(0.5, dtype=arf)
with allow_input(interval=True, float=True):
    np.asarray(0.5, dtype=arf)
```

Note that `allow_input()` does not affect for `arf()`, `arb()`, `acb()` constructors, but only for `np.asarray()` and `flarray()`.

#### `str` input

One can input `str` to `asarray()`.
If `dtype` is not specified, it will be automatically detected as `arb`. However, specifying `dtype` explicitly is recommended.

#### `asarray(dtype=acb)`

`asarray()` does not support separated input for real and imaginary parts.

Do the following instead, as `acb(1j)` is exact.

```python
from flint import arb, acb
from numpy_flint_arb import np

with pytest.raises(Exception):
    # python-flint does not support single argument str input for acb
    np.asarray("[0.5 +/- 0.001] + [0.5 +/- 0.001]j", dtype=acb)
with pytest.raises(Exception):
    # This is inexact and raises an error without allow_input()
    np.asarray(0.5 + 0.5j, dtype=acb)
with pytest.raises(Exception):
    # Mixing complex and arb is not supported by python-flint
    np.asarray("0.5 +/- 0.001", dtype=arb) + 1j * np.asarray("0.5 +/- 0.001", dtype=arb)
```

```python
>>> # This is possible but not recommended
>>> np.asarray("0.5 +/- 0.001", dtype=arb) + 1j * np.asarray("0.5 +/- 0.001", dtype=acb)
flarray([0.50 +/- 1.01e-3] + [0.50 +/- 1.01e-3]j,
        dtype=<class 'flint.types.acb.acb'>)
>>> # Recommended
>>> np.asarray("0.5 +/- 0.001", dtype=arb) + acb(1j) * np.asarray("0.5 +/- 0.001", dtype=arb)
flarray([0.50 +/- 1.01e-3] + [0.50 +/- 1.01e-3]j,
        dtype=<class 'flint.types.acb.acb'>)
```

## `fft` submodule

Some `fft` functions are implemented.

```python
>>> np.fft.fft(np.arange(1, stop=4))
flarray([6.00000000000000,
         -1.50000000000000 + [0.86602540378444 +/- 1.96e-15]j,
         -1.50000000000000 + [-0.86602540378444 +/- 1.96e-15]j],
        dtype=<class 'flint.types.acb.acb'>)
```

## `linalg` submodule

Some `linalg` functions are implemented.

```python
>>> A = np.asarray([[1, 2], [3, 4]], dtype=arb)
>>> np.linalg.inv(A)
flarray([[[-2.00000000000000 +/- 1.63e-15],
          [1.00000000000000 +/- 7.41e-16]],
         [[1.50000000000000 +/- 9.44e-16],
          [-0.500000000000000 +/- 4.17e-16]]],
        dtype=<class 'flint.types.arb.arb'>)
```

Internally 2 functions `tomat()` and `frommat()` are added to treat `flarray` as array of `arb_mat` / `acb_mat`, so that we can perform matrix operations like `np.linalg.solve` on `flarray`.

## `random` submodule

Since `python-flint` does not support random number generation, the `random` module just uses `np.random`.
Therefore, the return values may not be random up to the precision of `arb`, `acb`.

## `special` submodule

Some `scipy.special` functions are implemented.

```python
>>> np.special.jv(np.arange(2), arb(1))
flarray([[0.765197686557966 +/- 6.11e-16],
         [0.440050585744933 +/- 5.55e-16]],
        dtype=<class 'flint.types.acb.acb'>)
```

## What it does

- This package adds a `flarray` which [subclasses `ndarray`](https://numpy.org/doc/stable/user/basics.subclassing.html) in order to
  - Override `__array_namespace__` to `numpy_flint_arb.np`
  - Override `dtype` to return newly added `_fl_dtype` private attribute, since the actual internal dtype `object` cannot be overridden.
  - Override `__array_finalize__` as recommended by the NumPy docs to return `flarray` with proper `_fl_dtype` instead of `ndarray` after Numpy operations.
- Partially supports `linalg` and `(scipy.)special` functions.
-
- Does not perform any parallelization to avoid complexity and to fully utilize the great `python-flint` library
  - Using `arb_series` and `acb_series` may be faster for additions but this is too hacky.
  - Defining custom `dtype` is way too complicated
  - Writing C extension would be theoretically also possible but is still too complicated.
- Does not support `in` operator since it tries to convert the return value to bool. Use newly added `np.contains(x, y)` and `np.overlaps(x, y)` instead.
- Currently `dtype` of resulting `flarray` is inferred from `type(output.flat[0])`. If the output array is empty or its element type is inconsistent, the result will be inaccurate.

  ```python
  >>> (np.asarray([0], dtype=arb) * acb(1j)).dtype
  <class 'flint.types.acb.acb'>
  >>> (np.asarray([], dtype=arb) * acb(1j)).dtype
  <class 'flint.types.arb.arb'>
  ```
