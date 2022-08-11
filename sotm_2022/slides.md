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
- Transition cities away from motor vehicles
- Community-led designs
- Open source, open data, full government transparency
- Traffic simulation, evidence-based transportation modelling, but also just telling stories and marketing

## A/B Street

- Squeeze detail out of OSM -- individual lanes and turns
- That means two modeling choices in OSM cause havoc over and over

## Problem 1: short roads

dog-legs, in between dual carriageways, etc
cases where the tiny road segment is really part of an intersection

## Problem 2: parallel roads

dual carriageways

cyclestreets talk

## Consequences: rendering

not ideal, but usable examples

sometimes totally broken when things physically overlap

## Consequences: traffic simulation

gridlock.png

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

## Inspiration

neukolln

can we do this everywhere?

## One common problem

each consequence / domain can workaround these things in some way maybe

but let's solve all these problems once with a better data model

<!-- ....................................................................... -->

# What osm2streets does

## The demo

just demo the browser thing
import an area live or reimport or something

## Schema

- Roads
  - a bunch of lanes (type, direction, width) derived geometry is thickened linestring
- Intersections
  - Movements between lanes
  - Classifications

- graph
  - turn restriction relations with multiple 'via's
- semantically, lanes
- geometry

## The architecture

- rust: street network crate, import streets crate, output file
- input: osm.xml driving side, config

- streetexplorer JS app
	- leaflet or mapbox or whatever
        - compile rust layer to WASM, overpass fetch, call methods to generate network and then render to geojson layers for... polygons, lane markings. call APIs to get turns.

future ecosystem:
	- call through java and hook into josm
	- integrate parts of it with id editor

## How it works

- parse osm xml, extract bunch of data
- split ways into road segments, now we have a basic graph with raw tags
- osm2lanes (ish) to figure out whats happening for each road segment
- a bunch of transformations to fix various problems
  - collapse degen intersections
  - find and merge short roads into one intersection
  - collapse simple sausage links
  - detect and collapse more general dual carriageways
  - shrink overlapping roads
  - snap separate cyclepaths
- generate polygons for the intersections and lanes, "trim back" the road from the intersection

## How you can use it

- an OSM enthusiast
  - a new renderer for your hard work
  - validation for lane tagging
  - eventually, osm2lanes editor
- GIS enthusiast
  - export the polygons and stuff, bring into qgis and do your own stuff
- a tool author
  - call the library from any language (??) and build cool stuff on top of it
  - contribute upstream to fix the hard problems

## Technical choices

Rust can run anywhere, web or native, bindings easily
extremely performant

no databases, complex deployment
output for a clipped area is just a file

could look into tiling

<!-- ....................................................................... -->

# Example transformations

and these're configurable. for an osm editor directly, you dont want to merge dual carriageways! for rendering or a traffic sim or an analysis of road space or a router, you might want to.

## Simple: dog-leg intersections

## Moderate: simple sausage link

## Hard: separate cycletracks

## Extreme: dual carriageways

<!-- ....................................................................... -->

# Challenges and next steps

## Other features

- trace the block thing
- routing
- isochrones

why some of these and not existing routers and stuff? edits to the network, recalculate efficiently. the abst apps need to do stuff like that.

## streets2osm





- osm2lanes editor
- when there's more detail, use that instead. generate reasonable defaults for most cases, manually map the hard ones.
- how you can help

- the schema itself
  - crosswalks
  - modal filters, bollards
  - lanes arent thickened linestrings; the shape could be more detailed

## Thanks!

- <https://github.com/a-b-street/osm2streets>
- <https://github.com/a-b-street/osm2lanes>
- <dabreegster@gmail.com>
- <https://twitter.com/CarlinoDustin>
