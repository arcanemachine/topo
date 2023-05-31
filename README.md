# Geometry Library for Elixir

![Build Status](https://github.com/pkinney/topo/actions/workflows/ci.yaml/badge.svg)
[![Hex.pm](https://img.shields.io/hexpm/v/topo.svg)](https://hex.pm/packages/topo)

A Geometry library for Elixir that calculates spatial relationships between two
geometries. Geometries can be of any of the following types:

- Point
- LineString
- Polygon
- MultiPoint
- MultiLineString
- MultiPolygon

## Installation

```elixir
defp deps do
  [{:topo, "~> 0.5"}]
end
```

## Usage

**[Full Documentation](https://hexdocs.pm/topo/Topo.html)**

The `Topo` module provides functions for determining the relationship between
two geometries. Each function returns a boolean and accepts any combination of
Point, LineString, Polygon, MultiPoint, MultiLineString, or MultiPolygon.

- **`intersects?`** - Geometries **A** and **B** share at least one point in
  common.
- **`disjoint?`** - Disjoint geometries share no points in common. This is the
  direct opposite of the `intersects?` result.
- **`contains?`** - All points of geometry **B** lie within **A**. See section
  below on [Contains].
- **`within?`** - This is the direct inverse of `contains?`. All points of
  geometry **A** lie within geometry **B**.
- **`equals?`** - Geometries **A** and **B** are equivalent and cover the exact
  same set of points. By definition, **A** and **B** are equal if **A** contains
  **B** and **B** contains **A**. Equality does not necessarily mean that the
  geometries are of the same type. A Point **A** is equal to a MultiPoint that
  contains only the same Point **A**.

Each of these functions can be passed any two Geometries in either a Map with a
`:type` and `:coordinates` keys or as a struct generated via the [Geo library](https://github.com/bryanjos/geo). Coordinates are represented as atoms `{x, y}`
and multiple coordinates as Lists.

```elixir
a = %{type: "Polygon", coordinates: [[{2, 2}, {20, 2}, {11, 11}, {2, 2}]]}
b = %Geo.Polygon{coordinates: [[{2, 2}, {20, 2}, {11, 11}, {2, 2}]]}

Topo.equals? a, b # => true
```

Instead of a Point geometry, just a single coordinate can be used.

```elixir
a = %{type: "Polygon", coordinates: [[{2, 2}, {20, 2}, {11, 11}, {2, 2}]]}

Topo.intersects? a, {4, 6} # => true
```

The `Topo` library's functions will automatically attempt to "clean" geometries
passed to them:

- Linear Rings (including Polygons) will be reordered to a counter-clockwise
  direction.
- Polygon's Linear Rings will automatically be closed if the first point is not
  repeated as the last point.
- Points that are equal or collinear with surrounding points are removed from
  LineStrings or Polygons.

## A note on `contains?`

There are a few non-obvious special cases that are worth mentioning:

- A Polygon does not contain its own boundary. Specifically a LineString that
  is the exact same as a Polygon's exterior Linear ring is not contained within a
  that Polygon.

```elixir
a = %Geo.Polygon{coordinates: [[{2, 2}, {20, 2}, {11, 11}, {2, 2}]]}
b = %Geo.LineString{coordinates: [{2, 2}, {20, 2}, {11, 11}, {2, 2}]}

Topo.contains? a, b # => false
Topo.intersects? a, b  # => true
```

- A LineString does not contain it's own first and last point (unless those
  points are the same, as in a LinearRing)

```elixir
a = %Geo.LineString{coordinates: [{1, 3}, {2, -1}, {0, -1}]}
b = %Geo.LineString{coordinates: [{1, 3}, {2, -1}, {0, -1}, {1, 3}]}

Topo.contains? a, {1, 3} # => false
Topo.intersects? a, {1, 3} # => true
Topo.contains? b, {1, 3} # => true
```

## Float Precision Issues

It is possible that floating point math imprecision can cause incorrect results for certain inputs. This is often encountered during the line segment comparison (see [LineStringPolygonTest](https://github.com/pkinney/topo/blob/master/test/linestring_polygon_test.exs) for an example).  By default, `Topo` is strict on intersection math; however, if you with to allow a less strict requirement for line segment intersection, you can set an `:epsilon` value at compile time, which will be passed to the [SegSeg](https://github.com/pkinney/segseg_ex) library (see [here](https://github.com/pkinney/segseg_ex#float-precision-issues) for a more detailed explanation).

In your application's config file add

```elixir
config :topo, epsilon: true
```

Topo uses the `Application.config_env/3` function to avoid querying the value on each computation, so you may have to clean and recompile the dependencies of your application after changing.  The default value is `false` will will apply strict comparison to the resulting floating point numbers used in calculating line segment relationships.


## Tests

```bash
> mix test
```

