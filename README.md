# Route Planner

A C++ walking-tour generator. Give it a street map and an ordered list of places to
visit, and it prints turn-by-turn walking directions between them with a short
commentary at each stop.

```
Starting tour...
Welcome to <point of interest>!
<talking points for that stop>
Proceed <distance> miles <direction> on <street>
Take a <left|right> turn on <street>
...
Your tour has finished!
Total tour distance: <distance> miles
```

University coursework project. `base_classes.h` and `stops.h` define the required
interfaces; everything else is my implementation.

## How it works

| Piece | What it does |
|---|---|
| `GeoDatabase` | Loads the map into three hash maps: point of interest to coordinate, coordinate to its connected coordinates, and coordinate-pair to street name. Segments with points of interest get a midpoint node so the tour can branch off the street to reach them. |
| `Router` | A* shortest path between two coordinates, using great-circle (haversine) distance as the heuristic. |
| `TourGenerator` | Chains routes stop to stop and converts each waypoint pair into a `proceed` or `turn` instruction, deciding turns from the angle between consecutive segments. |
| `HashMap` | Hand-rolled separate-chaining hash map with a 0.75 max load factor and rehashing. No `std::unordered_map`. |

Consecutive `proceed` steps along the same street are merged before printing, so
you get "Proceed 0.4 miles north on Broxton Avenue" rather than one line per node.

## Build and run

```bash
g++ -std=c++11 -O2 *.cpp -o route_planner
./route_planner
```

The two input file paths are hardcoded in `main()` (`geodb.load(...)` and
`stops.load(...)`). Edit them to point at your own files before building. The data
files themselves are not in this repo.

## Input formats

`mapdata.txt`, repeating records:

```
<street name>
<start lat> <start lon> <end lat> <end lon>
<N = number of points of interest on this segment>
<poi name>|<poi lat> <poi lon>        (repeated N times)
```

`stops.txt`, one stop per line in visit order:

```
<point of interest name>|<what to say when you arrive>
```

Every point of interest named in `stops.txt` must exist in `mapdata.txt`, or the
tour comes back empty.
