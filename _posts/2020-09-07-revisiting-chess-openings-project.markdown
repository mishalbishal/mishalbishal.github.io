---
layout: post
title:  "Chess openings visualization using d3.js"
date:   2020-09-07 11:12:00 -0400
categories: projects
---

The [Chess Openings Visualization](https://mishalbishal.github.io/cs171-project2/)
is a group project that I made with Keoni Correa for a visualization course back in college. We were both interested in chess and saw this as an opportunity to learn the d3 visualization library while exploring a shared interest.

If you haven't used d3 before, you can [peruse the github repo](https://d3js.org). I've put a short summary here as well: To create visualizations in the browser you need to manage some way of having your data create and interact with html elements. Other charts libraries allow you to input the data in their proscribed format and either hit an API to create an image or use Javascript to create the charts for you. D3 is a lower level library in that it does not abstract as much of the visualization creation process away from the user. On the bright side, this means that there is a greater deal of flexibility in what to create.

The d3 library acts as a glue so that you don't have to manually manage the connections between your data (in whatever format you choose, unlike inflexible visualization libraries) and the rendered html elements. There are numerous tutorials for using d3 and they are very comprehensive. The best starting point is probably to visit [d3's github repo](https://d3js.org/) and read through the introductory examples and try tweaking them to your own use case.

d3 is a large library and it can be overwhelming to start using it. The good thing is that there are numerous examples available to peruse and learn from at the [https://bl.ocks.org/](https://bl.ocks.org/) website and the [d3 gallery at observablehq.com](https://observablehq.com/@d3/gallery).