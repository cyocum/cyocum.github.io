---
layout: post
date: 2020-08-05
title: "Making IrishGen SPARQL Part II: Constructs"
author: cgy
---

{% assign author = site.data.authors[page.author]%}
{{author.name}}

ORCID: <a href="https://orcid.org/{{ author.orcid }}" title="{{author.name}}">{{author.orcid}}</a>

The first post in this series covered the `select` query form and used
individuals with the name Báeth as a guide.  We will continue our
sojourn through SPARQL with the list of individuals named Báeth
collected in the first post in this instalment where the `construct`
query form will be introduced and discussed.

Simply but opaquely stated, the `construct` query form takes one
sub-graph and returns a new sub-graph which is transformed from the
original.  This means that you can search the dataset for a particular
pattern then transform it into another pattern that the user supplies.
While this might not sound very useful for IrishGen and, by extension,
Báeth, what it allows the user to do is to extract interesting and
useful parts of the overall dataset such as Báeth's children or, even
more interestingly, Báeth's genealogy as reconstructed by the
reasoner.

We will start by examining Báeth's children so that we can show their
relationship to Báeth.  In this, we will need to choose a particular
Báeth, using the query below:

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

The above query (with reasoning enabled) returns 15 results as shown below:

| x                                                                              | children |
| http://example.com/Rawl_B502/clann_aingeda.trig#Báeth                          | 12       |
| http://example.com/Rawl_B502/genelach_úa_m-bairrche.trig#Bóeth                 |  2        |
| http://example.com/Rawl_B502/genelach_benntraige.trig#Báeth                    |   1       |
| http://example.com/LL/de_genelach_dail_messi_corbb.trig#Baeth-a245b020         |   1       |
| http://example.com/LL/genelach_h_n-enechglais.trig#Baeth-50d79733              |   1       |
| http://example.com/Rawl_B502/genelach_úa_m-bairrche.trig#Báeth                 |   1       |
| http://example.com/Rawl_B502/item_úi_máeli_rubae_la_laigniu.trig#Báeth         |   1       |
| http://example.com/Rawl_B502/genelach_ciannachta.trig#Báeth                    |   1       |
| http://example.com/LL/genelach_h_mugroin_i_m-maig_liphi.trig#Baeth             |   1       |
| http://example.com/Rawl_B502/genelach_ceníuil_báeth.trig#Báeth                 |   1       |
| http://example.com/LL/forthart_fea.trig#Baeth                                  |   1       |
| http://example.com/Rawl_B502/úib_luchta.trig#Báeth                             |   1       |
| http://example.com/Rawl_B502/do_primforslointib_Lagen_inso.trig#Báeth-604282d0 |   1       |
| http://example.com/Rawl_B502/genelach_úa_rossa_i_úa_n_dícolla.trig#Báeth       |   1       |
| http://example.com/LL/eoganachta_casil.trig#Buith                              |   1       |

We discover that
`http://example.com/Rawl_B502/clann_aingeda.trig#Báeth` has 12
children that are directly related to that URL, which means that they
are all described as his children in a particular text and in one
particular MS for which see the URL above, and who is well suited to
demonstrate the basic construct facility.  The construct query below
will return all twelve of these and Stardog Studio will render the
graph into a nice visual format.

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

`construct` queries generally have have two bodies.  The first is just
after the `construct` keyword.  This first body is the target RDF the
user wishes to create. The `from` keyword here serves a similar
purpose as it does in the `select` query form and denotes that the
user is interested in having the Triplestore consider the entire
dataset when running the query.  The `where` keyword denotes the
pattern that the Triplestore should match.  In this case, the pattern
is the same as the RDF that the `construct` clause constructs.  The
effect of this is to extract the sub-graph of all of Báeth's children
from the entire graph of all of IrishGen.  This makes it possible to
visualise.  It is no use attempting to visualise the entire graph as a
user would most likely get lost or confused.  Extracting and
visualising sub-graphs allows the user to target persons of interest
and their genealogical information without becoming overloaded by
extraneous information.  Additionally, attempting to render
visualisations of vast graphs is a huge computational task and most
user's machines would not be powerful enough to accomplish it.

As an aside, the above query can be written more concisely by using
the single body form of `construct`, which can be useful when the user
wishes to have a small subset extracted from the graph then used in
visualisation.  The single body form of `construct` can only be used
with the constructed graph pattern exactly matches the pattern used in
the `where` clause:

```sparql
prefix rel: <http://purl.org/vocab/relationship/>

construct 
from <tag:stardog:api:context:all>
where {
    <http://example.com/Rawl_B502/clann_aingeda.trig#Báeth> rel:parentOf ?child
}
```

The visualisation that is returned by Stardog Studio is below.

<img src="{{site.baseurl}}/assets/images/construct_baeth_1.png" />

