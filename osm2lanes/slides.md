---
title: "osm2lanes"
subtitle: "12 March 2022, FOSSGIS OSM-Samstag"
author: "Dustin Carlino & Michael Droogleever"
format:
  revealjs:
    slide-number: true
    logo: logo.svg
---

# Intro

- These slides: <https://dabreegster.github.io/talks/osm2lanes/slides.html>
- Contribute: <https://github.com/dabreegster/talks>

## About Dustin

- <https://twitter.com/CarlinoDustin>
- A/B Street since 2018
- Alan Turing Institute since December 2021

## About Michael

- <https://github.com/droogmic/>
- <https://www.openstreetmap.org/user/droogmic>
- Software Engineer, ASML, Netherlands since 2018

## Talk Outline

1.  Background
2.  How it works today
3.  Complications
4.  Next steps / contributing

# Part 1: Background

## A/B Street

:::: {.columns}
::: {.column width="40%"}
![](osm_detail.png)
:::
::: {.column width="60%"}
- a bunch of tools to explore less cars in cities
- all work off a heavily processed map representation
    - road and intersection geometry
    - driveways between buildings and roads, parking lot capacity
    - turn restrictions, traffic signal timing, routing
:::
::::

## Edit Roads

![](https://a-b-street.github.io/docs/project/history/retrospective/edit_roads.gif)

## Simulate Traffic

![](https://a-b-street.github.io/docs/project/history/retrospective/traffic_sim.gif)

## Plan Bike Networks

[bike.abstreet.org](http://bike.abstreet.org)

![](https://a-b-street.github.io/docs/software/ungap_the_map/demo.gif)

## Low-traffic Neighborhoods

[ltn.abstreet.org](http://ltn.abstreet.org)

![](https://a-b-street.github.io/docs/software/ltn/ltn.gif)

## Sharing Code

- <https://github.com/a-b-street/abstreet/discussions/789>
- <https://github.com/a-b-street/abstreet/blob/master/raw_map/src/lane_specs.rs>
- Other projects looking at OSM lanes in detail
  - StreetComplete, Map Machine, Bjorn's JOSM plugin, shared-row, Cycle Streets, 3D Street

## Why's this hard?

![](cyclestreets.png)

- [Cyclestreets: Is the OSM data model creaking?](https://www.cyclestreets.org/news/2019/09/22/sotm2019/)

## Why does it have to be hard?

From left to right:

```
[
  { type=sidewalk, surface_color=black },
  { curb },
  { type=cycle, direction=forward, surface_color=red, est_width=1.5 },
  { curb },
  { type=travel, surface_color=black, direction=forward, allow taxi },
  { type=travel, surface_color=black, direction=backward, allow taxi },
  { curb },
  { type=cycle, direction=forward, surface_color=red, est_width=1.5 },
  { curb },
  { type=sidewalk, surface_color=black },
]
```

- Arguments for mapping as separate ways
  - geometry: doesn't help
  - per-lane detail: yes!

## End-user Stories

- vector map of whole world showing lane tagging
- Streetmix style editor for lane tagging
  - <https://github.com/openstreetmap/iD/issues/387>
- both of these help improve lane tagging
- road space studies
  - 60% of width used by 20% of people

# Part 2: How it works today

Web demo: <https://a-b-street.github.io/osm2lanes>

## Input

- An OSM way's tags
- A locale (country / region code), for inferring if not explicitly defined, for example
  - tag meaning
  - left- or right-handed driving
  - lane widths
  - lane separator styles

## Output

<https://www.openstreetmap.org/way/22760280>

```
{
  "Ok": {
    "road": {
      "lanes": [
        {
          "type": "travel",
          "direction": "backward",
          "designated": "bicycle",
          "width": 2.0
        },
        {
          "type": "separator",
          "markings": [
            {
              "style": "solid_line",
              "width": 0.2,
              "color": "white"
            }
          ]
        },
        {
          "type": "travel",
          "direction": "backward",
          "designated": "motor_vehicle",
          "width": 3.5
        },
        {
          "type": "separator",
          "markings": [
            {
              "style": "dotted_line",
              "width": 0.2,
              "color": "white"
            }
          ]
        },
        {
          "type": "travel",
          "direction": "forward",
          "designated": "motor_vehicle",
          "width": 3.5
        },
        {
          "type": "separator",
          "markings": [
            {
              "style": "solid_line",
              "width": 0.2,
              "color": "white"
            }
          ]
        },
        {
          "type": "travel",
          "direction": "forward",
          "designated": "bicycle",
          "width": 2.0
        }
      ],
      "highway": {
        "highway": {
          "Classified": "Secondary"
        },
        "lifecycle": "Active"
      }
    },
    "warnings": [
      "unimplemented: access, bicycle=designated"
    ]
  }
}
```

## Output, `osm2lanes`

- type
  - travel, parking, shoulder, separator
- designated
  - foot, bike, motor vehicle, bus
  - (in the future we will add `access=*` per lane)
- direction
  - forward, backward, both
- width
- markings / separators

## Inverse, `lanes2osm`

- An easier OSM lanes editor
  - Pick a way, grab its tags
  - `osm2lanes`
  - Edit the lanes with something Streetmix-like
  - `lanes2osm`
  - Upload the diff
- Maybe a tag "autoformatter" in iD/JOSM?
- Complications?

## Code overview

- <https://github.com/a-b-street/osm2lanes>
- Rust, Python, and Kotlin
- Originally...
  - make it easy for people to get involved, no matter the preferred language
  - "not hard" to keep the implementations in-sync
- Going forward...
  - Rust can target any build environment

## Tests!

<https://github.com/a-b-street/osm2lanes/blob/main/data/tests.yml>

## Code Walkthrough

- [Lane schema](https://github.com/a-b-street/osm2lanes/blob/main/rust/osm2lanes/src/road/lane.rs)
- [Nested switch case logic by transport mode](https://github.com/a-b-street/osm2lanes/blob/03afea4f6272568e2b91e6e575945e58e2654aa7/rust/osm2lanes/src/transform/tags_to_lanes/mod.rs#L382) \
  special case non-motorized paths, bus, bike, parking, sidewalks / shoulders
- forward and backward lanes handled separately, lanes usually appended inside-to-out
- separators inserted last based on the two adjacent lane types

# Part 3: Complications

## Errors are rampant

- <https://www.openstreetmap.org/way/389322095>

```
unsupported: cycleway=* with any cycleway:* values


bicycle = designated
cycleway = lane
cycleway:left = separate
cycleway:right = lane
highway = secondary
lanes = 4
lanes:backward = 2
lanes:forward = 2
lcn = yes
maxheight = default
maxspeed = 25 mph
name = Dexter Avenue North
oneway = no
surface = paved
```

## Error and Warning Handling

- issues encountered in tags classified as
  - ambiguity
  - unimplemented
  - unsupported
  - deprecated
- error is thrown if sufficiently problematic
- otherwise a warning is added, and we try and continue

## Inferred Values

not all data is right or wrong, usually it is missing

- Direct tagged data
- Calculated (total width must be the sum of the other widths)
- Inferred (splitting total width over all the lanes evenly)
- Default based on locale
- Reasonable default

the data consumer wants to know what lane data was tagged and what was a guess?

## Designated vs Allowed

- footpaths allowing, but not prioritizing, cyclists
- bus lanes allowing cyclists

this is a generic library

- most rendering applications simply wants to know what the designated purpose is
- a routing application wants to know what the full access is

## Separate Ways

<https://www.youtube.com/watch?v=LatorN4P9aA>

- Punt to the caller, feed in all of the ways?
  - next talk will go into more detail
  - just look for everything in between buildings/parks/water?
- also provide geometry, so we can glue things together in the right order?
- inferred separators
  - or <https://wiki.openstreetmap.org/wiki/Talk:Proposed_features/cycleway:separation>

# Part 4: Next steps / contributing

<https://github.com/a-b-street/osm2lanes>

## Test cases

we need a healthy combination of:
- real world examples (with mapillary / pictures to know what exists on the ground)
- esoteric examples, when real world examples cannot be found to test the extremes

## `osm2lanes`

Most implementation is in rust, \
but rust is not too hard to learn.

Try one of the [good first issues](https://github.com/a-b-street/osm2lanes/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22)!

### Web Interface

Web interface: https://a-b-street.github.io/osm2lanes/

- written almost entirely in rust
- allows for rapid improvement of implementation

# `osm2lanes` Future

## Per-lane Width

- Use it when it's tagged
- If we know the curb-to-curb width or entire road width...
  - sanely distribute width to known lanes

## Per-lane Data

- turn:lanes
- allowed vehicles (bus lanes with bikes or taxis)
- time-restricted turns or parking
- surface type
- speed limit

## Locales

- <https://github.com/streetcomplete/StreetComplete/tree/master/res/country_metadata>

## Library

This library needs to be used to be useful.

If you know a project that can use this functionality, help contribute there to make it a dependency.

Use it on your own projects!

## Web map

<https://github.com/a-b-street/osm2lanes/blob/main/web/index.html>

- Click a road, see its lanes in cross-section view
- Rust <-> Javascript API
- Publish on npm