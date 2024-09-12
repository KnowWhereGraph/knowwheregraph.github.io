# S2 Integration

KnowWhereGraph uses the [S2 system](http://s2geometry.io/) for a global grid structure. This allows for a structure where [RCC](https://en.wikipedia.org/wiki/Region_connection_calculus) relations can be used to locate things within, between, overlapping, etc cells. The process of forming these relation with custom geometries is what we refer to as *integration*.

## Integrating Your Data

We provide a [tool](https://github.com/KnowWhereGraph/s2-coverings) for both generating s2 cell geometries and integrating custom geometries with them. It's generally un-necessary to generate your own S2 cells because they've been generated and exist within KnowWhereGraph - and the relations can be formed in relation to those.
