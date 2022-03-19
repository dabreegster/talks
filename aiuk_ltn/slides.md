---
title: "A digital twin to design, analyse, and visualise low-traffic neighbourhoods"
subtitle: "AI:UK (draft talk)"
author: "Dustin Carlino"
format:
  revealjs:
    logo: aiuk_logo.png
    progress: false
    menu: false
    theme: custom.scss
---

# Outline

<!-- Easier URL to find slides -->

1. Background of LTNs
2. The tool
3. Technical overview
4. Next steps

# Part 1: Background

## The problem

- Car-centric cities
  - Greenhouse gases
  - Noise pollution
  - Air quality
  - Space for parking
  - Collisions
  - Suburban sprawl and unsustainable land use
  - Lack of exercise

## The problem

:::: {.columns}
::: {.column width="50%"}
![](urban_minor_roads_dft.png)
:::
::: {.column width="50%"}
Rise of traffic on local streets from sat-nav
:::
::::
*<https://assets.publishing.service.gov.uk/government/uploads/system/uploads/attachment_data/file/916749/road-traffic-estimates-in-great-britain-2019.pdf>*

## The response

:::: {.columns}
::: {.column width="50%"}
![](modal_filter.jpg)

*(Jack Fifield on Wikipedia, CC BY 2.0)*
:::
::: {.column width="50%"}
- Modal filters
- Area of effect
- Low traffic neighborhoods
:::
::::

## The response

- 2020 active travel fund
- Mixed public response
  - miscommunication
  - lack of public education
  - hasty consultations
  - genuinely poorly designed LTNs

## The current planning process

![](west_ealing.jpg)

Communication by diagram

## The current planning process

![](deegan_cad.png)

- Live workshops
- *<https://www.youtube.com/watch?v=pHucS2F33W8&t=1052s>*

## The current planning process

- How's this affect my trips?

# Part 2: The LTN tool

## Demo

## The LTN tool

- *<http://ltn.abstreet.org>*
- web browser, Mac, Windows, Linux
  - no mobile
- free, open source
- multiple audiences
  - local authorities / consultants
  - individuals
  - campaign groups

## Scope

- works anywhere, thanks to OpenStreetMap
  - most appropriate for cities

## Credits

- Dustin Carlino: project lead
- Cindy Huang: UX designer (beginning of March)
- Thanks
  - Robin Lovelace: product manager
  - Brian Deegan, Will Petty, Sustrans
  - Martin Lucas-Smith (CycleStreets)
  - Feedback and testing from many!

## Credits

- built on the A/B Street platform
- alumni: Michael Kirk and Yuwen Li

# Part 3: Technical overview

- Neighborhood connectivity
- Predicting rat-runs
- Defining a neighborhood
- Assessing overall impact
- Heuristics for placing filters

## Architecture

![](architecture.png)

## OpenStreetMap into a graph

:::: {.columns}
::: {.column width="50%"}
![](osm_graph.png)
:::
::: {.column width="50%"}
- Edges: road segments
- Nodes: junctions
- one-way streets, lane configuration
:::
::::

## A neighborhood

:::: {.columns}
::: {.column width="50%"}
![](neighborhood.png)
:::
::: {.column width="50%"}
- the perimeter
  - usually "major" roads designed to handle more traffic
- the interior
  - reduce traffic through here
:::
::::

## Cells

![](cells.png)

Everywhere reachable by driving within the neighborhood, without leaving

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

The graph of the neighborhood partitioned into strongly-connected components

## Cells as areas

![](west_ealing.jpg)

Can we match this?

## Voronoi diagrams

![](voronoi.png)

*(Balu Ertl on Wikipedia, CC BY-SA 4.0)*

Not straightforward to apply to line segments

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

Marching Squares to turn the grid into nice contours

## Edge cases with calculating cells

![](one_ways.png)

One-way streets

## Edge cases with calculating cells

![](carless.png)

Roads without motor vehicles

## Edge cases with calculating cells

![](border_cell1.png)

Is this one cell?

## Edge cases with calculating cells

![](border_cell2.png)

Does this path leave the neighborhood?

## Edge cases with calculating cells

![](border_cell3.png)

Decision: separate cells

## Predicting rat-runs

## Predicting rat-runs

![](ratrun_heatmap.png)

- What's a rat-run?
- Why are some streets quiet?

## Predicting rat-runs

![](ratrun_example.png)

## Rat-run definition

- A shortest path starting and ending on the perimeter road
- **Not** the bigger picture
  - Does this path save somebody time?
  - How many people might take this shortcut?

## Rat-run definition

![](ratrun_same_road.png)

- The start and end must be on different roads
- Name changes

## Rat-run definition

![](ratrun_use_perimeter.png)

In freeflow conditions, the perimeter road is faster

## Rat-run definition

![](ratrun_stay_inside.png)

Force the shortest path to stay inside the neighborhood

## Rat-run results

![](detour1.png)

## Rat-run results

![](detour2.png)

## Rat-run results

![](detour3.png)

## Defining a neighborhood

## Defining a neighborhood

![](boundary_default.png)

## Defining a neighborhood

![](boundary_water.png)

- This process will cross railroads, water
  - If it didn't, we're fine with heavy traffic on the nearest local road

## Defining a neighborhood

![](boundary_major_roads.png)

Sometimes major roads just end

## Defining a neighborhood

