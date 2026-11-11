# geo

Geospatial basics for [Ecko](https://ecko.sh): geohash encode and decode,
great-circle distance and bearing, and bounding boxes. Pure computation - no
capabilities.

## Install

```bash
ecko get github.com/ecko-lang/geo
```

```ecko
import geo
```

## Usage

```ecko
import geo

print(geo.encode(57.64911, 10.40744, 11))  # u4pruydqqvj
print(geo.decode("ezs42"))  # {lat: 42.60498046875, lon: -5.60302734375}

print(round(geo.km(51.5074, -0.1278, 48.8566, 2.3522)))  # 344 - London to Paris
print(round(geo.bearing(51.5074, -0.1278, 48.8566, 2.3522)))  # 148 degrees

box = geo.bbox(51.5074, -0.1278, 5000.0)  # 5 km around central London
print(geo.within(51.5007, -0.1246, box))  # true - Westminster
print(geo.within(51.4769, 0.0005, box))  # false - Greenwich
```

## API

| function | returns |
|---|---|
| `encode(lat, lon, precision)` | the geohash of a point, `precision` (1 to 22) characters long |
| `decode(hash)` | `{ lat, lon }` at the centre of the hash's cell |
| `decode_bbox(hash)` | the hash's cell as `{ min_lat, max_lat, min_lon, max_lon }` |
| `distance(lat1, lon1, lat2, lon2)` | the great-circle distance in metres |
| `km(lat1, lon1, lat2, lon2)` | the great-circle distance in kilometres |
| `bearing(lat1, lon1, lat2, lon2)` | the initial bearing in degrees clockwise from north, 0 to 360 |
| `destination(lat, lon, bearing_deg, metres)` | `{ lat, lon }` reached by travelling `metres` on that bearing |
| `bbox(lat, lon, metres)` | `{ min_lat, max_lat, min_lon, max_lon }` enclosing every point within `metres` |
| `within(lat, lon, box)` | `true` when the point is in the box, edges inclusive |
| `center(box)` | `{ lat, lon }` at the centre of a box |

## Notes

### Errors

| kind | reason | when |
|---|---|---|
| `geo` | `bad_coordinate` | a latitude is outside -90..90 or a longitude outside -180..180 |
| `geo` | `bad_geohash` | a hash is empty or has a character outside the geohash alphabet |
| `bug` | `bad_argument` | a coordinate or distance is not a number, `precision` is not an Int from 1 to 22, `bbox` got a negative radius, or a box is missing a field |

### Geohashes and boxes

**A geohash is an area, not a point.** `decode` gives you the centre because
that is usually what is wanted, but `decode_bbox` is the honest answer. The
shorter the hash, the larger the cell - a 1-character hash spans thousands of
kilometres, a 9-character one about ten metres.

**A shared prefix means proximity.** That is the entire point of the encoding:
`gcpvj0d` and `gcpvj0e` are neighbours, so a prefix search is a bounding-box
query with no spatial index. The one caveat is that the reverse does not hold -
two points either side of a cell boundary are close but share no prefix.

**The Earth is a sphere here**, of mean radius 6371008.8 m. That is within about
0.5% of the real ellipsoid, which is right for "how far apart are these" and
wrong for surveying. If you need Vincenty on WGS84, this is not that package.

**`bbox` is a filter, not an answer.** The box encloses the circle, so it
includes corners that are further away than the radius. Use it to narrow a set
cheaply, then confirm with `distance`.

**Boxes across the antimeridian wrap.** When the circle crosses longitude 180,
`bbox` returns a box whose `min_lon` is greater than its `max_lon`: it runs
east from `min_lon`, across 180, to `max_lon` (the GeoJSON convention).
`within` and `center` read it that way; a database query has to as well, with
`lon >= min_lon OR lon <= max_lon`. When the circle reaches a pole, the box
spans every longitude.

**Geohash input is case-insensitive.** `decode("EZS42")` is `decode("ezs42")`;
`encode` always writes lower case.

## Testing

```bash
ecko test
```

## License

[MIT](LICENSE)
