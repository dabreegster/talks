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

<!-- picture with waypoints, departure time & mode -->

```json
{
  "scenario_name": "minimal",
  "people": [
    {
      "trips": [
        {
          "departure": 10000,
          "origin": {
            "longitude": -122.303723,
            "latitude": 47.6372834
          },
          "destination": {
            "longitude": -122.31905,
            "latitude": 47.63786
          },
          "mode": "Bike",
          "purpose": "Meal"
        },
        {
          "departure": 12000000,
          "origin": {
            "longitude": -122.31905,
            "latitude": 47.63786
          },
          "destination": {
            "longitude": -122.3075948,
            "latitude": 47.6394773
          },
          "mode": "Walk",
          "purpose": "Recreation"
        }
      ]
    }
  ]
}
```

<https://a-b-street.github.io/docs/tech/dev/formats/scenarios.html>

## What

- More attributes about people
  - vehicle ownership
  - routing preference
  - willing to pay toll road
  - comfortable biking up-hills, at night, alongside fast traffic

## What

- One "typical" weekday?
  - Different scenario for weekends, special events
  - People have different travel behavior daily
  - Dangers of disaggregation

## Use cases

- traffic simulation
- low-traffic neighborhoods: where do people drive, to predict detours
- PCT & Ungap the Map: look for short driving trips that might cycle instead
- RAMP: daily behavior in shared spaces for COVID transmission
  - broken down by activity -- entertainment, work, home, retail
  - lockdown behavior, risk at venues

## Some open source travel demand models

- [Soundcast](https://www.psrc.org/activity-based-travel-model-soundcast)
- [NorMITs](https://github.com/Transport-for-the-North/NorMITs-Demand/)
- [grid2demand](https://github.com/asu-trans-ai-lab/grid2demand/)
- [SUMO](https://sumo.dlr.de/docs/Demand/Introduction_to_demand_modelling_in_SUMO.html)

# Part 3: Let's build one for the UK!

## Input data

![](uk_input_csv.png)

- `wu03ew_v2`: 2011, Location of usual residence and Place of work by Method of travel to work
  - Old, pre-pandemic
  - Only home -> work

## Input data: MSOA zones

![](msoa_zones.png)

## Overall idea

<!-- diagram of overall process -->

- For each (origin, destination, mode) desire line
  - Repeat for the number of trips here
    - Sample an origin and destination from the MSOA
    - Create a person who goes home -> work in AM, work -> home in PM

## Desire lines

```
{
  from_zone: zone123,
  to_zone: zone456,
  mode: walk, bike, drive...,
  number_trips: 500
}
```

- Re-shape the input into this format
  - One mode per entry
  - Filter + simplify the modes
- Why have an intermediate common format?

## Jittering: sampling an origin or destination

![](one_zone.png)

- Just inside the study area
- [odjitter](https://github.com/dabreegster/odjitter)
- Random points?
  - Origins: buildings where people live
  - Destinations: buildings where people work

## Buildings from OpenStreetMap tags

![](osm_bldg.png)

- Building type is rarely tagged

## Buildings from OpenStreetMap tags

![](osm_amenity.png)

- Make sure your library gives you the right data, `building=shop`
- Just look at the data in OSM

## Weighted choices (for origins)

- single-family home vs tower block
  - area of the polygon
  - `building:levels`
- [Colouring London](https://colouringlondon.org)
- [Seattle housing units](https://data-seattlecitygis.opendata.arcgis.com/datasets/parcels-1/explore)
- Use zoning codes?

## Trip attractor tables (for destinations)

![](attractor_table.png)

- cafe vs shopping mall
- [Trip attractor table](https://github.com/asu-trans-ai-lab/grid2demand/blob/main/examples/poi_trip_rate.csv)
  - ITE publishes one

## What data could help us pick better?

- Desire line breakdown by Standard Industry Codes?

## Aside: missing buildings in OSM

![](procgen.png)

- OSM isn't perfect, check your data
- Procedural generation
  - limitations
- Alternate data: OS building footprints?

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
