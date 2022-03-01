---
title: "From OD data to agent based modelling for car free futures"
author: "Dustin Carlino"
format: revealjs
---

# Intro

- revealjs: can all of this go on the title screen?
- date / venue
- abst logo
- links

## About me

- A/B Street since 2018
- Alan Turing Institute since December 2021
- Not an expert on TDM; I just need it as input

## Talk outline

1.  A/B Street from 10,000 feet
2.  Travel demand models overview
3.  From UK OD data to a demand model
4.  Bonus: Activity models
5.  Bonus: Mode shift
6.  Exercises / discussion

# Part 1: A/B Street overview

- architecture at a glance
- OSM to map: osm2lanes, osm2polygons, turns, etc
- a traffic sim
- a bunch of apps built on top of commmon platform

# Part 2: Demand model overview

## What

- https://a-b-street.github.io/docs/tech/dev/formats/scenarios.html
- disaggregation is strange: one "typical" day repeated?
- also attributes about people: what vehicles they own, comfortable biking up hills or alongside traffic

## Use cases

- traffic simulation
- LTN tool: just need to know where people drive, to A/B test routes
- PCT & Ungap the Map: look for short driving trips
- RAMP: needs to know purpose of trips / activity type on the other end, to model lockdown behavior and risk

## Other projects

- normits, soundcast
- grid2demand
- https://sumo.dlr.de/docs/Demand/Introduction_to_demand_modelling_in_SUMO.html

# Part 3: Let's build one for the UK!

- input is just home->work
- diagram of overall process

- Go through each desire line, sample an origin and destination from the appropriate zone. Sample from the normal time distributions

## Desire lines

```
{
  from_zone: zone123,
  to_zone: zone456,
    mode: walk, bike, drive...,
    number_trips: 500
}
```

along with a polygon per named zone, and a study area to clip to.

## Desires lines for the UK

- wu03ew_v2 and MSOA zones. https://github.com/a-b-street/abstreet/blob/master/importer/src/uk.rs

- re-shape into a different input format (one mode per row)

- why have an intermediate common format?

- why one mode per row?

## Sampling an origin or destination / jittering

- The simple case first: inside the map
- odjitter

- O: homes
- D: places where people work

## Detecting these from OSM tags

- most just `building=yes`
- actually go look at the data in OSM
- amenity nodes inside polygons

## Weighted subpoints (for origins)

- SFH vs a tower block
- some OSM tags that could help
- but frequency of those tags is low

## Trip attractor tables (for destinations)

- cafe vs shopping mall

## What data could help us pick better?

- what if we had SIC percentages for desire lines?

## Aside: missing buildings in OSM

- show the problem (and emphasize looking at the data!)
- procgen as a workaround
- tradeoffs
- OS building footprints?

## Study area size

- show zones partially intersecting study area
- we cant just keep expanding our study area forever
- use area overlap as a percentage?

## Jittering when out-of-bounds

https://github.com/a-b-street/abstreet/pull/853

when getting it wrong, we might send traffic on tiny service roads or residential streets!

## Back out of the details

- overall flow again
- just review all the edge cases we covered

## Enjoying the results

- watch the simulation in abstreet, zoom in on examples of gridlock, discuss the dangers of blindly trusting a microsimulation
- Routing

# Part 4: Bonus: Activity models

http://play.abstreet.org/papers/synthpop/strawmen.html#activity-model

# Part 5: Bonus: Mode shift

https://a-b-street.github.io/docs/software/ungap_the_map/tech_details.html#predict-impact

# Part 6: Exercises / discussion

- install
- import a new place
- procgen buildings, use qgis or geojson.io to look
- use R to transform UK or other data into desire line format?
- or: abstr for scenario.json files

- discussion: how to calibrate models to real traffic counts?
