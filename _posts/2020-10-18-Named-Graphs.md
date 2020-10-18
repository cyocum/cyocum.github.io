---
layout: post
date: 2020-10-18
title: RDF Datasets and Named Graphs
author: cgy
---

{% assign author = site.data.authors[page.author]%}
{{author.name}}

ORCID: <a href="https://orcid.org/{{ author.orcid }}" title="{{author.name}}">{{author.orcid}}</a>

Following on from earlier posts, this post will discuss the concept of
[RDF Datasets](https://www.w3.org/TR/rdf11-datasets/) also known as
Named Graphs.  The reader will need to be familiar with the basics of
Linked Data and SPARQL and reading earlier posts would be advisable
before continuing.  This post will proceed in four parts.  First, a
practical example as to why Named Graphs are necessary and useful.
Second, a discussion of the Named Graphs and the Default Graph and how
they interact.  Third, how Named Graphs are searched via SPARQL in the
context of IrishGen.  Fourth, and finally, how and why the
`owl:sameAs` predicate is used in IrishGen and what effects it has to
the result sets returned from the two Triplestores.  The outcome is
that the reader will have an appreciation for: why Named Graphs exist,
what this means for the structure of the IrishGen dataset, and how to
use this facility in their queries.  Additionally, the user will get
an appreciation for how and why `owl:sameAs` is handled in the two
Triplestores and what effect that will have on their query's result
sets.

# Provenance of Data

The medieval Irish genealogical tradition is not presented as a single
whole.  The texts that comprise it are contained within MSS which are
separated in time and space.  These were all compiled from various
sources and exist in their own right.  This situation is very messy
from a data modelling point of view.  There are competing constraints
which need to be balanced when attempting to translate the situation
as it stands within the MS tradition to the Linked Data universe.  On
the one hand, there is a solid core of information contained within
the major MS genealogical collections.  On the other hand, there are
substantive differences which must be respected.  Additionally, for
users of IrishGen, it is equally important to be able to search one
branch of the tradition or the other.  Without Named Graphs, as will
be discussed shortly, users who only wished to search, for instance,
the [Book of Leinster](https://en.wikipedia.org/wiki/Book_of_Leinster)
would have to contort their searches to narrow them.

When using a database such as IrishGen which replicates in
computational form data which is taken directly from primary sources,
it is imperative that IrishGen is clear concerning whence that
information derives.  As it stands, a triple does not carry with it a
method for partitioning information.  Within IrishGen, there is an
ad-hoc attempt at this by encoding the MS that a triple is from into
the triple's URL.  While this method can work with some extra
diligence from the user, it is far from perfect.  Thankfully, others
have encountered the same problem and the Semantic Web community has
created the concept of a "Named Graph".

To illustrate this, the situation in which there is no Named Graph for
a triple will be shown then the situation with a Named Graph will be
shown.

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?a
where {
    ?a irishRel:nomName "Find"
}
```

The above query in a situation where Named Graphs are not in use will
return every single triple that happens to match the Basic Graph
Pattern without respect to what manuscript the triple may have come
from.  From a research point of view, this is very unhelpful.
Usually, researchers will want to explore a specific genealogical
text.  While it can be very interesting to do a meta-analysis of the
entire corpus as a whole, such as when counting the number of distinct
individuals recorded, or useful when the user is searching to see if
information happens to be contained in the genealogies, such as when
the user has a name from another source and wishes to know if the same
name appears in the genealogical corpus, a database wide search is
often not what is wanted.

The solution to this problem is to partition the dataset into
sub-graphs.  Named Graphs allow the user to define what sub-graph a
triple will belong to.  In the case of IrishGen, Named Graphs define
the relationship between the triple and the sub-graph.  As with all
things in Linked Data, the sub-graph is defined by a URL.  In the case
of IrishGen, the URL is `http://example.com/` plus the commonly used
scholarly abbreviation.  For instance, in the case of the Book of
Leinster the URL is: `http://example.com/LL`.

More technically, when a Named Graph is attached to a triple, it is
transformed into a "quad".  This extra bit of information is attached
to every triple in a Named Graph.  For instance, a triple will
normally look like this to a Triplestore:

```turtle
<http://example.com/LL/lagin.trig#Find> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://xmlns.com/foaf/0.1/Person>
```

This triple states informally that "this particular instance of Find
is a person".  When the Named Graph is added, the triple will be
transformed thus:

```turtle
<http://example.com/LL/lagin.trig#Find> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://xmlns.com/foaf/0.1/Person> <http://example.com/LL>
```

This is now a quad (it has four basic elements rather than three).
This quad informally states: "Find is a person in the graph
<http://example.com/LL>".  The quad above can be read more liberally
as "Find is a person in the Book of Leinster".  To make the above
slightly more comprehensible, the [TRiG](https://www.w3.org/TR/trig/)
file format slightly extends the
[Turtle](https://www.w3.org/TR/turtle/) format to accommodate the
sub-graph.  So, if a user looks at the data directly, rather than
through a Triplestore, it will look something like this:

```turtle
<http://example.com/LL> {
     <#Find>
         a foaf:Person;
         irishRel:nomName "Find";
         rel:descendantOf <#Baiscni>;
         rdfs:comment "Senchan Torpeist cecinit isin Cocangaib".
}

```

To search for the instances of a nominal name "Find" in the Book of
Leinster's genealogies, the SPARQL query would be:

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

This ensures, with some very important caveats explored below, that
only information from the Book of Leinster will be returned to this
query, allowing the user to narrow to a specific MS or set of MSS.

At the moment of writing, Named Graphs are not composable.  For
instance, you cannot nest a Named Graph within another Named Graph.
For readers who are more familiar with the structure of the medieval
Irish genealogies, you cannot nest an item graph within a manuscript
graph.

# Named Graphs and the Default Graph (RDF Dataset)

Because triples were created before quads, the two ways of denoting
information in RDF must be compatible with each other.  This is done
by defining two separate domains: the Default Graph and any Named
Graphs.  Triples exist in the Default Graph while quads exist within
their own, distinct, Named Graphs as defined by the dataset.  In the
case of IrishGen currently, there are no triples but only quads.
Every bit of information stored within IrishGen is given a MS as a
Named Graph (sometimes termed "context").  This has consequences for
how SPARQL handles queries which deal with Named Graphs.

Sadly, Named Graphs have [many formal
definitions](https://www.w3.org/TR/2014/NOTE-rdf11-datasets-20140225/)
which can make understanding them confusing.  This situation is due to
the fact that the Semantic Web community has yet to come to a
consensus as to the direct formal meaning of a Named Graph.  However,
this does not detract from their usefulness in both SPARQL and
IrishGen.

# Named Graphs and SPARQL

From the foregoing sections, it may seem straightforward to add this
functionality to SPARQL.  Sadly, it is not.  The two implementations
that are recommended (Stardog and Ontotext GraphDB) handle RDF
Datasets in very different ways.  To anticipate slightly, Stardog
handles Named Graphs in a way which adheres to the standard closely
but GraphDB attempts to elide over the differences between the Named
Graph and the Default Graph.  While this elision on GraphDB's part
makes getting started with Named Graphs easier, it makes the entire
situation more complicated in the long run.

First, let us look at a naive query in Stardog:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?a
where {
    ?a irishRel:nomName "Find"
}
```

The query above will produce _zero_ results when used with IrishGen.
This has caused confusion to the curators several times over the
course of the project.  Why does this produce no results?  It is
because this query does not query the entire graph; it only queries
the Default Graph, the graph which contains only triples.  Find does
not appear in the Default Graph and thus the query will produce no
results.  The key to understanding this is that IrishGen is split into
MS Named Graphs and does not have triples in the Default Graph thus
searching the Default Graph will yield no results because there is
nothing in the Default Graph.

How do we create a query which will return what is expected?  For this
SPARQL has two methods, the first is the `from` keyword.  This keyword
is followed by the Named Graph that is of interest.  What this does is
pull the entire Named Graph into the Default Graph.  In effect, it
reduces the quads from the Named Graph to triples in the Default Graph
and then searches in the Default Graph. So, if we wanted to pull all
quads from LL into the Default Graph, we would use from like so:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?a
from <http://example.com/LL>
where {
    ?a irishRel:nomName "Find"
}
```

This will search the Default Graph with all the quads added from the
Named Graph `<http://example.com/LL>`.  This is useful when a user
wishes to combine triples and quads into a single query.

The query above produces 19 results which meets expectations.  If the
user only wishes to search quads from a single Named Graph, SPARQL has
the `from named` construct which, in effect, narrows a query to that
particular Named Graph.

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

Before continuing to demonstrate combining `from` and `from named` in
a single query, it is useful to restate the purpose of these
statements as they can be confusing. `from` pulls quads from a Named
Graph into the Default Graph.  This means that triples and quads can
be merged together in a single query.  `from named` retains the
distinction between Default Graph and Named graph.  Queries that use
`from named` will need to use the `graph` SPARQL keyword to enable
access to quads from a Named Graph but the Default Graph is always
available outside the `graph` keyword.  For instance:

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
does not search the Default Graph.

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
quads from [Rawl
B502](https://,en.wikipedia.org/wiki/Bodleian_Library,_MS_Rawlinson_B_502)
are pulled into the default graph and searched while information from
LL stays within its Named Graph and is searched separately.  The
combination of both graphs is merged using the `union` SPARQL keyword
which merges two result sets into a single result set.  However, see
below concerning how Stardog's and GraphDB's `owl:sameAs` semantics
can make these results confusing in different ways depending on the
Triplestore used.  The above query can be written just the same using
two `from named` statements and using just the `graph` keyword.  As an
example:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?a ?g
from named <http://example.com/Rawl_B502>
from named <http://example.com/LL>
where {
	graph ?g {
		?a irishRel:nomName "Find"
	}	
}
```

The last question that may be asked is: how does a user query all
graphs without needing to specify each?  This is not currently
possible in the [SPARQL 1.1
standard](https://www.w3.org/TR/sparql11-overview/) so this is where
the situation becomes complicated and the solution depends on the
Triplestore that the user chooses.  In Stardog there are two ways to
do this.  First there is a special graph named
`<tag:stardog:api:context:all>`.  This special graph, only available
in Stardog, will automatically pull Named Graph all quads into the
Default Graph.  Second, the user can set a property on the database in
the Stardog settings named `Query All Graphs` which pulls all Named
Graph quads into the Default Graph by default, which makes using the
`from named`, `from`, and `graph` keywords unnecessary to search all
quads at once.

Turning to GraphDB, the situation is slightly different.  Taking the
original naive query from above:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?a
where {
    ?a irishRel:nomName "Find"
}
```

In GraphDB, this query will return 105 results, which is the total
number of instances of "Find" in the dataset.  Why does this return so
many results?  This is due to the way in which GraphDB constructs its
[Default
Graph](http://graphdb.ontotext.com/documentation/standard/query-behaviour.html):

> > GraphDB constructs the default dataset as follows:
>
>     The dataset’s default graph contains the merge of the database’s default graph AND all the database Named Graphs;
>
>     The dataset contains all Named Graphs from the database.

The reason given in the documentation is thus:

> There are two reasons for this behavior:
>
>     It provides an easy way to execute a triple pattern query over all stored RDF statements.
>
>     It allows all Named Graph names to be discovered, i.e., with this query: SELECT ?g { GRAPH ?g { ?s ?p ?o } }.

The query returns so many results because all queries are as if
Stardog's "Query All Graph" preference is enabled.  In other words,
all quads in all Named Graphs and all triples in the Default Graph are
merged by default in GraphDB.  If a query specifies a `from` and `from
named`, GraphDB's documentation states:

> If either FROM or FROM NAMED are used, the database’s default graph is no longer used as input for processing this query. In effect, the combination of FROM and FROM NAMED clauses exactly defines the dataset.

In other words, to have GraphDB act in a similar way as Stardog's
default, the user will need to specify the datasets exactly in their
query.

# SameAs and SPARQL searches

The medieval Irish genealogies contain instances of the same
individual across multiple MSS.  IrishGen links the instances of these
individuals to each other using OWL's `sameAs` predicate.  Using this
predicate to link all instances of an individual together has
consequences for searching which has an effect on the use of Named
Graphs and a large effect on the results returned.  Users must be
aware of this as it can have a large effect on the results returned
and the interpretation of the results returned.

Stardog's documentation
[states](https://www.stardog.com/docs/#_same_as_reasoning) that:

> The way sameAs reasoning works differs from the OWL semantics
> slightly in the sense that Stardog designates one canonical
> individual for each sameAs equivalence set and only returns the
> canonical individual. This avoids the combinatorial explosion in
> query results while providing the data integration benefits.

This means that if the user searches a Named Graph, if the canonical
`owl:sameAs` is chosen such that it is in a different graph, it could
appear in results in place of the one that is in the Named Graph being
searched.  This can cause surprise to a user but they are technically
and by definition the same; however, there is nothing that anyone can
do about this due to the fact that Stardog does the above.
Additionally, for an individual who is the same across many graphs,
this can change each time the database is loaded as the system
randomly assigns a canonical individual.

GraphDB has an approach which is the exact opposite to Stardog.
Instead of using forward-chaining to create `sameAs` relations,
GraphDB creates an [equivalence
class](https://www.w3.org/TR/owl2-syntax/#Equivalent_Classes) and does
backwards-chaining:

> There is no restriction on how to choose this single node that will represent the class as a whole, so we pick the first node that enters the class. After creating such a class, all statements with nodes from this class are altered to use the class representative. These statements also participate in the inference.
>
> The equivalence classes may grow when more owl:sameAs statements containing nodes from the class are added to the repository. Every time you add a new owl:sameAs statement linking two classes, they merge into a single class.
>
> During query evaluation, GraphDB uses a kind of backward chaining by enumerating equivalent URIs, thus guaranteeing the completeness of the inference and query results. It takes special care to ensure that this optimization does not hinder the ability to distinguish between explicit and implicit statements.

More detail is available in GraphDB's
[documentation](http://graphdb.ontotext.com/documentation/standard/sameas-optimisation.html).

To demonstrate how this effects GraphDB's results, if the user runs
the query in Stardog:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?a
from <http://example.com/LL>
where {
    ?a irishRel:nomName "Find"
}
```

There will be 19 results.

| http://example.com/LL/ciannacht.trig#Find                             |
| http://example.com/LL/clanna_ébir_i_l-leith_chuind.trig#Find-0dc31110 |
| http://example.com/LL/clanna_ébir_i_l-leith_chuind.trig#Find-7b6d0720 |
| http://example.com/LL/clanna_ébir_i_l-leith_chuind.trig#Find-194ec360 |
| http://example.com/LL/dáil_caiss.trig#Find                            |
| http://example.com/LL/do_thaicraige_arad.trig#Find                    |
| http://example.com/LL/flaithe_h_riacain.trig#Find                     |
| http://example.com/LL/forslonti_dáil_messi_corb.trig#Find-6ec2dee0    |
| http://example.com/LL/genelach_clainde_brannduib.trig#Find-68dfd695   |
| http://example.com/LL/genelach_h_falgi.trig#Find                      |
| http://example.com/LL/h_airgialla.trig#Find                           |
| http://example.com/LL/h_n_echdach.trig#Find                           |
| http://example.com/LL/lagin.trig#Find                                 |
| http://example.com/LL/n_dési.trig#Find                                |
| http://example.com/LL/rig_ailig.trig#Find                             |
| http://example.com/LL/rig_h_falge.trig#Fhind                          |
| http://example.com/LL/síl_daimini.trig#Fhind                          |
| http://example.com/LL/síl_daimini.trig#Fhind-a5cb5100                 |
| http://example.com/LL/tairdelbaig.trig#Find                           |

If it is run under GraphDB, it will return 16 results.  The difference is:

| http://example.com/LL/clanna_ébir_i_l-leith_chuind.trig#Find-7b6d0720 |
| http://example.com/LL/clanna_ébir_i_l-leith_chuind.trig#Find-194ec360 |
| http://example.com/LL/síl_daimini.trig#Fhind-a5cb5100                 |

Which one of these results is correct?  Both are if the user considers
that GraphDB creates an equivalence class that combines answers when
doing inferencing.  The two answers can be reconciled using GraphDB's
pseudo-graphs:

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

What is `onto:explicit`?  As the GraphDB documentation discussed above states:

> FROM onto:explicit – using only this clause (or with FROM onto:disable-sameAs) produces the same results as when the inferencer is disabled (as with the empty ruleset). This means that the ruleset and the disable-sameAs parameter do not affect the results.

This disables the inferencer and returns what is explicitly stated in
the IrishGen files.  Is this what the user will always want though?
That is up to the user at the time the query is run.

To return to the query above which contains two Named Graphs, if the
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

Although, it would be easier to do the following:

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

The above is easier to write and read.  This returns 44 results, which
contrasts with the 21 results returned from Stardog.  This means that
there are 44 instances of the name "Find" but only 21 distinct
individuals who could be identified as `owl:sameAs` each other.  To
state this slightly differently, the key to the distinction is that
there are 44 instances of the name "Find" while there are only 21
actual individuals who can be treated the same as each other.  To
reinstate the `owl:sameAs` and reimpose the `owl:sameAs` logic, all
one needs to do is remove the `onto:explicit` and the above query will
return 21 just the same as Stardog.

As the reader can see, using OWL 2's `sameAs` has various effects on
the outcome of queries.  This needs to be kept in mind whenever a
query is run.  A user must keep in mind the effects of choosing one
Triplestore over another as each Triplestore makes different choices
on how to represent the OWL 2 standard and these choices can have a
large effect on the result sets returned.

# Conclusion

Named Graphs allow IrishGen to partition the various MSS sources into
their own sub-graphs.  This allows the user to interrogate one MS
tradition or another, which can be important to certain investigations
such as when a user is searching to verify whether a certain piece of
information exists in a certain MS version.  For instance, a name is
found within a MS and the user wishes to check the genealogies
contained in the same MS quickly to see if that name is found within
it.

However, choosing a Triplestore has consequences for querying in the
presence of Named Graphs.  GraphDB's implementation mentioned above
has this consequence: it makes searching the amalgam of the data taken
from all MSS easy.  Stardog adheres to the standard more closely but
because of its backwards chaining reasoning, it can be very slow when
reasoning is enabled.  Moreover, the way in which `owl:sameAs` is
treated in Stardog and GraphDB has an effect on the result set that
needs to be considered when writing and running queries.
