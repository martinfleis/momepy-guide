# Network analysis

Part of the morphometric analysis is an analysis of street networks. While `momepy` at the moment focuses more on smaller-scale analysis, some of the network-based characters are included. The main difference here is that classes in the `graph` module, allowing this kind of study, are based on `networkx.Graph`, not `GeoDataFrame`.

This section covers:
- [Converting from GeoDataFrame to Graph and back](convert)
- [Network analysis](network)
- [Multiple Centrality Assessment](centrality)