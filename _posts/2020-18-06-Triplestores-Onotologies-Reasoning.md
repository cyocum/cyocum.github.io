---
layout: post
date: 2020-06-18
title: Triplestores, Ontologies, and Reasoning
author: cgy
---

{% assign author = site.data.authors[page.author]%}
{{author.name}}

ORCID: <a href="https://orcid.org/{{ author.orcid }}" title="{{author.name}}">{{author.orcid}}</a>

This post will cover the Semantic Web/Linked Data concepts of
Triplestores and Ontologies in depth with a focus on the IrishGen
dataset.  If the reader is new to Semantic Web/Linked Data then
beginning with ["IrishGen: RDF, Linked Data, and The Semantic Web"]({%
post_url 2020-06-17-IrishGen-RDF-Linked-Data-Semantic-Web %}) will
orient the reader before approaching this set of topics.

[IrishGen](https://github.com/cyocum/irish-gen) is, at its simplest, a
static set of text files which conform to a certain format that is
readable by computers and, to a degree, humans.  While this is useful
in itself, one can use text based string matching system like
[grep](https://www.gnu.org/software/grep/) to quickly search the files
for relevant information, it is sub-optimal and better systems exist.
[Tim Berners-Lee](https://orcid.org/0000-0003-1279-3709)'s original
vision for the Semantic Web was a software agent that would crawl the
web then using the structured information retrieved, answer user's
queries on the web (see ["Whatever Happened to the Semantic
Web?"](https://twobithistory.org/2018/05/27/semantic-web.html)).  To
be able to fulfil this vision, the software agent would need a
storage facility to store the found information.  Once the information
was gathered the software agent would then need a query system for the
user to ask their questions.[^1] These two systems exist: Triplestore
and [SPARQL](https://www.w3.org/TR/sparql11-query/), a detailed
discussion of which will be deferred to a future post.  Additionally,
to support those systems and take advantage of the first order logic
reasoning system afforded by RDF, a system of ontologies was created
which culminated in the [OWL 2
specification](https://www.w3.org/TR/owl2-overview/).


[^1]: This does raise the question that in an environment where
    information is added, deleted, and changed with startling
    velocity: will the software agent's data ever be current enough to
    answer a query accurately?

Due to the fact that the web is a graph, a Triplestore is type of
graph database.  Moreover, the graph collected by a Triplestore is
occasionally referred to as a _Knowledge Graph_.  A Triplestore is a
concrete implementation of the RDF and related standards in a database
form while a Knowledge Graph is a theoretical construct which
describes various means and methods of representing knowledge in a
computer.  These terms are often used interchangeably and can cause
confusion if a reader is unfamiliar with their use.

## Triplestores

While there are many different Triplestores available, including free
open source version such as [Apache
Fuseki](https://jena.apache.org/documentation/fuseki2/), there are two
commercial Triplestores which the curators have found to work
particularly well with IrishGen:
[GraphDB](http://graphdb.ontotext.com/) by
[Ontotext](https://www.ontotext.com/) and
[Stardog](https://www.stardog.com/).  While they are commercial
products, Ontotext produces a [free
version](https://www.ontotext.com/products/graphdb/graphdb-free/)
which has a limited but still fully usable and very useful set of
features and Stardog has a very generous 60 day [limited
trial](https://www.stardog.com/get-started/), which can be renewed
with a new download, generally a new and updated version of the
product.  Triplestores are generally server based systems which tend
to be very resource intensive; however, they can be run on a local
computer and be useful.  From here, Stardog and GraphDB will be the
reference point for Triplestores.

Triplestores can ingest the various RDF seraliziations into its store,
which are often called _repositories_.  Repositories store a set of
triples which are logically connected.  In reality, it entirely up to
the user to decide what "logically connected" may mean.  The point is
that Triplestores can store more than one large set of triples at once
much like the more common Relational Database Management System
(RDBMS).

While downloading and installing Triplestores on a user's machine is
outside the scope of this post, discussing the ease of use and various
ingestion and inference strategies of the two Triplestores will help
guide the reader to the Triplestore that best fits their particular
situation.  The two Triplestores have very different approaches to
ingesting RDF into their respective repositories.  GraphDB takes a
much more graphical approach where the user can indicate multiple
local files at a time to be ingested (see [Importing local
files](http://graphdb.ontotext.com/documentation/standard/loading-data-using-the-workbench.html#importing-local-files)).
On the other hand, Stardog takes a much more traditional command line
driven approach with the `stardog data add` command (see [Command Line
Interface](https://www.stardog.com/docs#_command_line_interface)).

## OWL 2 and Logical Inference (Reasoning)

As briefly discussed in ["IrishGen: RDF, Linked Data, and The Semantic
Web"]({% post_url 2020-06-17-IrishGen-RDF-Linked-Data-Semantic-Web
%}), [OWL 2](https://www.w3.org/TR/owl2-overview/) allows for the
creation of new RDF predicates.  This section will take an example
from IrishGen itself to demonstrate the basics of using OWL 2 to
define new RDF predicates.  The proper creation and curation of
ontologies is a specific discipline within Computer Science called
[Ontology
Engineering](https://doi.org/10.1007/978-0-387-39940-9_1315); however,
the basics are easily grasped and while the reader may never have
occasion to create their own ontology, it is instructive to have an
understanding of the mechanics of ontology construction so that
ontologies and logical inference that they allow may be used to their
full potential.

Within the genealogies for instance, population groups are often
referenced thus with this example coming from the the [Book of
Leinster](https://www.vanhamel.nl/codecs/Best,_et_al._1954-1983) (LL)
genealogical item [Genelach H-Úa
Lomthuile](https://celt.ucc.ie/published/G800011F/text002.html):

```
Laidcgnen a quo h-Úi Buide.
```

```
Laidcgnen by which Úi Buide.
```

The way in which this is reflected in IrishGen is [thus](https://github.com/cyocum/irish-gen/blob/master/LL/genelach_h-%C3%BAa_lomthuile.trig):

```turtle
<#Laidcnén>
    a foaf:Person;
    irishRel:nomName "Laidcnén";
    rel:childOf <#Cummine>;
    irishRel:ancestorOfGroup <#h-ÚiBuide>.

<#h-ÚiBuide>
    a irishRel:PopulationGroup ;
    irishRel:populationGroupName "h-Úi Buide" .
```

The two triples that define the Úi Buide population group in RDF is
what is of most interest as it shows the two kinds of things that OWL
2 can do: first, define a _Class_ and a predicate.

The `PopulationGroup` class is defined in a turtle file,
[earlyIrishRelationships.ttl](https://github.com/cyocum/irish-gen/blob/master/earlyIrishRelationship.ttl).
One of the more notable features of OWL is that it can be defined in
the same RDF structures that the data is itself encoded in.  To define
`PopulationGroup` as an [OWL
Class](https://www.w3.org/TR/2012/REC-owl2-primer-20121211/#Classes_and_Instances)
is done thus:

```turtle
:PopulationGroup a owl:Class .
```

That is it.  In the context of a Triplestore which implements OWL 2
the URL
`http://example.com/earlyIrishRelationship.ttl#PopulationGroup` now
has a meaning.  In this case the statement `<#h-ÚiBuide> a
irishRel:PopulationGroup` means that the URL `<#h-ÚiBuide>` is an
instance of the class of all
`http://example.com/earlyIrishRelationship.ttl#PopulationGroup`s.
This may not be particularly interesting at the moment but it will be
come more apparent as more interlocking features of OWL 2 are
introduced.

```turtle
:populationGroupName
    a owl:DatatypeProperty ;
    rdfs:domain :PopulationGroup ;
    rdfs:range xsd:string .
```

Moving to the second triple `<#h-ÚiBuide> irishRel:populationGroupName
"h-Úi Buide"`, the OWL 2 definition above defines the predicate
`populationGroupName`.  These predicates come in two forms [Object
Property](https://www.w3.org/TR/2012/REC-owl2-primer-20121211/#Object_Properties)
and [Datatype
Property](https://www.w3.org/TR/2012/REC-owl2-primer-20121211/#Datatypes).
The difference between these things are subtle but interesting.
Object properties involve URLs on both sides of the triple's
predicate.  Datatype properties involve _literals_ in the object
position of the predicate and a URL on the subject side of the
predicate.  What is meant by a literal is either a number like `42` or
a string like `"h-Úi Buide"`.  For the avoidance of doubt, literals
cannot be the subject of a triple.  In all cases, literals are defined
in the [XML Schema
Definition](https://www.w3.org/TR/xmlschema11-2/#built-in-datatypes)
specification (hence, the `xsd` prefix).

`rdfs:domain` and `rdfs:range` are slightly obscurely labelled.
`rdfs:domain` informs the system that the URL in the subject position
use with the defined predicate is a member of the specified class.  In
this case, the URL in the subject position should be marked as the
class `PopulationGroup`.  `rdfs:range` informs the system that the URL
in the object position is a member of the specified class.  In this
case, `xsd:string`.

The effect of all the foregoing is that if a URL is used with a
predicate, a Triplestore that implements OWL 2 will _infer_ that the
URL belongs to a particular class without further intervention from a
user.  Thus, using the turtle abbreviation `a` is often not necessary
as the Triplestore will infer the type based on its use in the RDF.
This kind of inference is often called _reasoning_ and a system that
implements the logical system of OWL 2 is called a _reasoner`_.  There
are several reasoners that are independent of a Triplestore and can be
used without a storage system such as
[Fact++](https://doi.org/10.1007/11814771_26),
[HermiT](http://www.hermit-reasoner.com/), and Pellet (which happens
to be the reasoner used by Stardog).

```turtle
<#h-ÚiBuide> irishRel:populationGroupName "h-Úi Buide".
```

Looking again at the above triple, the effect of the OWL 2 definition
is that a Triplestore's reasoner will infer that `<#h-ÚiBuide>` is a
`PopulationGroup` and infer that `"h-Úi Buide"` is a string.  A note
of caution is appropriate at this point.  While a Triplestore with a
reasoner will infer, it will _not_ enforce.  Thus, for instance if a
user puts a number literal, like _42_, into the object position in the
above triple, a reasoner will correctly ingest this and not inform the
user.  Enforcement or validation of triples can be done using the
[Shapes Constraint Language (SHACL)](https://www.w3.org/TR/shacl/)
specification if the Triplestore supports it.  Additionally, if two
predicates define different classes for their respective URLs, a user
can end up in a position where a reasoner infers a `Person` class and
a `Cat` class on the same URL, for instance, which will confuse a user
who expecting a `Person` and a `Cat` to be distinct individuals.  The
curators need to be careful when using predicates to ensure they do
not cause odd or confusing inferences to be made.

However, there are even more powerful features of OWL 2.  Predicates
that can be defined as
[_transitive_](https://www.w3.org/TR/2012/REC-owl2-primer-20121211/#Property_Characteristics)
in OWL which means that if a predicate connects A and B while B uses
the same predicate to connect B to C then A has the same effect as if
A also connected to C.

An instance from IrishGen should make this more clear.

```turtle
<#Laidcnén>
    a foaf:Person;
    irishRel:nomName "Laidcnén";
    rel:childOf <#Cummine>;
    irishRel:ancestorOfGroup <#h-ÚiBuide>.
```

IrishGen heavily uses the Relationship ontology which defines
`rel:parentOf` and `rel:childOf`.  These are defined as a
`rdf:subPropertyOf` `rel:ancestorOf` and `rel:descendantOf`.
`rel:ancestor` and `rel:descendantOf` are themselves defined as
transitive and the inverse of each other.  Summing all of that up, it
means that a reasoner can completely reconstruct a genealogical line
without human intervention.  An instance of this will be illustrative.
Without needing to encode by hand Laidcnén has 2301 ancestors.

This is incredibly powerful.  It means that only a minimum of
information needs to be encoded or curated by a human and the benefit
of inferring new information about existing information in the
Triplestore.  This is completely unlike a relational database which
needs all the information encoded or inferred by logic by computer
systems written by humans.

This singular feature of Linked Data was the driver behind the
decision to use Triplestores and RDF rather than conventional
relational or other graph databases.  Massive amounts of human power
would be necessary to encode all the information not only directly
stated but also logically implied by the genealogies.  That level of
data extraction and encoding by team of humans who are experts in the
medieval Irish genealogical tradition, even a large team assisted by
specialised technology and automation, would be prohibitively
expensive.  In essence, any assisting automation would by necessity
reinvent Linked Data rather than just taking advantage of an already
existing technology.

## Triplestores and RDF strategies

Now that the reader is acquainted with inferrencing, it would be good
to discuss inferrencing strategies.  While this may seem like a very
esoteric topic, it is critical to the experience of writing and
running queries on Triplestores and will effect the perception of
usefulness of Triplestores in general.

There are two broad categories of inferrencing strategies (see
[Artificial Intelligence: A Modern
Approach](http://www.worldcat.org/oclc/1021874142), pp. 208-385 for a
discussion and for a mathematically complete description of both
approaches see [Graph based knowledge representation: computational
foundations of conceptual
graphs](http://www.worldcat.org/oclc/297545606), pp. 278-301).  The
first is called _forward-chaining_ which is implemented by GraphDB. In
this strategy, when each assertion, in this case each triple, is added
to the Triplestore, the ontologies loaded will be consulted and any
triples that would be created by the logical rules set by the ontology
are created as are any other logical implications.  This means that
all inferrencing is done at the time that the triple is added to the
database.  The second is called _backwards-chaining_ which is
implemented by Stardog.  In this strategy, the system uses a user
query as a goal and works backwards from that goal.  If the goal cannot
be logically inferred from the set of triples and the ontologies
present in the Triplestore, nothing is returned.  Backwards-chaining
is the inverse of forward-chaining.

These two strategies have a large effect on how a user will perceive a
system when querying it.  In the forward-chaining strategy, all
inferences are done at the time that the files are added to the
Triplestore.  This means that the repository on disk can be very large
relative to the RDF files ingested.  For instance, at the time of
writing, GraphDB's on disk representation of IrishGen, using
forward-chaining, is 2.1 GB compared to the 4.4 MB size of the RDF
files.  On the other hand, querying this database is very fast as all
inferences are computed before a query is run.  Additionally,
ingesting the RDF files can take a long time as the system will need
to work out all the inferences, if any, for each triple.  In the
backwards-chaining strategy, since the inferrencing is deferred until
the time of the query, the on disk size is very small.  For instance
in Stardog, the on disk representation of IrishGen is 81.3 MB.  The
difference for users is felt in the speed of querying either of these
Triplestores.  In Stardog, queries that may seem very straight forward
can take an seemingly inordinate amount of time to complete.  However,
this makes adding or deleting triples from the database very fast as
the Stardog will not need to find and remove inferred statements.

Which of these two systems the reader chooses to use is a matter of
personal preference.  Both systems implement much of the standards,
although each has its own details that the reader will need to
understand while using it.  GraphDB is better if the user has the disk
space to hold all the inferred triples and wishes to have very fast
queries.  Stardog is better if the user wants more strict compliance
with the standards and fast loading of IrishGen files, but queries can
take a very long time to complete.

## Conclusion

The foregoing discussion has introduced the reader to Triplestores, a
kind of graph database which is used to store and query RDF.  This led
to a discussion of OWL 2 logical inferrencing and topics that a user
who wants to write queries will need to know before attempting to use
querying in Triplestores.  OWL 2 in conjunction with RDF gives a
powerful set of tools with which to model the medieval Irish
genealogical tradition.  The ability of OWL 2 to logically reason
about the data to create new data substantially expands and transforms
that data in ways that a team of humans who are experts in the
medieval Irish genealogies would not be able to do affordably.

While the actual act of querying will be deferred until a future
discussion, the reader will now be equipped to understand the whole
context for querying the IrishGen before handling SPARQL specifically.
