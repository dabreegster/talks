---
title: "osm2streets"
subtitle: "Street networks with detailed geometry"
date: "21 August 2022, State of the Map in Firenze"
author: "Dustin Carlino, Ben Ritter, Michael Droogleever"
format:
  revealjs:
    slide-number: true
---

# Talk outline

- Background
- What osm2streets does
- Example transformations
  - dog-leg intersections
  - dual carriageways
  - parallel cycletracks along a main road
- Challenges and next steps

<!-- ....................................................................... -->

# Background

## My general story

- Quit big tech in 2018
- Software to...
  - Transition cities away from motor vehicles, towards walking, cycling, public transit
  - Engage communities in designing, not just voting/approving
- Open source, open data, full government transparency
- A balance
  - Evidence-based transportation modelling
  - Just telling stories, marketing the idea of more sustainable cities

## A/B Street

![](road_width_osm.png)
![](road_width_abst.png)

- Squeeze detail out of OSM -- individual lanes, turns
- Guess missing data
  - Sidewalks, crosswalks
  - Infer traffic signal timing, number of parking spaces
- Two modelling choices in OSM cause havoc over and over

## Problem 1: short roads

![](dogleg_alley.png)

- OSM represents roads as a center-line
- Some segments of that are the middle of an intersection
- "Dog legs" -- almost a 4-way intersection
- Also between dual carriageways

## Problem 2: parallel roads, separate objects

![](waterloo.png)

