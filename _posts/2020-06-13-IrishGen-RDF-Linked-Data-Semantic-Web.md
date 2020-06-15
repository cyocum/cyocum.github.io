---
layout: post
date: 2020-06-13
title: "IrishGen: RDF, Linked Data, and The Semantic Web"
author: cgy
---

{% assign author = site.data.authors[page.author]%}
{{author.name}}

ORCID: <a href="https://orcid.org/{{ author.orcid }}" title="{{author.name}}">{{author.orcid}}</a>

## Introduction

This post intends to cover many of the underlying technical
specifications and formats that IrishGen uses.  This will help orient
readers to the IrishGen dataset and the technical environment which it
inhabits.  Additionally, a set of common specific errors that a user
or a contributor may encounter in the dataset will be explored.  A
careful reading of this will aid the reader who wants to use IrishGen
in their own research and study. The audience for this post should be
more familiar with the structure of the early Irish verb, for
instance, than the structure of a file format.

## Semantic Web and Linked Data

Before we begin, it would be beneficial to introduce two general terms
that will be important if the reader wishes to explore the wider
context within which IrishGen sits: _Semantic Web_ and _Linked Data_.
These terms are basically interchangeable and the reader will see both
elsewhere in the literature.  A history of the terms can be found in
"[Whatever Happened to the Semantic
Web?](https://twobithistory.org/2018/05/27/semantic-web.html)".

## Resource Description Framework

The Resource Description Framework (RDF) is a
[standard](https://www.w3.org/TR/2014/REC-rdf11-concepts-20140225/)
defined by the [World Wide Web Consortium](https://www.w3.org) which
is meant to represent data on the World Wide Web.  The RDF
specification sets the theoretical underpinning for data on the web
and thus the way that data is handled in IrishGen.  The form and
format of IrishGen all flow from this one specification.

RDF defines a particular kind of graph database which is also known as
a Knowledge Graph (see "[Knowledge
Graphs](https://arxiv.org/abs/2003.02320)" for an extremely
comprehensive and detailed introduction to the topic).  The structure
of RDF is relatively simple with only three active elements: the
subject, the predicate, and the object.  These three elements are
defined and combined in different ways to create a flexible way to
represent data.  One of the more odd features of RDF is that
everything is a [URL](https://url.spec.whatwg.org/), which can cause
initial confusion.

While RDF is a theoretical construct, a means of expressing this
consruct is necessary.  For this, several file formats (also known as
"serializations") which conform to the framework were created.  Among
the most common are:
[RDF/XML](https://www.w3.org/TR/rdf-syntax-grammar/),
[Notation3](https://www.w3.org/TeamSubmission/n3/),
[Turtle](https://www.w3.org/TR/turtle/),
[TRiG](https://www.w3.org/TR/trig/), and
[JSON-LD](https://www.w3.org/TR/json-ld/).  The curators chose TRiG as
the file format but it can be automatically translated into any of the
other formats as needed.

## RDF Formats and IrishGen: An Example

An example in Notation3 format which is [adapted from
IrishGen](https://github.com/cyocum/irish-gen/blob/master/LL/dal_corpri_arad.trig)
will help to see what the above is in practice:

```turtle
<http://example.com/LL/dal_corpri_arad.trig#Flaithbertach> 
<http://www.w3.org/1999/02/22-rdf-syntax-ns#type> 
<http://xmlns.com/foaf/0.1/Person> 
```

A small note on formatting is necessary before beginning: URLs are
always surrounded by `<` and `>` to distinguish them from other forms
of text that may appear in the file.

The first line is the subject of the RDF statement.  If one thinks of
this as a sentence in a SVO language, the first URL is in the subject
position.  This is the URL about which the statement makes an
assertion (for a more formal defintion of terms like TBox and ABox,
see [Handbook of Knowledge
Representation](http://www.worldcat.org/oclc/968676609) ).  The
subject can be any URL, even one that you cannot dereference, as in
the instance above `http://example.com` is a dummy URL, which the
curators chose as they do not have the resources to maintain a website
at the current time.  The reason that this works is due to RDF's Open
World Assumption (see [Artificial Intelligence: A Modern
Approach](http://www.worldcat.org/oclc/1021874142), pp. 208-385) which
means that a RDF aware computer system reading this will assume that
the URL `http://example.com/LL/dal_corpri_arad.trig#Flaithbertach`
exists irrespective of its availability on the web at the time the
system becomes aware of it.  This can cause some problems which will
be covered at the end of this post.

The second line is the predicate of the RDF statement.  Much thought
and discussion goes into this element of a triple as it contains within
it the ability to _reason_ about the graph using first order logic
(see [Artificial
Intelligence](http://www.worldcat.org/oclc/1021874142), pp. 251-313).
Reasoning in terms of RDF will be more fully explored in a future post
but, to anticipate, the ability to reason about the graph was a
decisive factor when choosing to use RDF over any other graph database
technology.

The third line is the object of the RDF statement.  This is the effect
that applying the predicate will have on the subject.  In some cases
this can be as simple as giving the subject a static label (such as
giving the proper nominative of their forename) or in more complex
cases can involve other URLs elsewhere defined on the web.

While the triple as it is called is a flexible and powerful method to
define information, there is one thing missing: logical grouping of
triples.  This is accomplished with the "quad" which adds an extra
section to the triple:

```turtle
<http://example.com/LL/dal_corpri_arad.trig#Flaithbertach> 
<http://www.w3.org/1999/02/22-rdf-syntax-ns#type> 
<http://xmlns.com/foaf/0.1/Person> 
<http://example.com/LL>
```

This is also known as a [RDF
Dataset](https://www.w3.org/TR/rdf11-concepts/#section-dataset) of
which TRiG is the seralisation and itself a small extension of the
Turtal format.  In IrishGen, this is used to organize triples by the
MS in which the triple appears and allows queries to target just
certain MS rather than the entire dataset at once.

A note on the URLs used in IrishGen for individuals who appear in the
genealogies will assist the reader at this point.  When a curator is
translating (manually or by automation) a geneaology to create Linked
Data, a new URL must be coined for each individual instance of a name
which appears in the source material.  This does not give that
individual any information beyond what is declared.  Thus, for
instance, informally we may say that "In LL, Flaithbertach is a
person" but it is more formally stated as "In the graph pointed to by
the URL <http://exmaple.com/LL>, the URL
<http://example.com/LL/dal_corpri_arad.trig#Flaithbertach> exists and
is declared to be an RDF class <http://xmlns.com/foaf/0.1/Person>",
which is far more combersome way of stating the facts.  They are, in
effect, a nameless person.  This is why the reader will see names
explicitly attached to a URL.  One of the additional benefits of this
is that different forms of a name can be recorded without needing to
coin a new URL for each.

Moreover, an individual's URL also be read.  First, the
`http://example.com` can be ignored as it is just a way for accounting
for a base URL that is needed for the system as a whole to work.  The
second element `LL` denotes the MS that holds the information
described, which is in addition to the quad URL which also denoates
MS.  A URL is just a string with a certain format to a Linked Data
system and thus the fourth element of a quad is added to allow the
system to manipulate things at the level of a MS. Then, the second
element is the directory which holds the TRiG file in the [Git
repository](https://github.com/cyocum/irish-gen/tree/master/LL).  The
final element is a slightly reformatted version of the title of the
_item_ which holds the individual (for the item structure of the
genealogies, see Holmberg, '[Towards a Relative Chronology of the
Milesian Genealogical
Scheme](https://dash.harvard.edu/handle/1/37945002)', pp. 17-18).  The
data, of course, is inside the file.  This is useful when searching as
many systems will give the full URL for what they find and the user
can track down the exact item that the information appears in.

With the formalities of an individual's URL out of the way, the reader
can see the above Notation3 would be very cumbersome for humans to
either create or read.  Thus, Notation3 is not often used in practice.
A more human readable format is Turtle of which TRiG, already
mentioned, is a slight extension.  For example, the above can be
written:

```turtle
@base <http://example.com/LL/dal_corpri_arad.trig> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

<http://example.com/LL> {
  <#Flaithbertach> a foaf:Person 
}
```

This is a much condensed and in a way easier to read version of the
above.  Turtle (and by extension TRiG) condenses the format by moving
common elements to the beginning of the file with shortened prefixes
to be reused elsewhere and giving sensible defaults for very commonly
used elements.  In the above, the URL which defines Flaithbertach to
the system is pulled mostly into the `@base` which means that any URL
fragment will be given this full URL when read by the system.  The URL
`<http://www.w3.org/1999/02/22-rdf-syntax-ns#type> ` is merely given a
default expression as `a` due to the fact that it is very common.
`foaf:` is declared as a shortened prefix at the top of the file and
reused.  In IrishGen, there are many people as they make up the
majority of the data so keeping it short is good practice.


## Ontologies and IrishGen

At this point a reader may wonder where predicates come from, which is
an entirely reasonable subject to consider at this point.  Predicates
may come from one of two sources.  The first is the RDF specification
itself which defines a few, general purpose predicates.  The other,
and far more complex, source is two specifications: [RDF
Schema](https://www.w3.org/TR/rdf-schema/) (RDFS) and the Web Ontology
Language (OWL), which has two versions (see [Web Ontology Language
Reference](https://www.w3.org/TR/owl-ref/) for version one and [OWL 2
Web Ontology Language Document Overview (Second
Edition)](https://www.w3.org/TR/owl2-overview/) for version two).  For
all practical purposes, OWL 2 is the _de facto_ source for all
predicates in IrishGen even if they are declared with an RDFS URL.

The curators have modified various publically avaliable ontologies and
created a few of their own to more accurately model the situation in
the medieval Irish genealogical source material.  For instance,
 the curators created the idea of `PopulationGroup` which are fairly
common entities in the geneaologies and the ability of persons to be
ancestors or decendants of these groups (for implementations of these
predicates and other predicates see the [Early Irish Relationships
Ontology](https://github.com/cyocum/irish-gen/blob/master/earlyIrishRelationship.ttl)).

Ontologies allow new predicates to be added and thus new things to be
expressed about data in a RDF dataset.

A brief aside concerning databases will be useful at this point.
While all of the above deals with file formats and defining data,
actually searching that data is notibly absent.  Searching collections
of RDF datasets involves the use of a database technology called a
Triplestore.  There are many open source and commercial Triplestores
in use but IrishGen is generally used with
[GraphDB](https://www.ontotext.com/products/graphdb/) or
[Stardog](https://www.stardog.com/).  Querying RDF is a deep and
complicated subject that will be reserved for another post where the
implications can be properly explored in depth.

The foregoing demonstrate in a microcosm how the curators of IrishGen
use RDF and its seralisations to organise medieval Irish geneaological
information.  This, of course, is just one small sample of the
information avaliable within IrishGen.  However, this should orient
and assist the reader when they encounter the IrishGen in its file
format.  The rest of this post will be devoted to the various kinds of
errors and difficulies that a reader and curator can encounter when
using the dataset.

## Difficulties and Challenges of Using RDF

Now the reader is aquainted with the various parts of RDF and its
ecosystem of terms.  The choices made do not come without cost and
without their own challenges.  This section will explore some of the
choices that IrishGen has encountered over the years and give some
solutions to those which may be of interest to readers.

Human error is the bane of may digital curation projects.  IrishGen is
no less effected.  While typos are generally an annoyance, in IrishGen
and RDF generally, they can cause entire systems to go awry.  A case
in point, a vast number of the entries in IrishGen have the form
below:

```turtle
<#Flaithbertach>
    a foaf:Person;
    irishRel:nomName "Flaithbertach";
    rel:childOf <#Crunmael>.
```

Informally, this states that Flaithbertach is a person who has the
name (in the nominative case) Flaithbertach and is the child of a
person denoted by the URL `<#Crunmael>`.  While automated generation
of this entry can help (as discussed in [Human Curation and Digital
Datasets: A Problem in Multiple
Parts]({{site.baseurl}}/2020/06/07/Human-Curation-and-Digital-Datasets.html)),
this still requires human intervention and thus the possibility of
error.  If an error is introduced of the form:

```
<#Flaithbertach>
    a foaf:Person;
    irishRel:nomName "Flaithbertach";
    rel:childOF <#Crunmael>.

```

Notice the difference between `childOf` and `childOF`.  This is subtle
and easily missed even during a full audit.  The effect of this change
is that the predicate is assumed to exist but it has no effect.
Essentially, this means that when placed in a Triplestore that can
apply reasoning to a triple, no reasoning will apply.  The implication
of this is that now Flaithbertach is not the child of Crunmael and
when searched for under its proper form, `childOf`, it will not
appear.  The system is broken and if this is in a complex query,
incorrect results will be returned to the user.  The old adage
"Garbage In, Garbage Out" but it is more serious because given the
complex nature of some of the queries, the data can look at first
sight as not garbage but still cause serious problems. For a user who
is not adept with computers and is not ready to expect this in a human
curated database, it can be very frustrating and confidence draining.

There is not much that can be done for this kind of problem overall.
More careful curators and automation can decrease the occurance as
discussed in "[Human Curation and Digital
Datasets]({{site.baseurl}}/2020/06/07/Human-Curation-and-Digital-Datasets.html)"
but this will never fully elimiate the possibility.

Another form of error can cause real problems in a system.  Take the
below more fleshed out version:

```turtle
@base <http://example.com/LL/dal_corpri_arad.trig> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .
@prefix rel: <http://purl.org/vocab/relationship/> .

<http://example.com/LL> {
  <#Flaithbertach> a foaf:Person ;
                   rel:childOf <#Crunmael>.
}
```

The above is a full TRiG file that will be parsed successfully by a
system that can parse the TRiG file format.  The file above translated
informally, taking into consideration as explained above that the URLs
are not themselves people with names, into English states: "In LL,
Flaithbertach is a person and is the child of Crunmael".  As the
astute reader will have noticed, Crunmael is nowhere mentioned.  Due
to the Open World Assumption, the existence of Crumael will be assumed
by the system.  In fact, in the face of reasoning and the `rel`
ontology, he will also be assumed to be a person.  While this can be
useful, it can also create "ghost individuals" who exist only because
a human made a mistake somewhere.  Additionally, if there are two
individuals with the same name in the same item and thus in the same
file with the same URL, a completely skewed graph can result because
the Triplestore will assume they are referring to the same person.
The curators avoid this siutation generally by appending a random
fragment of a universally unique identifier (UUID, for a formal
definition, see [RFC 4122](https://www.ietf.org/rfc/rfc4122.txt)) to
the URL to distinguish between two individuals with the same name.
However, occasionally the UUID fragment will be missed during the
creation of the URL and two seperate people will be accidentally
merged together, which is the inverse of the "ghost person" problem.

The most difficult and insidious form of error that can occur is the
`owl:sameAs` error.  One of the most useful things IrishGen can do is
to not only extract information and make it searchable but also
connect the same individuals and lines of descent across MSS.  The way
that IrishGen does this is by using the OWL 2 `sameAs` built-in
predicate.  This predicate is special as it has properties that others
do not (for a formal description of `owl:sameAs` as it pertains to its
special properties see [OWL 2 Web Ontology Language Profiles (Second
Edition)](https://www.w3.org/TR/2012/REC-owl2-profiles-20121211/)).
Suffice to say, it is a powerful predicate and can cause problems of
its own.  If an individual is mistakenly marked as `owl:sameAs`
another unrelated individual, this will cause a Triplestore to return
an incorrect graph and thus either confuse, in the best case, or
mislead, in the worst, a user.  Triplestores can [in some cases
explain
why](https://www.stardog.com/docs/#_explaining_reasoning_results) it
returned a graph of such a form but more often than not, no
explanation is avaliable and it is up to the user or curator examine
the string of reasoning that caused such a result.  This situation is
caused by the curators and their understanding and interpretation of
the source material and sometimes curators make mistakes or
misunderstand the material.  When this occurs, the dataset can be and
most likely will be amended, unlike a book such as [Corpus
Genealogiarum Hiberniae](http://www.worldcat.org/oclc/911333553).
However, this can also cause a user to lose confidence in the
integrity of the dataset.

There will always be the possibility of error in a dataset or
database.  Every person in the modern world has encountered a system
in which an error was introduced either by another human or by an
automated process which as gone awry.  These errors can cause anything
from minor annoyance while going about daily activities to
catastrophic life altering consequences.  There is, in effect, no
avoiding error; the possibility of error can only be minimized.  The
amount of minimization that can be done is effected by the
availability of resource and skill while curating the dataset either
in an automated or manual fashion.  In the case of IrishGen, resources
are very limited so user expectations should be set accordingly.

## Conclusion

This post intended to orient a reader who may not have experience in
file formats and the related standards to which IrishGen conforms and
prepare the reader to use IrishGen in their own research.
Additionally, some of the errors that may creep into the dataset due
to human error were explored to warn the reader about potential
pitfalls in the dataset itself.  This should give the reader the
confidence to begin to read the files in the dataset and to understand
the breadth and depth of information avaliable.  A future post will
explore the options for performing structured queries on the dataset
and how Triplestores work from a user's point of view, which will
equip the reader to appreciate what these technologies can do for
their own research.

