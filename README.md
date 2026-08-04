# geo - Ecko Std Lib Package

Geospatial basics: geohash encode/decode, great-circle distance and bearing,
and bounding boxes.

Pure computation - no capabilities.

## Install

```bash
ecko get github.com/ecko-lang/geo
```

```ecko
import geo
```

## Usage

```ecko
geo.encode(57.64911, 10.40744, 11)   # "u4pruydqqvj"
geo.decode("ezs42")                  # { lat: 42.605, lon: -5.603 }
geo.decode_bbox("ezs42")             # the cell, not just its centre

geo.km(51.5074, -0.1278, 48.8566, 2.3522)      # 343.5 - London to Paris
geo.bearing(51.5074, -0.1278, 48.8566, 2.3522) # 148 degrees
geo.destination(51.5074, -0.1278, 90.0, 100000.0)

box = geo.bbox(51.5074, -0.1278, 5000.0)
geo.within(lat, lon, box)
```

## API

| function | what it does |
|---|---|
| `encode(lat, lon, precision)` | geohash of that many characters |
| `decode(hash)` | `{ lat, lon }` at the centre of the cell |
| `decode_bbox(hash)` | `{ min_lat, max_lat, min_lon, max_lon }` |
| `distance(lat1, lon1, lat2, lon2)` | great-circle metres |
| `km(...)` | the same in kilometres |
| `bearing(lat1, lon1, lat2, lon2)` | initial bearing, degrees from north |
| `destination(lat, lon, bearing, metres)` | where you arrive |
| `bbox(lat, lon, metres)` | box enclosing that radius |
| `within(lat, lon, box)` | point-in-box, edges inclusive |
| `center(box)` | the centre of a box |

## Notes

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

## Testing

```bash
ecko test
```

Offline and deterministic. The geohash cases are the canonical published
examples; the distances are values derivable from the sphere itself - a degree
of longitude at the equator, and half a great circle - rather than numbers read
off this implementation.

## License

MIT
