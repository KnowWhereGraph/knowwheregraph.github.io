# Ontology

*KnowWhereGraph has a large ontology*.

It's no secret that navigating *any* ontology is a time consuming, often difficult process. KnowWhereGraph is no exception.

We provide a [Widoco](https://github.com/dgarijo/Widoco) documentation page for our ontology which includes diagrams to help compartmentalize, visualize, and digest the various components in a way to help better understand how the graph is structured.

[KnowWhereGraph Widoco Ontology Docs](https://stko-kwg.geog.ucsb.edu/lod/ontology)

## Key Points

There are several key points that make working with and understanding the ontology easier.

### 1. Understanding Time

KnowWhereGraph uses [OWL time](https://www.w3.org/TR/owl-time/) for temporal information. We don't use the *entire* ontology, just a few pieces that are summarized below.

#### Time Instants

The most basic node that represents time are nodes of type `time:Instant`. These nodes typically have four properties:

1. A label with the full datetime
2. An XSD:Date representation
3. An XSD:DateTime representation
4. The year

An example is shown below where the full datetime and year are retrieved from a node of type, `time:Instant`.

```SPARQL
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

1. `time:hasBeginning` (when the period of time starts)
2. `time:hasEnd` (when the period of time ends)

The convenient bit about nodes of this type are that they reference `time:Instant` nodes through the two relations above.

An example of pulling out the start and end datetimes of an interval is given below. Note how the pattern of querying `time:Instant` is used. *All we've done is start at a `tine:Interval` and grabbed the values from the attached `time:Instant`*.

```SPARQL
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX time: <http://www.w3.org/2006/time#>
select ?time_label?datetime_begin ?datetime_end where { 
    ?time_onterval rdf:type time:Interval .
    ?time_onterval rdfs:label ?time_label .
    ?time_onterval time:hasBeginning ?time_begin .
    ?time_onterval time:hasEnd ?time_end .
    ?time_begin time:inXSDDateTime ?datetime_begin .
    ?time_end time:inXSDDateTime ?datetime_end .
} limit 1
```

#### Connecting Things to Time

When connecting events and data to time KnowWhereGraph uses its own term, `kwg-ont:hasTemporalScope`. You will *always* use this relation to obtain information temporal information about an event.

One important note to take is that data of the same class can have either `time:Instant` *or* `time:Interval` data. This means that if you're querying for data of type `kwg-ont:Hazard` and asking for temporal information - you need to look for both `time:Instant` *and* `time:Interval` connections.

An example is shown below where we count the number of `kwg-ont:Hazard`s that are connected to `time:Instant`. At the time of writing this, there are 1,248,050 hazards with time instant data.

```SPARQL
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX time: <http://www.w3.org/2006/time#>
PREFIX kwg-ont: <http://stko-kwg.geog.ucsb.edu/lod/ontology/>
select (count (?scope) as ?count) where { 
    ?hazard rdf:type kwg-ont:Hazard .
    ?hazard kwg-ont:hasTemporalScope ?scope .
    ?scope rdf:type time:Instant .
}
```

Doing the same for intervals, we see that there are 1,048,577 events with time interval data.

```SPARQL
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX time: <http://www.w3.org/2006/time#>
PREFIX kwg-ont: <http://stko-kwg.geog.ucsb.edu/lod/ontology/>
select (count (?scope) as ?count) where { 
    ?hazard rdf:type kwg-ont:Hazard .
    ?hazard kwg-ont:hasTemporalScope ?scope .
    ?scope rdf:type time:Interval .
}
```

##### Example: Start and End Dates for All Hazards

Because some hazards have instants while others have intervals, the SPARQL becomes a little more complicated to get information about both at the same time. To avoid excluding each type, we have to make use of SPARQL's `OPTIONAL` clause.

```SPARQL
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX time: <http://www.w3.org/2006/time#>
PREFIX kwg-ont: <http://stko-kwg.geog.ucsb.edu/lod/ontology/>
select ?datetime ?datetime_begin ?datetime_end where { 
    ?hazard rdf:type kwg-ont:Hazard .
    ?hazard kwg-ont:hasTemporalScope ?hazard_time .
    OPTIONAL {
        ?hazard_time rdf:type time:Interval .
        ?hazard_time time:hasBeginning ?time_begin .
        ?hazard_time time:hasBeginning ?time_end .
        ?time_begin time:inXSDDateTime ?datetime_begin .
        ?time_end time:inXSDDateTime ?datetime_end .
    }
    OPTIONAL {
        ?hazard_time rdf:type time:Instant .
        ?hazard_time time:inXSDDateTime ?datetime .
    }
} LIMIT 10
```

The query results are shown below. The important thing to note here is that we have datetimes for instants and the full range for time intervals.

<img src="./images/ontology/time_combined.png" alt="drawing" width="900"/>

### 2. Understanding Space

Spatial information is encoded using the [GeoSPARQL](https://www.ogc.org/standard/geosparql/) standard.

This means that

1. You can expect all classes that represent spatial things to be some subclass of `geo:Feature`.
2. Geometries will be connected to features with `geo:hasGeometry` and `geo:hasDefaultGeometry`
3. Geometry literals (the actual coordinates that make up the geometry) can be obtained from the geometry node with `geo:asWKT`

An example is shown below where the geometry node for a `kwg-ont:Hazard` is retrieved, followed by its geometry literal. This pattern is used across KnowWhereGraph where different node types are in place of `kwg-ont:Hazard`.

```SPARQL
PREFIX geo: <http://www.opengis.net/ont/geosparql#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX kwg-ont: <http://stko-kwg.geog.ucsb.edu/lod/ontology/>
select ?hazard_label ?geometry_label ?geometry_literal where {
    ?hazard rdf:type kwg-ont:Hazard .
    ?hazard geo:hasGeometry ?geometry .
    ?hazard rdfs:label ?hazard_label .
    ?geometry rdfs:label ?geometry_label .
    ?geometry geo:asWKT ?geometry_literal .
} limit 1
```

The results for the query are shown below.

<img src="./images/ontology/geometry.png" alt="drawing" width="900"/>

### 3. Understanding Data Values

Data is encoded using the [SOSA](https://www.w3.org/TR/vocab-ssn/) ontology. 

## Conclusions

