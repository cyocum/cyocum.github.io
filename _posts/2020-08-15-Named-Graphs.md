---
layout: post
date: 2020-08-15
title: "RDF Datasets and Named Graphs"
author: cgy
---

{% assign author = site.data.authors[page.author]%}
{{author.name}}

ORCID: <a href="https://orcid.org/{{ author.orcid }}" title="{{author.name}}">{{author.orcid}}</a>

Following on from earlier posts, this post will discuss the concept of
[RDF Datasets](https://www.w3.org/TR/rdf11-datasets/) also known as
Named Graphs.  The reader will need to be familiar with the basics of
Linked Data and SPARQL and reading earlier posts would be advisable
before continuing.  This post will proceed in three parts.  First, a
theoretical overview of the situation both in terms of RDF and how
that interacts with IrishGen.  Second, what a Named Graph looks like
within the IrishGen dataset and how to read the dataset in light of
Named Graphs.  Third, and finally, how to use named graphs with SPARQL
and what capabilities they allow when searching the database.  The
outcome is that the reader will have an appreciation for: why Named
Graphs exist, what this means for the structure of the IrishGen
dataset, and how to use this facility in their research.

# RDF Datasets (Named Graphs)

Informally, an RDF Dataset is constructed of, at least, two graphs.
The first graph is the Default Graph.  The second is a sub-graph which
is distinct from the default graph and is given an arbitrary name by
the user.  By default, all triples will exist within the Default
Graph.  Any triple which exists within a named graph will have an
extra part (sometimes termed "context") which will turn the triple
into a quad.

Formally, an RDF Dataset is a collection of RDF graphs plus an unnamed
default graph.  Each, except for the default graph, is a collection of
triples that form a part of that named graph.  The name for the graph
is a URI, everything in Linked Data is a URI, which defines the scope
of the graph.  In practice, what this means is that an RDF triple when
defined in a Named Graph becomes an RDF quad.

The reason for the quad is that it allows users to partition data from
different but related sources in a coherent way.  All triples that are
contained within a named graph can be searched without needing to
search the entire graph.  For systems that potentially contain
millions or billions of triples, this gives large speed improvements
as a Triplestore can ignore triples in other Named Graphs.

However, there is one small problem. What about plain triples that are
not apart of any Named Graph?  These are placed within the default
unnamed graph.  Thus, when searching, the user will need to choose how
and when to include the default graph.  If the user does not
understand how default graphs and named graphs interact, their queries
will come up empty or, even worse, leave out important information.

This is an important concept for IrishGen.  The medieval Irish
genealogical tradition is not presented as a single whole.  The texts
that comprise it are contained within MSS which are separated in time
and space.  These were all compiled from various sources, including
each other, and exist in their own right.  This situation is very
messy from a data point of view.  There are competing constraints
which need to be balanced when attempting to translate the situation
as it stands within the MS tradition to the digital.  On the one hand,
there is a solid core of information contained with in the major MS
genealogical collections.  On the other hand, there are substantive
differences which must be respected.  Additionally, for users of
IrishGen, it is equally important to be able to search one branch of
the tradition or the other.  Without Named Graphs, as will be
discussed shortly, users who only wished to search, for instance, LL
would have to contort their searches to narrow their search.
Otherwise, users would have to wade through the entire graph without
respect to the MS tradition.

# IrishGen and Named Graphs

As an example of the problem of MS provenance without Named Graphs,
take a triple concerning Find mac Cumail from LL,

```turtle
<#Find>
    a foaf:Person;
    irishRel:nomName "Find";
    rel:descendantOf <#Baiscni>;
    rdfs:comment "Senchan Torpeist cecinit isin Cocangaib".
```

If one were to encounter this triple in the dataset, is it obvious
which manuscript this triple comes from? The answer is yes and no.
First of all, if you take the URI fragment from the first triple
`<#Find> a foaf:Person;` then apply the `@base` from the beginning of
the file: `<http://example.com/LL/lagin.trig#Find> a foaf:Person`.
From there, you can read the URL as follows: `http://example.com/` can
be ignored as it is the base URI that is a portion that is necessary
to fulfil the RDF framework to have a URL but is otherwise
meaningless; the `/LL/` portion of the URL, however, looks very much
like the manuscript abbreviation for the Book of Leinster (LL) and, in
fact, it is; the next portion of the URI is `lagin.trig` which is the
current file; finally, `#Find` is the individual.  Without Named
Graphs, this reading procedure would need to be done for each URI
returned.  This is inefficient and, as will be seen, difficult, but
not impossible, to replicate in SPARQL.

So, if, for the sake of modelling the genealogical situation in the
MSS, we wished to add the manuscripts as objects known to the
Triplestore, we would need to find a way of doing that rather than the
method delineated above.  Named Graphs model just this situation.  If
we wished to indicate the manuscript within which a triple appears as
a sub-graph of the nameless default graph, we can do this in the TriG
file format like so:

```turtle
<http://example.com/LL> {
    <#Find>
        a foaf:Person;
		irishRel:nomName "Find";
		rel:descendantOf <#Baiscni>;
		rdfs:comment "Senchan Torpeist cecinit isin Cocangaib".
}
```

The above states that the triples that constitute an instance of Find
are a part of the `<http://example.com/LL>` graph.  This is much
easier to write than adding an extra `<http://example.com/LL>` to each
an every triple.  Looking through the IrishGen dataset, the reader
will notice that each file states which named graph a set of triples
to belongs to.  These Named Graphs represent the MS whence a set of
triples derives.  This allows the triples to be separated but still
linked across graphs, which is the entire point of Linked Data.

# Named Graphs and SPARQL

From the foregoing sections, it may seem straight-forward to add this
functionality to SPARQL.  Sadly, it is not.  The two implementations
that are recommended (Stardog and Ontotext GraphDB) handle RDF
Datasets in very different ways.  To anticipate slightly, Stardog
handles Named Graphs in a way which adheres to the standard closely
but GraphDB attempts to elide over the changes to SPARQL which are
necessary to give effect to Named Graphs.  While this elision on
GraphDB's part makes getting started with Named Graphs easier, it
makes the entire situation more complicated in the long run.

First, let us look at a naive query in Stardog:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?a
where {
    ?a irishRel:nomName "Find"
}
```

The query above will produce _zero_ results.  This has even cause
confusion to the curators several times.  Why does this produce no
results?  It is because this query does not query the entire graph; it
only queries the nameless default graph.  Find does not appear in the
nameless default graph and thus it will produce no results.  How do we
create a query which will return what is expected?  For this SPARQL
has two methods, the first is the `from` keyword.  This keyword is
followed by the named graph that is of interest.  What this does is
pull the entire Named Graph into the nameless default Named Graph.
So, if we wanted to pull all triples from LL into the default graph,
we would use from like so:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?a
from <http://example.com/LL>
where {
    ?a irishRel:nomName "Find"
}
```

This produces 19 results which meets expectations.  However, SPARQL
has another method for doing this related to `from` called `from
named`.  If this is applied to the query above with no other
modifications, the result will be again _zero_.  This is because `from
named` does not pull the named graph into the default graph, it only
makes it available to the rest of the query.  To actually get the
expected result, the query needs to be like so:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?a
from named <http://example.com/LL>
where {
    graph ?g {
        ?a irishRel:nomName "Find"
    }
}
```

This will return the expected 19 results. `from` and `from named` can
be used in conjunction to create more complicated sets of searchable
graphs.  For instance:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?a
from <http://example.com/Rawl_B502>
from named <http://example.com/LL>
where {
    graph ?g {
        ?a irishRel:nomName "Find"
    }
}
```

The query above will again only return 19 results, which is because it
does not search the default graph.

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?a ?g
from <http://example.com/Rawl_B502>
from named <http://example.com/LL>
where {
    { ?a irishRel:nomName "Find" }
    union {
		graph ?g {
			?a irishRel:nomName "Find"
		}
    }	
}
```

This illustrative query will return 21 results where genealogical
information from Rawl B 502 is pulled into the default graph and
searched while information from LL stays within its sub-graph and is
searched separately.  The combination of both graphs is merged using
the `union` SPARQL keyword which merges two queries into a single
result set.  However, see below concerning how Stardogs and GraphDB's
`owl:sameAs` semantics can make these results confusing in different
ways depending on the Triplestore used.

The last question that may be asked is: how does a user query all
graphs without needing to specify each?  This is not currently
possible in the [SPARQL 1.1
standard](https://www.w3.org/TR/sparql11-overview/) so this is where
the situation become complicated and the solution depends on the
Triplestore that the user chooses.  In Stardog there are two ways to
do this.  First there is a special graph named
`<tag:stardog:api:context:all>`.  This graph will automatically pull
all triples known into the default graph.  Second, the user can set a
property on the database in the Stardog settings named `Query All
Graphs` which makes everything known by Stardog available to be
queried without needing to specify the graphs.

One last note about how Stardog deals with Named Graphs and
`owl:sameAs`.  Stardog's documentation
[states](https://www.stardog.com/docs/#_same_as_reasoning) that:

> The way sameAs reasoning works differs from the OWL semantics
> slightly in the sense that Stardog designates one canonical
> individual for each sameAs equivalence set and only returns the
> canonical individual. This avoids the combinatorial explosion in
> query results while providing the data integration benefits.

This means that if the user searches a graph, if the canonical
`owl:sameAs` is chosen such that it is in a different graph, it could
appear in results in place of the one that is in the graph being
searched.  This can cause surprising results; however, there is nothing
that anyone can do about this due to the fact that Stardog does the
above.  Additionally, for an individual who is the same across many
graphs, this can change each time the database is loaded as the system
randomly assigns a canonical individual.

Turning to GraphDB, the situation is slightly different.  Taking the
original naive query from above:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?a
where {
    ?a irishRel:nomName "Find"
}
```

In GraphDB, this query will return 105 results.  Why does this return
so many results?  This is due to the way in which GraphDB constructs
its [default
graph](http://graphdb.ontotext.com/documentation/standard/query-behaviour.html):

> > GraphDB constructs the default dataset as follows:
>
>     The dataset’s default graph contains the merge of the database’s default graph AND all the database named graphs;
>
>     The dataset contains all named graphs from the database.

The reason given in the documentation is thus:

> There are two reasons for this behavior:
>
>     It provides an easy way to execute a triple pattern query over all stored RDF statements.
>
>     It allows all named graph names to be discovered, i.e., with this query: SELECT ?g { GRAPH ?g { ?s ?p ?o } }.

If the user runs the query:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 
prefix onto: <http://www.ontotext.com/>

select ?a
from <http://example.com/LL>
from onto:explicit
where {
    ?a irishRel:nomName "Find"
}
```

There will be 19 results as from Stardog.  What is `onto:explicit`?
This has to do with the way in which GraphDB deals with `owl:sameAs`
equivalency and acts much like Stardog's merging of those nodes that
have an effect on the result set.  To break this, `onto:explicit`
pseudo-name graph is used (see [Optimization of
owl:sameAs](http://graphdb.ontotext.com/documentation/standard/sameas-optimisation.html)).

To return to the query above which contains two named graphs, if the
user wished to have all the instances of `irishRel:nomName "Find"`
from LL and Rawl B502 without the `owl:sameAs` equivalencies
interfering with the results, the query would be:

```sparql
prefix onto: <http://www.ontotext.com/>
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?a ?g
from <http://example.com/Rawl_B502>
from named <http://example.com/LL>
from onto:explicit
from named onto:explicit
where {
    { ?a irishRel:nomName "Find" }
    union {
		graph ?g {
			?a irishRel:nomName "Find"
		}
    }	
}
```

Although, it would be easier to just do this following:

```sparql
prefix onto: <http://www.ontotext.com/>
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?a ?g
from named <http://example.com/Rawl_B502>
from named <http://example.com/LL>
from named onto:explicit
where {
	graph ?g {
		?a irishRel:nomName "Find"
	}
		
}
```

The above is easier to write and read.

# Conclusion

Named graphs allow IrishGen to partition the various MSS sources into
their own sub-graphs.  This allows the user to interrogate one MS
tradition or another, which can be important to certain
investigations.  However, choosing a Triplestore has consequences for
querying in the presence of named graphs.  GraphDB's elision
mentioned above has this consequence: it makes searching the amalgam
of the data taken from all MSS easy but makes searching a single MS
confusing.  Stardog adheres to the standard more closely but because
of its backwards chaining reasoning, it is excessively slow when
reasoning is enabled.  Moreover, the way in which `owl:sameAs` is
treated in Stardog and GraphDB can cause confusing and confounding
results.  A user will need to keep this in mind when choosing which
Triplestore to use for a specific task.