As one can see, this produces a star-like structure of children around
Báeth.  At this point one can start exploring the wider graph by
right-clicking on the nodes and choosing "Expand from node", which
will pull from the entire graph (the construct query being just a
starting point).  However, in Stardog Studio there is a problem where
unless the "Query All Graphs" property is set to true in Stardog, it
will not expand.  Briefly, GraphDB has "Query All Graphs" essentially
on by default and is not a property that the user can set.  Stardog
has the property off by default which can cause confusion when a user
is encountering both Triplestores.  A second and more pressing problem
is that expanding nodes in the face of reasoning can cause a query
that a user's laptop cannot finish in a reasonable time or finish at
all in the face of the query timeout restrictions.  Although, in
larger server based installations, it would be possible.  In Stardog,
this can be overcome by turning reasoning off when doing an
exploration of the graph if the query does not return in a reasonable
time but this limits what is possible.  It is up to the user's
discretion to understand what their query involves and what
compromises they are willing to tolerate.

To set the Stardog parameter "Query All Graphs" to true in Stardog
Studio, the user will need to go to "Databases" then select their
database from the list of databases.  In the right-hand pane, the user
will need to choose "Properties".  In the search box directly below
the "Properties" header, type "query all".  This will cause the option
to appear and the user can then check the checkbox to enable "Query
All Graphs".

To contrast this with a `select` query that returns the same
information but in a textual format, the query below recreates the
`construct` query.

```sparql
prefix rel: <http://purl.org/vocab/relationship/>

select ?child
from <tag:stardog:api:context:all>
where {
    <http://example.com/Rawl_B502/clann_aingeda.trig#Báeth> rel:parentOf ?child
}
```

with the result being:

| child                                                    |
| http://example.com/Rawl_B502/úib_luchta.trig#Comdellach  |
| http://example.com/Rawl_B502/úib_luchta.trig#Fogartach   |
| http://example.com/Rawl_B502/úib_luchta.trig#Díummassach |
| http://example.com/Rawl_B502/úib_luchta.trig#Flann       |
| http://example.com/Rawl_B502/úib_luchta.trig#Flanngus    |
| http://example.com/Rawl_B502/úib_luchta.trig#Lassirne    |
| http://example.com/Rawl_B502/clann_eichlich.trig#Nárgusa |
| http://example.com/Rawl_B502/úib_luchta.trig#Aingid      |
| http://example.com/Rawl_B502/úib_luchta.trig#Eichlech    |
| http://example.com/Rawl_B502/úib_luchta.trig#Airechtach  |
| http://example.com/Rawl_B502/úib_luchta.trig#Dóelgus     |
| http://example.com/Rawl_B502/úib_luchta.trig#Abiél       |

The same set of URLs has been returned, but merely as a list which
cannot be further visualised or queried.

Continuing with the graph visualsation created above, one can explore
the graph a bit at a time as one can see below, with reasoning turned
off to ensure that the query can finish:

<img src="{{site.baseurl}}/assets/images/construct_baeth_2.png" />

A much more ambitious graph to construct is the graph of Báeth's
ancestors or to reconstruct his genealogical line from the
database.  This query introduces a few new constructs which are
handy when working with SPARQL.

```sparql
prefix rel: <http://purl.org/vocab/relationship/>

construct {
    ?s rel:childOf ?parent.
    ?y rel:parentOf ?baeth.
} 
from <tag:stardog:api:context:all>
where {
    values ?baeth {<http://example.com/Rawl_B502/clann_aingeda.trig#Báeth> }

    {
      ?s rel:ancestorOf ?baeth;
         rel:childOf ?parent
    } union {
      ?y rel:parentOf ?baeth
    }
}
```

The `construct` form first shows what will be the product of the
query.  The `values` keyword is used to store static URLs which will
be used repeatedly within the query.  This cuts down on the visual
complexity of the query overall.  The next part of the query is the
meat of the process.  Expressed informally, this says: "return all
people who are the ancestors of Báeth and their parent-child
relationship and return the parent of Báeth as well".  These are then
reconstructed in the `construct` form to create the reconstructed
genealogy of Báeth.  The `union` is used to get Báeth themselves
attached to the output through their parent which makes it slightly
easier to interpret the visualisation.

<img style="height:1000px;width:auto;" src="{{site.baseurl}}/assets/images/construct_baeth_3.png" />

One will note that the genealogy only goes back to Imchad.  This is
due to the fact that Imchad is not linked to any other part of the
genealogies.  The reason for this is outside the scope of this post
but could be as simple as a missing `owl:sameAs` link from elsewhere.
The curators tend to be conservative and if there is no clear evidence
that two individuals are the same, the policy is to leave them as is.
Finally, a note about performance.  This query took 21000 milliseconds
or about 21 seconds to run.  Reconstructing this by hand would take
much longer and creating a visualisation even longer still.  That it
results in an interactive visualisation is just one more additional
benefit.

The graph created by the construct query above is an amalgam of data
from all the genealogical collections in the corpus.  A graph from one
single tradition would require the use of the `from named` portion of
the query and will be the subject of another post because `from named`
requires special treatment as it interacts with named graphs in some
non-obvious ways.

As we can see, `construct` queries allow the transformation of RDF
graphs found in the Triplestore into other RDF graphs.  Using this, we
can extract interesting sub-graphs from the dataset and visualize them
with both Stardog Studio and GraphDB supporting this functionality.
This allows the reconstruction of genealogical lines and visual
exploration of the data that is latent in the dataset through a
Triplestore's reasoning capabilities.
