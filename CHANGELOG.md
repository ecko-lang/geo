# Changelog

## 0.39.0

### Fixed

- `bbox` now contains every point within the radius when the circle crosses
  the antimeridian. Before, the box was clipped at 180, so a point 2 km away on
  the other side was outside it.
- `bbox` at high latitudes now contains the whole circle. The longitude span
  was too narrow there (at 80 degrees north with a 500 km radius, points on
  the circle fell outside the box). A circle that reaches a pole now spans
  every longitude.
- `bbox` with a negative radius fails with `bug`/`bad_argument` instead of
  returning an inverted box.
- `encode` with a precision that is not an Int from 1 to 22 (`1.5`) fails with
  `bug`/`bad_argument` instead of quietly rounding.
- `decode` and `decode_bbox` accept upper-case geohashes.
- `within` and `center` given a map that is not a box fail with
  `bug`/`bad_argument` naming the function, instead of "map has no key".
- Errors carry a `reason` (`bad_coordinate`, `bad_geohash`) and name the
  function, and a coordinate that is not a number is a `bug`/`bad_argument`.

### Breaking

- A box from `bbox` that crosses the antimeridian now has `min_lon` greater
  than `max_lon`. Migration: if you filter with `min_lon <= lon <= max_lon`
  yourself, use `geo.within`, or test `lon >= min_lon or lon <= max_lon` when
  `min_lon > max_lon`.
