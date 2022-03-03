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

## Talk outline

1.  Background
2.  How it works today
3.  Complications
4.  Next steps / contributing

# Part 1: Background

## A/B Street

- a bunch of tools to explore less cars in cities
- all work off a heavily processed map representation

## Edit roads

![](https://a-b-street.github.io/docs/project/history/retrospective/edit_roads.gif)

## Simulate traffic

![](https://a-b-street.github.io/docs/project/history/retrospective/traffic_sim.gif)

## Plan bike networks

[bike.abstreet.org](http://bike.abstreet.org)

![](https://a-b-street.github.io/docs/software/ungap_the_map/demo.gif)

## Low-traffic neighborhoods

[ltn.abstreet.org](http://ltn.abstreet.org)

![](https://a-b-street.github.io/docs/software/ltn/ltn.png)

## Sharing code

- <https://github.com/a-b-street/abstreet/discussions/789>
- Other projects looking at OSM lanes in detail
  - StreetComplete, Map Machine, Bjorn's JOSM plugin, shared-row, Cycle Streets, 3D Street

## Why's this hard?

Cyclestreets example

## End-user stories

- vector map of whole world showing lane tagging
- streetmix style editor for lane tagging
  - <https://github.com/openstreetmap/iD/issues/387>
- both of these help improve lane tagging

# Part 2: How it works today

Start with web demo: https://a-b-street.github.io/osm2lanes/

## Input

- localization

## Output

- lane type, width, direction
- turn:lanes, allowed vehicles (bus lanes with bikes or taxis), time restrictions, surface type
- markings / separators are lanes!

## Inverse, lanes2osm

## Web tool

## From code

- multi language or not? we've diverged
  - python 3.9 and kotlin too
- originally trying to make it easy for other people to get involved

## Code walkthrough

## Inferred values

# Part 3: Complications

## Separate ways

- the caller has to figure out all of the related ways?
  - next talk will go into more detail
  - just look for everything btwn buildings/parks/water
- also provide geometry, so we can glue things together in the right order?
- inferred separators

## schema may be kinda complicated?

# Part 4: Next steps / contributing

## Web tool steps

- npm

## Test cases

## Per-lane width

and if we can get total road/RoW width from another method, sanely distributing to lanes