- A different way for each side of a dual carriageway, cycletracks with some protection from the road, sidewalks
- Why tag this way?
  - More detail
  - String key=value schema makes tagging one object awkward
  - (Would JSON, nested lists help?)
  - 3 years ago: [Is the OSM data model creaking?](https://2019.stateofthemap.org/sessions/DW7WW8/) by CycleStreets

## Consequences: rendering

![](tempe_junction_osm.png)

- 8 intersections
- 10 road segments that're really part of the intersection

## Consequences: rendering

![](tempe_junction_abst_before.png)

Not ideal, but still usable

## Consequences: rendering

![](phantom_collision.gif)

Sometimes parallel roads with inferred width will physically overlap

more examples...

## Consequences: traffic simulation

![](gridlock.png)

traffic signal timing

## Consequences: editing a road's lanes

streetmix, abst, or -hinthint- osm2lanes

## Consequences: selecting a neighbourhood boundary

not just traffic sim, other tools. 'walk around the block'
ok works great often
but bad geometry breaks it, and dual carriageways get weird

## Consequences: other projects

Cycling Infrastructure Prioritisation Toolkit says Blackfriars need a cycle lane

routing instructions

## Example of all of these

![](tempe_junction_osm.png)
![](tempe_junction_abst.png)

show abst before fixing

- rendering
- inference of pedestrian movement
- traffic signal timing

ideally the east/west is just one object too!

## Inspiration

neukolln

can we do this everywhere?

## One common problem

- Each of these problems might have a workaround for each domain
  - (But it might be very complicated...)
- What if there's a better data model to consume instead of OSM directly?
- What if we can solve all of these problems at once?

<!-- ....................................................................... -->

# What osm2streets does

## The demo

just demo the browser thing
import an area live or reimport or something

## The schema

- Just road segments and intersections
- Roads
  - A center line-string, but thickened width a total width
  - A list of lanes from left-to-right
    - Type (general travel lane, bus lane, cycle lane, parking, sidewalk, grass median, striped buffer with bollards)
    - Width
    - Direction
    - Derived geometry: a thickened line-string
- Intersections
  - Complexity classification: regular crossing, uninterrupted connection (turn lane appears), multi-connection (a dual carriageway splits), terminus (dead-end)
  - Control: uncontrolled, stop signs, signalized
  - A polygon
    - The roads exactly meet this polygon at a right angle
  - Movements through the intersection

## The schema

- Partly a graph (routing)
  - (Not a simple one: turn restriction relations with multiple `via` ways)
- Semantics
  - Is the cycle lane protected from traffic?
- Geometry

## The architecture

1st...

- rust: street network crate, import streets crate, output file
  - within st network: roads and intersections data, route, render to geojson
- input: osm.xml driving side, config

2nd...

- streetexplorer JS app
	- leaflet or mapbox or whatever
        - compile rust layer to WASM, overpass fetch, call methods to generate network and then render to geojson layers for... polygons, lane markings. call APIs to get turns.

future ecosystem:
	- call through java and hook into josm
	- integrate parts of it with id editor

## How it works

1.  Parse OSM XML, extract raw data
2.  Split ways into road segments (giving a graph with tags on edges)
3.  Lanes per road segment
  - Still using a big gross function (with unit tests)
  - Proper rewrite: [osm2lanes](https://github.com/a-b-street/osm2lanes)
  - Sidewalks are often not tagged, make some configurable guesses
4.  Transformations to fix various problems
  - collapse "degenerate" intersections between 2 roads
  - find and merge "short" roads into one intersection
  - collapse simple "sausage link" patterns
  - detect and collapse more general dual carriageways
  - if all else fails, shrink physically overlapping roads
  - merge parallel cyclepaths with the main road
5.  Generate polygons for the intersections and roads
  - "Trim back" the road center-line from the intersection

## How you can use it

- an OSM enthusiast
  - a new renderer for your hard work
  - validation for lane tagging
  - WIP: osm2lanes editor, don't learn complex lane tagging

## How you can use it

TODO: show qgis example

- GIS
  - Export the polygons, use in your QGIS projects

## How you can use it

- an OSM tool author
  - Call the library from any language (?) and build anything on top of it!
  - Contribute upstream to fix the hard problems

- The API
  - Import this osm.xml file (or grab from Overpass)
  - Run these transformations to simplify the network
  - Get lane detail (define your own routing score function for walking safety/comfort)
  - Render geometry to GeoJSON (polygon areas, lane markings)
  - More later: routing, isochrones, tracing areas between roads

- Integration
  - Working: Rust, JS / browser
  - Planned: Java, Python, R
  - Worst case: call a command-line tool

## Technical choices

- Rust: extremely performant, language makes it hard to introduce some bugs, native or web
- Output is just a single file (or in-memory)
  - No databases
  - No complex deployment, just static file hosting
- As large as "city scale"
  - We could look into tiling

<!-- ....................................................................... -->

# Example transformations

- Configurable
  - an OSM editor: don't merge intersections or parallel roads
  - routing, rendering, transportation analysis: opt into everything

- General strategy
  - Pattern match on some situation (looking at both graph and geometry)
  - Label what's matched for debugging
  - Resolution:
    - Graph (remove edges, reconnect something)
    - Geometry (straighten the line between dual carriageways)
    - Lanes (append the lanes from one road onto another, inserting a barrier lane)
  - Interactive development / debugging
    - Use StreetExplorer and step through (visual print debugging)
  - Testing
    - Clip representative OSM examples
    - Catch regressions

## Simple transformation: dog-leg intersections

## Moderate transformation: simple sausage link

## Hard transformation: separate cycletracks

## Extreme transformation: dual carriageways

<!-- ....................................................................... -->

# Challenges and next steps

## Other features

- Tracing around the block
  - requires edge of the road and intersection polygons
- Routing
  - Score functions for cycling safety/comfort can use detailed lane information
  - How many traffic signals does a route go through?
- Isochrones / 15-minute walk-sheds

## Other features

- Why not Valhalla, GraphHopper, OSRM?
- A/B Street needs this, maybe others do too
  - Need to deploy the server, can't run in the browser offline
  - Edit the street network and efficiently calculate consequences
- Extra layers (or separate projects) built on top of osm2streets

## osm2lanes editor

- show demo
- help needed with both osm2lanes and JS/web app/design
- prototype as a separate app, but ideally integrate with id and josm

## Promote area:highway tagging

- We'll never algorithmically figure out all the cases
- Make it easier to map the curb or more detail for hard cases
- osm2streets can generate reasonable defaults for most cases
- How can osm2streets use existing tagged areas to override?

## The schema itself

- Lanes aren't thickened linestrings -- the shape could be more detailed
- Crosswalks
- Modal filters / bollards
- Advanced stop lines, bike boxes
- Pedestrian crossing islands

## How you can help

- Programming, integrating the API in your tools, working on the schema, design
- Bug reports (for osm2lanes -- defining the correct result)

## Thanks!

- <https://github.com/a-b-street/osm2streets>
- <https://github.com/a-b-street/osm2lanes>
- <dabreegster@gmail.com>
- <https://twitter.com/CarlinoDustin>
- Special thanks to Ben Ritter, Michael Droogleever, Tobias Jordans
