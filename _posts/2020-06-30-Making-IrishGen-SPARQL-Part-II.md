---
layout: post
date: 2020-06-30
title: "Making IrishGen SPARQL Part II: Constructs"
author: cgy
---

{% assign author = site.data.authors[page.author]%}
{{author.name}}

ORCID: <a href="https://orcid.org/{{ author.orcid }}" title="{{author.name}}">{{author.orcid}}</a>

The first post in this series covered the `select` query form and used
Baeth as a guide.  We will continue our sojourn through SPARQL with
Baeth in this next installment where the `construct` query form will
be introduced and discussed.

Simply but opaquely stated, the `construct` query form takes one
sub-graph and returns a new sub-graph which is transformed from the
original.  This means that you can search the dataset for a particular
pattern then transform it into another pattern that the user supplies.
While this might not sound very useful for IrishGen and, by extension,
Baeth but what it allows the user to do is to extract interesting and
useful parts of the overall dataset such as Baeth's children or, even
more interestingly, Baeth's genealogy as reconstructed by the
reasoner.

We will start by examining Baeth's children.  In this, we will need to
choose a particular Baeth.  Using the query below:

```sparql 
prefix foaf:  <http://xmlns.com/foaf/0.1/>
prefix rel: <http://purl.org/vocab/relationship/>

select ?x (count(distinct ?child) as ?children)
from <tag:stardog:api:context:all>
where {
	?x foaf:name ?y;
        rel:parentOf ?child
    filter regex(?y, "^B[áa]eth$", "i")
}
group by ?x
order by desc(?children)
```

We discover that
`http://example.com/Rawl_B502/clann_aingeda.trig#Báeth` has 12
children.  This will be enough to demonstrate the basic construct
facility.  The basic construct query below will return all twelve of
these and Stardog Studio will render the graph into a nice visual
format.

```sparql
prefix rel: <http://purl.org/vocab/relationship/>

construct {
    <http://example.com/Rawl_B502/clann_aingeda.trig#Báeth> rel:parentOf ?child
} 
from <tag:stardog:api:context:all>
where {
    <http://example.com/Rawl_B502/clann_aingeda.trig#Báeth> rel:parentOf ?child
}
```

`construct` queries generally have have two bodies.  After the
`construct` keyword which is the target RDF the user wishes to
create. The `from` keyword here serves a similar purpose as it does in
the `select` query form and denotes that the user is interested in
pulling the entire graph into the default graph or more simply, that
the entire dataset should be considered when running the query.  The
`where` keyword denotes the pattern that the Triplestore should match.
In this case, the pattern is the same as the RDF that the `construct`
clause constructs.  The effect of this is to excise the graph of all
of Baeth's children from the entire graph of all of IrishGen.  This
makes it possible to visualise.  It is no use to attempt to visualise
the entire graph as a user would most likely get lost or confused.
Extracting and visualizing sub-graphs allows the user to target
persons of interest and their genealogical information without
becoming overloaded by extraneous information.  Additionally,
attempting to render visualizations of vast graphs is a huge
computational task and most user's machines would not be powerful
enough to do it.

As an aside, the above query can be written more concisely by using
the single body form of `construct`, which can be useful when the user
wishes to have a small subset extracted from the graph then used in
visualization:

```sparql
prefix rel: <http://purl.org/vocab/relationship/>

construct 
from <tag:stardog:api:context:all>
where {
    <http://example.com/Rawl_B502/clann_aingeda.trig#Báeth> rel:parentOf ?child
}
```

The visualization that is returned by Stardog Studio is below.

<img src="{{site.baseurl}}/assets/images/construct_baeth_1.png" />

As one can see, this produces a star-like structure of children around
Baeth.  At this point one can start exploring the graph by
right-clicking on the nodes and choosing "Expand from node".  However,
in Stardog Studio there is a problem where unless the "Query All
Graphs" property is set to true in Stardog, it will not expand.
Briefly, GraphDB has "Query All Graphs" essentially on by default and
is not a property that the user can set.  Stardog has the property off
by default which can cause confusion when a user is encountering both
Triplestores.  A second and more pressing problem is that expanding
nodes in the face of reasoning can cause a query that a user's laptop
cannot finish in a reasonable time or finish at all in the face of the
query timeout restrictions.  In Stardog, this can be overcome by
turning reasoning off when doing an exploration of the graph if the
query does not return in a reasonable time but this limits what is
possible.  It is the user's discretion to understand what their query
involves and what compromises they are willing to tolerate.

With reasoning turned off, one can explore the graph a bit at a time
as one can see below:

<img src="{{site.baseurl}}/assets/images/construct_baeth_2.png" />

A much more ambitious graph to construct is the graph of Baeth's
ancestors or to reconstruct his geneaological line from the
Triplestore.  This query introduces a few new constructs which are
handy when working with SPARQL.

```sparql
prefix rel: <http://purl.org/vocab/relationship/>

construct {
    ?s rel:childOf ?parent.
    ?y rel:parentOf ?baeth.
} where {
    values ?baeth {<http://example.com/Rawl_B502/clann_aingeda.trig#Báeth> }

    {
      ?s rel:ancestorOf ?baeth;
         rel:childOf ?parent
    } union {
      ?y rel:parentOf ?baeth
    }
}
```

The `constuct` form first shows what will be the product of the query.
The `values` keyword is used to store static URLs which will be used
repeatedly within the query.  This cuts down on visual complexity
overall of the query.  The next part of the query is the meat of the
process.  This informally says: "return all people who are the
ancestors of Baeth and their parent-child relationship and return the
parent of Baeth as well".  These are then reconstructed in the
`construct` form to create the reconstructed genealogy of Baeth.  The
`union` is used to get Baeth themselves attached to the output through
their parent which makes it slightly easier to interpret the
visualization.

<img src="{{site.baseurl}}/assets/images/construct_baeth_3.png" />

One will note that the geneaology only goes back to Imchada.  This is
due to the fact that Imchada is not linked to any other part of the
genealogies.  The reason for this is outside the scope of this post
but could be as simple as a missing `owl:sameAs` link from elsewhere.
The curators tend to be conservative and if there is no clear evidence
that two individuals are the same, the policy is to leave them as is.
Finally, a note about performance.  This query took 21000 milliseconds
or about 21 seconds to run.  Reconstucting this by hand would take
much longer and creating a visualization even longer still.  Resulting
in an interactive visualization is just one more additional benefit.

As we can see, `construct` queries allow the transformation of RDF
graphs found in the Triplstore into other RDF graphs.  Using this, we
can extract interesting sub-graphs from the dataset and visualize them
using Stardog Studio.  This allows the reconstruction of geneaological
lines and visual exploration of the data that is latent in the dataset
through a Triplestore's reasoning capabilities.
