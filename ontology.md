# Ontology

*KnowWhereGraph has a large ontology*. It's no secret that navigating *any* ontology is a time consuming, difficult process. KnowWhereGraph is no exception.

We provide a [Widoco](https://github.com/dgarijo/Widoco) documentation page for our ontology which includes diagrams to help compartmentalize, visualize, and digest the various components.

[KnowWhereGraph Widoco Ontology Docs](https://stko-kwg.geog.ucsb.edu/lod/ontology)

## Key Points

There are several key points that make working with and understanding the ontology easier.

### Understanding Time

KnowWhereGraph uses [OWL time](https://www.w3.org/TR/owl-time/) for temporal information. We don't use the *entire* ontology, just a few pieces that are summarized below.

#### Time Instants

The most basic node that represents time are nodes of type `time:Instant`. These nodes typically have four properties:

1. A label with the full datetime
2. An XSD:Date representation
3. An XSD:DateTime representation
4. The year

An example is shown below where the full datetime and year are retrieved from a `time:Instant`.

```SPARQL
PREFIX geo: <http://www.opengis.net/ont/geosparql#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX time: <http://www.w3.org/2006/time#>
select ?time_label ?datetime ?year where { 
    ?time_node rdf:type time:Instant .
    ?time_node rdfs:label ?time_label .
    ?time_node time:inXSDDateTime ?datetime .
    ?time_node time:inXSDgYear ?year .
} limit 1

```

#### Time Intervals

Time intervals are nodes that represent a period of time. It has two properties of interest that point to nodes time instants (see above):

1. time:hasBeginning (when the period of time starts)
2. time:hasEnd (when the period of time ends)

#### Connecting things to time

When connecting events and data to time KnowWhereGraph uses its own term, `kwg-ont:hasTemporalScope`. You will *always* use this relation to obtain information about time.

An example is shown below where the temporal information about a `kwg-ont:Hazard` is retrieved, using all the techniques from above

#### Filtering on Time

### Understanding Space

Spatial information is encoded using the [GeoSPARQL](https://www.ogc.org/standard/geosparql/) standard. 

This means that

1. You can expect all classes that represent spatial *things* to be some subclass of `geo:Feature`.
2. Geometries will be connected to features with `geo:hasGeometry` and `geo:hasDefaultGeometry`
3. Geometry literals (the actual coordinates that make up the geometry) can be obtained from the geometry node with `geo:asWKT`

An example is shown below where the geometry node is retrieved, followed by the geometry literal.

```SPARQL
PREFIX geo: <http://www.opengis.net/ont/geosparql#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
select ?s_label ?o_label ?geometry where { 
    ?s geo:hasGeometry ?o .
    ?s rdfs:label ?s_label .
    ?o rdfs:label ?o_label .
    ?o geo:asWKT ?geometry .
} limit 1
```

### Understanding Data Values

Data is encoded using the [SOSA](https://www.w3.org/TR/vocab-ssn/) ontology.
