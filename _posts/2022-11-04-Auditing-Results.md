---
layout: post
date: 2022-11-04
title: "Reasoner Accountability"
author: cgy
---

{% assign author = site.data.authors[page.author]%}
{{author.name}}

ORCID: <a href="https://orcid.org/{{ author.orcid }}" title="{{author.name}}">{{author.orcid}}</a>

I wrote an
[email](https://lists.w3.org/Archives/Public/semantic-web/2022Jul/0017.html)
to the Semantic Web email list in July 2022 on this topic.  While I
received some interesting responses, it did not stir the kind of
interesting discussion that I had hoped for.

One of the basic problems of the Semantic Web is accountability.  In a
world where anyone can say anything about facts, it is hard to discern
what the output of the system should be.  This was discussed by Cory
Doctorow and described as
[metacrap](https://chnm.gmu.edu/digitalhistory/links/pdf/preserving/8_17.pdf).  In
terms of IrishGen, this is less of a concern because of two things:
first, the type of information that is encoded is bounded by its
subject matter; second, the statements can be traced back to their
origin information in the various Irish manuscripts by the more
intrepid individual.  However, there is one weak link in this:
reasoner accountability.

There are a couple of aspects of this.  First, as IrishGen is a human
curated database, human errors occur.  Catching these and correcting
them is a part of ensuring that everyone has confidence that the
information translated from primary sources is faithful to them.
Second, catching coding errors in the reasoner so that they can also
be corrected.  When reasoners are used as heavily as they are in
IrishGen, the question of accountability becomes critical.

Of the two systems that I have the most experience with (Stardog and
GraphDB), Stardog does have the ["Explain Reasoning
Results"](https://docs.stardog.com/inference-engine/#explaining-reasoning-results)
capability.  However, this does not work when the question is: what
path did the reasoner take to obtain this particular result?  The
Stardog reasoning explain system will only explain what inferences
were done for particular classes.  The Stardog reasoning system will
not explain triples that it has created while evaluating the query
(for more on forward and backwards chaining reasoners see
[Triplestores, Ontologies, and Reasoning]({%post_url
2020-06-22-Triplestores-Onotologies-Reasoning %}).  GraphDB provides
two plug-ins for this: [Proof and
Provenance](https://graphdb.ontotext.com/documentation/10.1/inference.html).
I have not yet had a chance to use these as they seem to have been
added in the GraphDB 10 version of the system.  When I have some more
time, I will have a chance to use these and see if they do what I hope
that they do.

However, the overriding point of this post is that for users who are
more sceptical of systems like IrishGen, auditability of the
reasoner's choices is critical to convincing them that these systems
can be held to account in the decisions that are made.  Additionally,
auditability allows curators to investigate any anomalous results and
demonstrate that the reasoner has created a sound interpretation of
the data and how it has derived its results.  Hopefully, more Semantic
Web systems will start having these kinds of plugins so that users can
interrogate the integrity of their systems.

