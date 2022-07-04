---
title: "osm2streets"
subtitle: "Generating OpenStreetMap street networks with better geometry"
date: "7 July 2022, ASG Birmingham"
author: "Dustin Carlino"
format:
  revealjs:
    slide-number: true
---

# Talk outline

- OpenStreetMap primer
- Problems
- Possible solutions
- Next steps and API design

<!-- OpenStreetMap is a widely used data source for street networks worldwide. Due to its data model, representing complex streets and junctions (often involving dual carriageways and parallel cycle tracks and footpaths) is challenging and inconsistent. As a result, many applications struggle to render, provide simple routing directions, reason about road space allocation to different modes of travel, and apply urban morphology analyses. osm2streets is an early-stage effort to transform the raw data into a simpler representation of streets and help many of these applications. This talk will motivate the project and discuss early results. -->

# OpenStreetMap primer

![](osm_view_birmingham.png)

<https://www.openstreetmap.org/#map=18/52.45115/-1.93205>

## OSM primer

![](osm_edit_birmingham.png)

## OSM representation of streets

![](osm_way.png)

- **Node**: Points with key/value strings
- **Way**: Line-string with key/value strings

## OSM representation of streets

![](tags_to_lane_specs.png)

- The way's attributes describe the lanes, direction, speed limit, access restrictions...
- The way has to be split when any of these change

## Footpaths and cycle paths: as attribute

![](attribute_of_way.png)

## Footpaths and cycle paths: as attribute

![](cyclestreets.png)

*From [CycleStreets SoTM talk](https://www.cyclestreets.org/news/2019/09/22/sotm2019/)*

- People don't like tagging this way

## Footpaths and cycle paths: as separate ways

![](blackfriars_separate.png)

- Also for dual carriageways

# Problems with this representation

## Rendering

![](rainier_osm.png)

- Ideally we can do this
- Note intersection polygons are guessed from road width

## Rendering

![](bristol_abst.png)

- Different OSM roads overlap each other

## Rendering

![](bristol_osm.png)

## Rendering

![](bristol_satellite.png)

## Rendering

![](neukolln_map.jpg)

- The state of the art by Berlin OSM community
- <https://strassenraumkarte.osm-berlin.org/?map=micromap#20/52.49555/13.42073>
- Pocket parking, curb bulbs, complex junctions

## How is road space allocated today?

- COVID pavement widths ([Madrid project](https://distanciamiento.inspide.com/))
- Street parking capacity vs vehicle ownership (Neukoelln blog post)

## Editing a road by lanes

![](editing.png)

*A/B Street (top) & Streetmix (bottom)*

- For civic engagement, or just to correct OSM data more easily

## Statistics per road

![](cyipt.png)

- Cycling Infrastructure Prioritisation Toolkit says Blackfriars need a cycle lane...
- Spatial inequality of streets without sidewalks
- How many collisions along a divided highway?

## Routing instructions

![](uturn.png)

- Every SatNav ever: "Use the right lane to bear slightly left, then take a sharp right"
- A human: "Turn right at the intersection"
- Cost function for how many traffic signals crossed

## Blockfinding

![](blockfinding_good.png)

- Traces the inside of a "city block", exactly following road outlines

## Blockfinding

![](blockfinding_bad.png)

## Traffic simulation

![](gridlock.png)

- Rules about not blocking junctions

## Traffic signal timing

![](traffic_signals.png)

- Editing UI and automatic heuristics

# Possible solutions

## osm2streets

<https://github.com/a-b-street/osm2streets>

- Split out from A/B Street
  - <https://github.com/a-b-street/osm2lanes>
  - [Intersection geometry](https://a-b-street.github.io/docs/tech/map/geometry/index.html)
- Effort driven by 2 open source contributors

## osm2streets

![](raw_map_editor.png)

- A directed graph with geometry
- Start from OSM, then apply transformations to it
  - Collapse degenerate intersections when nothing important differs
  - Find and merge short roads
  - Shrink parallel overlapping roads (stop-gap)
  - Snap parallel cycleways to road (broken)
- Ben's ideas for the final structure

## osm2streets

![](test_cases.png)

- Browse test cases: <https://a-b-street.github.io/osm2streets/>
- OSM input, GeoJSON output: <https://github.com/a-b-street/osm2streets/tree/main/tests/src/bristol_sausage_links>

## osm2streets: sausage links


example sausage links

dual carriageways in general... preserving the graph structure of turns

## OSM representations

future idea: linear referencing + changing widths

	split the road when a turn lane appears or not?

	in OSM, draw overlapping ways to indicate intervals?

## momepy's GSOC

graph and geometric approach

## Find width between buildings/areas

- isnt it simple? just perpendicular line and find stuff? maybe?

## Manually draw polygons in OSM as additional things for grouping

- only for the hard cases
- with a web app making this easy, why not?

# Desired output

- something callable from python, JS, etc
- osmx kind of library, work with OSM street networks at a higher level

- routing, rendering, semantic description of roads

- blockfinding
- 15m isochrone
- what do you need it for?

- sometimes 


## We want a representation that...

1. shows geometry pretty accurately
2. lets you understand hierarchy -- entire junction, just the north approaching road and its crossing islands, individual lanes, stopping line
3. represents routing / turn restrictions
4. represents semantics that apply over space (loading zones, no parking overnight)





# Other next steps

- drag n drop OSM lane editor
	- split roads when lanes differ
	- show topdown view

- just get current 2D rendering, but dynamically from overpass

# Questions for audience

schema for street space? cityGML?
