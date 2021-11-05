---
layout: post
date: 2021-11-05
title: Individuals vs Instances
author: cgy
---

{% assign author = site.data.authors[page.author]%}
{{author.name}}

ORCID: <a href="https://orcid.org/{{ author.orcid }}" title="{{author.name}}">{{author.orcid}}</a>

When reading the Irish Genealogies, you will often encounter situations like this in _Genelach Clainde Conchobuir Corraig_:

> ...o shléib fodess Fiacra Tort.

Followed by further down the page by:

> Fiachra Tort secht mc leis...

In this case, both names indicate the same individual but there are
two instances of their name in the same section of text.  There are
two strategies when modelling this in a database.  First, you can wrap
all the information into the same instance in the database.  This
makes sense as they are indeed the same individual and should be
treated the same.  However, the database does not now follow the text
that it is modelling and it will take a reader a little while to work
out what has happened.  Additionally, there is the possibility,
admittedly remote, that a repeated name in a genealogy will get
confused for one another and, when fixing it, the auditor will need to
disentangle the two individuals.  Second, the database can have two
separate entries for each instance and tie them together somehow.  In
this method, a reader and more easily follow the flow of the text in
the original source material.  Additionally, a direct link can be made
to the instance that a reader or user wishes to reference.

In the case of IrishGen, a reader will encounter both strategies
employed.  This is due to the transition from the text to RDF is
undertaking in only a semi-automated fashion.  This mean there is
always a human in the process.  While this is usually beneficial, it
can also mean that inconsistencies like this can happen.  To be more
honest, it happens when I am being lazy doing the translation.  Most
times this is not a problem as all the information in the source
material is represented and can be searched.

In a traditional relational database, it can be slightly difficult to
represent the second strategy.  Usually, it would mean creating a
second "link table" which would then have all the links between the
two entities in the originating table.  This can often turn into a
many-to-many reference situation.  Again, while it is not unusual to
do something like this in a relational database, it is just slightly
awkward and creates odd looking SQL queries.

In a graph database like IrishGen and RDF, the situation becomes
easier to represent.  It is just another edge and link to other
instances of an individual.  In the case above, this is represented
like so:

```
     <FiacraTort>
        a foaf:Person;
        irishRel:nomName "Fiacra Tort";
        agrelon:hasParent <CollaUais> ;
        rel:childOf <CollaUais>;
        irishRel:ancestorOfGroup <hTurtri>, <FirLi>, <hMeicUais>.

...

     <FiachraTort-269b781f>
        a foaf:Person ;
        irishRel:nomName "Fiachra Tort" ;
        owl:sameAs <FiacraTort> ;
        irishRel:numChild 7 .
```

You will notice that there is a `owl:sameAs` edge which links one
instance to another.  This is where the situation becomes slightly
complicated.  There is the concept of "Fiacra Tort" as a single
individual who is referenced twice in this text.  There are also two
_instances_ of the same individual in the text.  To represent this
situation, OWL 2 has the `sameAs` predicate.  This is also known as
the `SameIndividual` predicate (see OWL 2 Primer [4.7 Equality and
Inequality of
Individuals](https://www.w3.org/TR/2012/REC-owl2-primer-20121211/#Equality_and_Inequality_of_Individuals)).
What this does in most Triplestores is to merge the two nodes and
present information from both of them when they are searched.  This is
usually what a user will want.  Although, occasionally it is not so
most Triplestores will also allow the `sameAs` predicate semantics to
be disabled.

This distinction between individual and instance may seem like hair
splitting but it allows IrishGen to reference both the idea of an
individual when searching and the ability to represent more closely
the text as it stands in the original source material which eases
auditing.
