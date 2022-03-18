---
title: "A digital twin to design, analyse, and visualise low-traffic neighbourhoods"
subtitle: "AI:UK spotlight talk"
author: "Dustin Carlino"
format:
  revealjs:
    slide-number: true
    logo: logo.svg
---

# Outline

These slides: ...

- Background of LTNs
- The tool
- Technical overview
- Next steps

# Part 1: Background

In response to rising levels of motor vehicles using sat-nav to cut through residential streets and avoid traffic, local authorities across the UK have been creating low traffic neighborhoods (LTNs). Modal filters (planters or bollards in the middle of the street) prevent drivers from passing through, but allow pedestrians and cyclists. When filters are strategically placed to prevent all through-traffic, people in the area are likely to enjoy better air quality, less noise pollution, and higher levels of physical activity.

The 2020 active travel fund jump-started many new LTN schemes, but there has been a mixed public response, stemming partly from miscommunication, lack of education, and hastened consultations. To design and share LTN schemes, planners at local authorities currently use manual workflows in existing GIS software -- or sometimes just sketching ideas over satellite imagery. The resulting schemes rely on humans to assess all possible routes for through-traffic; sharing the plans with the public is usually done with a low-resolution diagram; and members of the public have no easy way to propose an alternate design.

<!-- sustrans references -->

# Part 2: The tool

- <http://ltn.abstreet.org>
- web browser, Mac, Windows, Linux (no mobile)
- free, open source
  - <https://github.com/a-b-street/abstreet/tree/master/apps/ltn>
- targets local authority planners, individual residents, and campaign groups

## Credits

- developed in a few months
- built on top of the A/B Street platform
  - alumni: Michael Kirk, Yuwen Li

## Scope

- works anywhere, thanks to OpenStreetMap
- most appropriate for cities
- tuned for the UK

## Demo

# Part 3: Technical overview

![](architecture.png)

Join the workshop tomorrow at 15:00 for details

## OpenStreetMap into a graph

<!-- img -->

- Edges: road segments
- Nodes: junctions
- one-way streets, lane configuration

## A neighborhood

![](neighborhood.png)

- the perimeter
  - usually "major" roads designed to handle more traffic
- the interior
  - reduce traffic through here

## Cells

![](cells.png)

- Everywhere reachable by driving within the neighborhood, without leaving

## Cells

![](adjacent_cells.png)

## Disconnected cells

![](disconnected.png)

- A cell must touch the perimeter somewhere!
  - Otherwise, drivers can't get in / out

## Calculating cells

- Pretty straightforward graph floodfill
- Start from an unvisited road segment in the interior
  - expand
  - don't cross modal filters, non-driveable roads, or the perimeter

## Floodfill

![](floodfill1.png)

## Floodfill

![](floodfill2.png)

## Floodfill

![](floodfill3.png)

## Floodfill

![](floodfill4.png)

## The result

![](scc.png)

- The graph of the neighborhood partitioned into strongly-connected components

## Cells as areas

![](west_ealing.jpg)

- Can we match this?

## Voronoi diagrams

![](voronoi.png)
(Balu Ertl on Wikipedia, CC BY-SA 4.0)

- Not straightforward to apply to line segments

## Approximating with grids

- Diffusion on a 10 meter grid
- Stop when there's a collision
  - Record the adjacency for the 4-color theorem

## Approximating with grids

![](grid1.png)

## Approximating with grids

![](grid2.png)

## Approximating with grids

![](grid3.png)

## Approximating with grids

![](marching_squares.png)

- Marching Squares to turn the grid into nice contours

## Edge cases with calculating cells

- One-way streets

![](one_ways.png)

## Edge cases with calculating cells

![](carless.png)

- Roads without motor vehicles

## Edge cases with calculating cells

![](border_cell1.png)

- Is this one cell?

## Edge cases with calculating cells

- Does this path leave the neighborhood?

![](border_cell2.png)

## Edge cases with calculating cells

- Decision: separate cells

![](border_cell3.png)

## Predicting rat-runs

<!-- heatmap. why some streets quiet or not? -->

- What makes a street quiet or not?
- No reason to drive to a cul-de-sac / deadend
- For the red heatmap of detours and to point out problems, need to find shortcuts THROUGH the neighborhood

