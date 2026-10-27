# NumPy

Current stable: 2.x. Sources linked per item.

## Integers overflow in silence

- Fixed-size integer arrays wrap around: `np.power(100, 9, dtype=np.int32)` is `-1486618624`, and casting follows C
  rules (`int64` 300 → `int8` 44). NumPy warns on scalar overflow but not on arrays. Check the range with
  `np.iinfo` / `np.finfo` and choose the dtype before the arithmetic.
  https://numpy.org/doc/stable/user/basics.types.html · https://numpy.org/doc/stable/reference/arrays.promotion.html
- `int64` combined with `uint64` promotes to `float64`, losing precision above 2^53.

## NumPy 2 promotion (NEP 50)

- Python scalars are weak: `np.float32(3) + 3.0` stays `float32`, and `np.int8(1) + 1000` raises `OverflowError`.
  Code that relied on upcasting to `float64` loses precision.
- The default integer is 64-bit on every 64-bit platform.
- Removed aliases: `np.float_`, `np.complex_`, `np.NaN`, `np.Inf`, `np.string_`, `np.unicode_`, `np.product`,
  `np.cumproduct`, `np.round_`, `np.alltrue`, `np.sometrue`, `np.cast`, `np.asfarray`. `np.array(x, copy=False)`
  becomes `np.asarray(x)`. ruff rule `NPY201` finds them.
- https://numpy.org/doc/stable/numpy_2_0_migration_guide.html

## Missing values and floats

- `np.sum` and `np.mean` propagate `nan`; `np.nansum` returns 0 on an all-`nan` slice and `np.nanmean` returns `nan`
  with a `RuntimeWarning`. Pass `dtype=np.float64` to `nanmean` on `float32` data.
  https://numpy.org/doc/stable/reference/generated/numpy.nanmean.html ·
  https://numpy.org/doc/stable/reference/generated/numpy.nansum.html
- Compare floats with `np.isclose(a, b, rtol=…, atol=…)`: it tests `|a - b| <= atol + rtol * |b|`, is asymmetric, has
  `atol=1e-08` (too loose for values far below 1), and treats `nan` as unequal unless `equal_nan=True`.
  https://numpy.org/doc/stable/reference/generated/numpy.isclose.html

## Shapes and memory

- Broadcasting aligns trailing dimensions, which must match or be 1; a broadcast can materialize a huge
  intermediate. https://numpy.org/doc/stable/user/basics.broadcasting.html
- Basic slicing returns a view, and writing to it changes the original; advanced (fancy or boolean) indexing returns
  a copy. `reshape` and `ravel` return views when they can, `flatten` always copies; `.base is None` means a copy.
  https://numpy.org/doc/stable/user/basics.copies.html

## Randomness

`rng = np.random.default_rng(seed)` (PCG64) is the recommended generator, passed explicitly to the code that draws;
`np.random.seed` and `RandomState` are legacy. https://numpy.org/doc/stable/reference/random/index.html