![](boundary_dual_carriageway.png)

Spaces in between motorway loops or dual carriageways

## Boundary adjustment

- Plenty of reasons the heuristics aren't perfect
- Road classification varies regionally

## Boundary adjustment

Maybe road classification is worth revisiting!

- Different priorities about moving vehicles through an area
- Should that high street also be a through-route?
- Lots of resistance to LTNs is really resistance to the chosen boundary

## Boundary adjustment

![](boundary1.png)

Don't be too prescriptive; let users adjust

## Boundary adjustment

![](boundary2.png)

## Boundary adjustment

![](blocks.png)

- Per block
- Partitioning into contiguous neighborhoods
- Adding a block to one neighborhood removes it from another

## Blockfinding

:::: {.columns}
::: {.column width="50%"}
![](block_order.png)
:::
::: {.column width="50%"}
- Trace around the edge of a road
- Uses the shape of roads and intersections, inferred from OpenStreetMap
- A block internally tracks a list of (road, left/right)
:::
::::

## Merging two blocks

:::: {.columns}
::: {.column width="50%"}
![](merge1.png)
:::
::: {.column width="50%"}
- Find the common slice of the perimeters
- "Rotate" the perimeter until the common part matches up
- Slice and stitch together
:::
::::

## Merging two blocks

![](merge2.png)

## Merging two blocks

![](deadends.png)

- Collapsing dead-ends
- Clockwise and counter-clockwise blocks
  - Winding order

## Blockfinding limitations

![](blocks_water.png)

Only traces roads, not natural features

## Blockfinding limitations

![](blocks_edge.png)

Edge of the study area

## Blockfinding limitations

![](bridges1.png)

- Bridges / tunnels
  - Trace the 2D area of the planar graph?
  - Skip them?

## Blockfinding limitations

![](bridges2.png)

## Blockfinding limitations

![](holes.png)

You can't always draw the boundaries you want

## Assessing overall impact

## Assessing overall impact

![](all_rat_runs.png)

- Show all rat-runs at once
- Rank neighborhoods by number of internal rat-runs

## Assessing overall impact

- Is the rat-run realistic?
- How many people will try to take it?
- With new filters, how will traffic detour overall?
  - Maybe spillover to another neighborhood

## Assessing overall impact

![](demand_model.png)

- Where do driving trips begin and end?
- For neighborhood-scale detours, MSOA zones are too large

## Assessing overall impact

![](impact_prediction.png)

- Calculate all driving routes before and after filters
- Look for quieter and busier streets

## Assessing overall impact

- 2011 census data, commuting for work only
- Urban Observatories traffic counts
    - Calibrate / validate
    - Where do the 5,000 vehicles/hour along a road start and end?
- Bring your own demand model (doesn't need to be public data)

## Heuristics for placing filters

## Heuristics for placing filters

- Human judgment
  - Prioritize near parks, areas with prior problems
  - Can the bin collection truck turn around?
- The use of automation
  - Seed ideas
  - Choice overload

## Where should the filter go?

![](heuristic_before.png)

## Where should the filter go?

![](heuristic_greedy.png)

- Greedy: the road with the most rat-runs
- Whack-a-mole

## Where should the filter go?

:::: {.columns}
::: {.column width="50%"}
![](heuristic_border.png)
:::
::: {.column width="50%"}
- Only one entrance per cell
  - Expensive, likely unpopular
  - Very different results for residents
  - Simpler crossings along the perimeter
:::
::::

## Heuristics for placing filters

![](heuristic_human.png)

Split large cells (intuition)

## Heuristics for placing filters

![](heuristic_split.png)

Minimum cut of the graph

## Heuristics for placing filters

![](heuristic_split_balance.png)

- Trivial solutions near the perimeter
- How large are the two resulting cells?

## Heuristics for placing filters

![](grid.png)

Gridded streets?

## Heuristics for placing filters

![](grid_diagonal.png)

# Part 4: Next steps

- polish, continue testing with local authorities
- share proposals online
- training material (user guide, video tutorials)
- popular feature requests
  - bus gates
  - one-way streets
  - demographics -- who lives near interventions?
  - mode shift / traffic dissipation

## Current usage

- Nottingham
- Campaigning group in Lyon
- Initial testing in ~10 other places

## Engagement model

- <dcarlino@turing.ac.uk>
- I can help import your city
- Do you have travel demand data?
- What else should this tool do?
- I'm looking for help from software engineers & UX designers

## LTNs are just the start

- interventions on perimeter roads
  - safe crossings
  - bus/cycle lanes
  - traffic signal timing

## LTNs are just the start

![](edit_roads.gif)

- other parts of A/B Street can help
  - road space reallocation
  - 15-minute neighborhoods / land use patterns
  - traffic simulation (rapid prototyping)

## Where's the AI?

- Classic computer science graph algorithms vs machine learning
- You can't argue with the results *(barring data quality / bugs)*
  -  The rat-runs show specific shortcuts!
  -  The cells show if it's possible to cut through or not!
- Explainable systems are perceived as more trustworthy

## Engaging communities

- This tool is meant for **everyone**
- People are visual, understand maps
- Build consensus in live workshops

## Conclusion

- *<http://ltn.abstreet.org>*
  - Code at *<https://github.com/a-b-street/abstreet>*
- <dcarlino@turing.ac.uk>
- Contact me to import a city, schedule training, discuss ideas