## Predicting rat-runs

- Every way a driver might cut through a neighborhood "reasonably"?
- Every possible path between two borders of a neighborhood?
- Important: unit is just this neighborhood. We don't know why somebody would try to go from A to B, or how many might -- we need to see the bigger picture for that. Later.

## Predicting rat-runs

- The path shouldn't go OUTSIDE the neighborhood / use the perimeter
- If it did, it might not represent a real problem
- So we can be smarter, and just do the analysis by cell
- And we HAVE to force the pathfinding to stay in the cell

## Predicting rat-runs

- This still leave some silly examples. Somebody wouldn't do this unless you're in Houston highway exit hopping
- So make sure the entry/exit is on a "different" road
- Defining this is a little hard; use road name in OSM. Not perfect but

## Initial boundaries

- We've been talking in units of a neighborhood, but how're these defined?
- Take a "major" road from OSM and trace a complete loop
  - Sometimes a major road just ends; Stovell Ave / Matthews Lane in levenshulme
- This process will cross railroads, water
  - If it didn't, the local road on that part of the perimeter would be OK to have traffic

## Boundary adjustment

- But the heuristics are just a start! Plenty of reasons it might not make sense:
  - OSM classification isnt quite right
  - Small spaces in between a dual carriageway count!
  - Legitimately you want to join two neighborhoods into one piece

## Boundary adjustment

And the most important: get people to revisit road classification!

- Road classification was done a while ago with different priorities about moving vehicles through an area
- Maybe it's time to rethink that high street as a thoroughfare

## Boundary adjustment

- Regardless of the reason, I don't want this tool to be prescriptive about boundaries! Let humans adjust:

<!-- show it -->

## Boundary adjustment

- Operates block by block
- All blocks partitioned into contiguous neigborhoods
- If you add a block to one neighborhood, you remove it from the other; the boundary shifts

## Limitations with boundary adjustment

- Bridges, tunnels, edge of map
- Tiny blocks
- You can't create holes
- It's not perfect; you can't always draw the boundaries you want

## Assessing overall impact

- some simple summaries
  - rank neighborhoods by how many internal rat runs they have
  - show rat runs / quiet streets everywhere

## Assessing overall impact

- Revisit the rat-run idea. How will traffic really detour in the short-term?
- Need a travel demand model; where do driving trips begin and end?
- If we know, calculate route before and after filters, show roads with less/more traffic

## Assessing overall impact

- At the moment, only have something based on 2011 data and commute trips
- Working with Urban Observatories to get better data

## Heuristics for placing filters

- Always need human judgment -- prioritize calming near a park, or a load zone, or based on road width
- But getting some automation help is always nice, especially when there's overwhelming number of choices
- So for a neighborhood, what're filters that could be placed automatically?

## Heuristics for placing filters

- Focus on road with most rat-runs, play whack-a-mole

## Heuristics for placing filters

- Brute force: only one entrance/exit to every cell

## Heuristics for placing filters

- Split large cells into smaller pieces; this'll probably also have the effect of reducing rat runs
- Consider a filter on every road segment. Does it split the cell?
- Many solutions just cut off near the start of a neighborhood. For the resulting two cells, take the smaller number of roads (aka size). Maximize that
- Show some surprising results / solutions. I didn't see some of these.

## A large-scale vision

- Show a very large-scale change

# Part 4: Next steps

- polish what's there
- sharing proposals online
- further engagement with LAs, prioritize accordingly
- new stuff
  - bus gates, one-way interventions
  - mode shift / traffic dissipation
  - census

## LTNs are just the start

- interventions on boundary roads
  - crossings
  - bus/cycle lanes
  - traffic signal timing
- other parts of A/B Street can help
- go simulate the interventions

## Why this approach can engage community usefully

- It's very visual; people are used to reasoning about maps. Be familiar.
- You can't argue with the results
  -  The rat runs show the path somebody could take through!
  -  The cells show if it's possible to drive through or not!
  -  (Barring data quality / bugs)
- There's no magic / AI. It's explainable, and thus trustworthy

## Conclusion

- <http://ltn.abstreet.org>
- <dcarlino@turing.ac.uk>
- <https://github.com/a-b-street/abstreet>
- Contact me to import a city, to schedule training, to discuss ideas
