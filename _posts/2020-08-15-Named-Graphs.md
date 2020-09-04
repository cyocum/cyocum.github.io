---
layout: post
date: 2020-08-15
title: "RDF Datasets and Named Graphs"
author: cgy
---

{% assign author = site.data.authors[page.author]%}
{{author.name}}

ORCID: <a href="https://orcid.org/{{ author.orcid }}" title="{{author.name}}">{{author.orcid}}</a>

Following on from earlier posts, this post will disucss the concept of
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
geneaological tradition is not presented as a single whole.  The texts
that comprise it are contained within MSS which are seperated in time
and space.  These were all compiled from various sources, including
each other, and exist in their own right.  This situation is very
messy from a data point of view.  There are competing constraints
which need to be balanced when attempting to translate the situation
as it stands within the MS tradition to the digital.  On the one hand,
there is a solid core of information contained with in the major MS
geneaological collections.  On the other hand, there are substantive
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
Graphs, this reading proceedure would need to be done for each URI
returned.  This is ineffeicent and, as will be seen, difficult, but
not impossible, to replicate in SPARQL.

So, if, for the sake of modling the genealogical situation in the
MSS, we wished to add the manuscripts as objects known to the
Triplestore, we would need to find a way of doing that rather than the
method delinated above.  Named Graphs model just this situation.  If
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

The above states that the triples that consitute an instance of Find
are a part of the `<http://example.com/LL>` graph.  This is much
easier to write than adding an extra `<http://example.com/LL>` to each
an every triple.  Looking through the IrishGen dataset, the reader
will notice that each file states which named graph a set of triples
to belongs to.  These Named Graphs represent the MS whence a set of
triples derives.  This allows the triples to be seprated but still
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

This produces 19 results inline with expectations.  However, SPARQL
has another method for doing this related to `from` called `from
named`.  If this is applied to the query above with no other
modifications, the result will be again _zero_.  This is because `from
named` does not pull the named graph into the default graph, it only
makes it avaliable to the rest of the query.  To actually get the
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

The query above will again only return 19 results.

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?a ?b
from <http://example.com/Rawl_B502>
from named <http://example.com/LL>
where {
    ?b irishRel:nomName "Find"
    graph ?g {
        ?a irishRel:nomName "Find"
    }

```

This really rather odd but illustrative query will return 475 results
where geneological information from Rawl B 502 is pulled into the
default graph and searched while information from LL stays within its
sub-graph and is searched seperately.  The combination is creates to
columns that are slightly smashed togther but it illistrates the point
about where triples are placed using the two keywords.

The last question that may be asked is: how does a user query all
graphs without needing to specify each?  This is not currently
possible in the SPARQL 1.1 standard so this is where things become
complicated and the solution depends on the Triplestore the user
chooses.  In Stardog there are two ways to do this.  First there is a
special graph named `<tag:stardog:api:context:all>`.  This graph will
automatically pull all triples known into the default graph.  Second,
the user can set a property on the database in the Stardog settings
named `Query All Graphs` which makes everything known by Stardog
avaliable to be queried without needing to specify the graphs.

One last note about how Stardog deals with Named Graphs and
`owl:sameAs`.  Stardog's documentation
[states](https://www.stardog.com/docs/#_same_as_reasoning) that:

> The way sameAs reasoning works differs from the OWL semantics
> slightly in the sense that Stardog designates one canonical
> individual for each sameAs equivalence set and only returns the
> canonical individual. This avoids the combinatorial explosion in
> query results while providing the data integration benefits.

This means that if the user searches a graph, if the conanical
`owl:sameAs` is chosen such that it is in a different graph, it could
appear in results in place of the one that is in the graph being
searched.  This can cause suprising results; however, there is nothing
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

select ?a
from <http://example.com/LL>
where {
    ?a irishRel:nomName "Find"
}
```

This will return 15 results versus Stardog's 19. 

Running the query with `from named` instead:

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

Also yields 15 results.

As a side note, if you follow the `owl:sameAs` links for the ones that
do not respect the graph boundries, the reader will discover that they
correspond to some of the ones returned by Stardog thus a correlation
table can be constructed.

| Stardog                                                               | GraphDB                                                                                         |
| http://example.com/LL/lagin.trig#Find                                 |                                                                                                 |
| http://example.com/LL/do_thaicraige_arad.trig#Find                    | http://example.com/LL/do_thaicraige_arad.trig#Find                                              |
| http://example.com/LL/dáil_caiss.trig#Find                            | http://example.com/Rawl_B502/de_genelogia_dál_chais_ut_inuenitur_in_psalterio_caissil.trig#Find |
| http://example.com/LL/clanna_ébir_i_l-leith_chuind.trig#Find-0dc31110 |                                                                                                 |
| http://example.com/LL/ciannacht.trig#Find                             |                                                                                                 |
| http://example.com/LL/clanna_ébir_i_l-leith_chuind.trig#Find-7b6d0720 | http://example.com/Rawl_B502/clanna_ébeir_h_i_l_leith_chuind.trig#Find                          |
| http://example.com/LL/clanna_ébir_i_l-leith_chuind.trig#Find-194ec360 | http://example.com/Rawl_B502/_92.trig#Find                                                      |
| http://example.com/LL/forslonti_dáil_messi_corb.trig#Find-6ec2dee0    | http://example.com/LL/forslonti_dáil_messi_corb.trig#Find-6ec2dee0                              |
| http://example.com/LL/rig_ailig.trig#Find                             | http://example.com/Rawl_B502/_94.trig#Find                                                      |
| http://example.com/LL/h_n_echdach.trig#Find                           | http://example.com/LL/h_n_echdach.trig#Find                                                     |
| http://example.com/LL/n_dési.trig#Find                                | http://example.com/LL/n_dési.trig#Find                                                          |
| http://example.com/LL/tairdelbaig.trig#Find                           | http://example.com/LL/tairdelbaig.trig#Find                                                     |
| http://example.com/LL/h_airgialla.trig#Find                           | http://example.com/LL/h_airgialla.trig#Find                                                     |
| http://example.com/LL/genelach_clainde_brannduib.trig#Find-68dfd695   | http://example.com/LL/genelach_clainde_brannduib.trig#Find-68dfd695                             |
| http://example.com/LL/rig_h_falge.trig#Fhind                          | http://example.com/Rawl_B502/genelach_úa_failgi.trig#Find                                       |
| http://example.com/LL/síl_daimini.trig#Fhind                          | http://example.com/Rawl_B502/genelach_úa_fáeláin.trig#Find                                      |
| http://example.com/LL/síl_daimini.trig#Fhind-a5cb5100                 |                                                                                                 |
| http://example.com/LL/flaithe_h_riacain.trig#Find                     | http://example.com/LL/flaithe_h_riacain.trig#Find                                               |
| http://example.com/LL/genelach_h_falgi.trig#Find                      | http://example.com/LL/genelach_h_falgi.trig#Find                                                |


# Conclusion

As the reader can see, choosing a Triplestore has consequences for
querying.  GraphDB's eilision mentioned above has this consequence: it
makes searching the amalgam of the data taken from the MSS easy but
makes searching a single MS confusing.  Stardog adheres to the
standard more closely but because of its backwards chaining reasoning,
it is excessively slow when reasoning is enabled.  Moreover, the way
in which `owl:sameAs` is treated in Stardog can cause confusing
results.  Although, this is also a problem in GraphDB as demonstrated.
A user will need to keep this in mind when choosing which Triplestore
to use for a specific task.

