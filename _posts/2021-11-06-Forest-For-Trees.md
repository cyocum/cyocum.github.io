---
layout: post
date: 2021-11-06
title: "Seeing the Forest for the Trees: Graphs and Trees in Designing IrishGen"
author: cgy
---

{% assign author = site.data.authors[page.author]%}
{{author.name}}

ORCID: <a href="https://orcid.org/{{ author.orcid }}" title="{{author.name}}">{{author.orcid}}</a>

There are several design decisions around the use of data structures
in IrishGen which I have not covered yet which may be both interesting
and educational.  This post will cover why certain certain data
structures were used to model the Irish genealogies.  These data
structures intersect with the wider goals of the project and it is
these goals which inform what file formats and data structures appear
in the database.

# IrishGen's Goals

The goal of the IrishGen database is to make the Irish genealogies
_structurally_ searchable.  This is a different goal than to make the
Irish genealogies _textually_ searchable.  For something to be
textually searchable means that a user can use something like Google
to search and find instances of text within a source document (like a
web page).  This can be done fairly easily using [Apache
Lucene](https://lucene.apache.org/) and its cousin project [Apache
Solr](https://solr.apache.org/).  These two software packages are
often used to replicate Google-like search functionality at a large
scale.  In fact, Apache Lucene is often paired with other software
packages, for instance, [BaseX](https://basex.org/) to allow for both
structural and textual searching
([XQuery](https://en.wikipedia.org/wiki/XQuery) plus Lucene).  This
strategy is, in fact, how [eDIL](http://www.dil.ie/) implements its
full text search functionality.  To make something structurally
searchable, a database designer needs to model the data in a way that
suits the underlying information which includes not only text but
structural markers that are latent in that text.  What the user can do
once this is done is query the structure of the data rather than the
data itself.  This can show patterns in the data that the user may be
interested in.  When this is combined with the textual search, the
user can then use both to answer whatever question they had in the
first place.  The clue is often in the name of the query language.
For instance, [SQL](https://en.wikipedia.org/wiki/SQL) stands for
_Structured Query Language_.  When someone creates a query in SQL,
they are querying not just the data but the structure of the data.

One last note before moving on, IrishGen does _not_ attempt to model
the textual evolution of the Irish genealogies.  Attempting to model
the textual evolution of the Irish genealogies would be a very
ambitious goal.  While it may be possible to model and search the
textual evolution of the Irish genealogies and given the tight
constraint on resources, a much more modest and achievable goal is to
do what IrishGen has done: structurally model the Irish genealogies
from the manuscript sources as they survive in the present time.  The
imagined use case for IrishGen is when a researcher has a genealogy
from elsewhere in medieval Irish literature and wishes to discover if
that genealogy appears within the broader Irish genealogical
tradition.  IrishGen is the first step when making these kinds of
enquiries.  The next step is to spend time determining where and how
that genealogy fits within the wider genealogical literature,
including the textual and linguistic strata within which the
particular instance sits.

# Structural Considerations

When considering the fine grained structure of the Irish genealogies,
Matthew Holmberg's PhD thesis, [Towards a Relative Chronology of the
Milesian Genealogical
Scheme](https://dash.harvard.edu/handle/1/37945002), is a very
valuable overview (especially pages 18-26).  He lists the several
broad categories for which see the link above.  A particularly good
example of structural aspects of a text is the "String Pedigree" as
discussed by Holmberg.  The example used here has been used before in
[_Human Curation and Digital Datasets: A Problem in Multiple
Parts_]({% post_url 2020-06-07-Human-Curation-and-Digital-Datasets
%}):

> Flaithbertach m Crunmael m Commáin m Fhinain m Faigfhir m Ernine m Feic m. Ieir m Gussa m Fobrich m Maeil m. Ainmerech m Fir Roith m Muine m Fir Nued m Fir Lugdach m Buain m Argatibair m Corpri Cluchechair m Con Corbb.

Here a glance at this pedigree will show the reader that the
structural element here is `m`, which stands for _mic_ (son of).
Large portions of the Irish genealogies are structured in this way.
Being able to query a database for "X son of Y" is a much more
powerful query than "show me all documents that contain X".  There are
other structural elements latent in the text which the curators use to
translate the text from the genealogical documents into a computer
parsable and searchable form.

Traditionally, genealogies are represented by trees.  Most often these
are called [B+ trees](https://en.wikipedia.org/wiki/B%2B_tree) because
one mother/father pair can have multiple offspring.  If you are
unfamiliar with the notion of trees in Computer Science, they are a
fundamental aspect of the discipline, I would highly recommend reading
the linked Wikipedia article and other resources to help facilitate
your understanding of the rest of the post.  B+ trees are particularly
useful in Computer Science generally and they sit at the heart of most
relational database's data indexing systems.  However, in the case of
the Irish genealogies, the tree model breaks down because there exist
in the Irish genealogies alternate genealogies which sit directly
alongside and sometimes within a genealogy.  For instance, from
_Genelach Eoganachta Glennamnach_ in the Book of Leinster

> Finguine m Loegaire m Duib Da Bairend. m Crundmael m Fogertaig m Fhailbe Flaind nó sic in aliis libris mc Crundmael m Dondgaile m Faelgusa m Nath Fraich m Colgan m Failbe Flaind.

While this structure could be modelled in a B+ tree like fashion, it
would cause the query necessary to retrieve or even attempt to
retrieve the data to be rather convoluted and awkward to write.  If
something becomes awkward to write in your chosen query language, this
is often a sign that the data model is either complex, which is
sometimes necessary and expected, or the data model does not have the
correct fit for the underlying data itself, in this case the
genealogies.  The question then becomes: is there a way to model
alternate genealogies without imposing too much awkwardness when
writing a search query?

B+ trees (and tree-like data structures in general) are actually a
sub-category of a more general data structure: the
[graph](https://en.wikipedia.org/wiki/Graph_(abstract_data_type)).  In
the Graph data structure, there are no constraints about how many
links can be made between different bits of information.  While in a
B+ tree, a node in the tree can have many children nodes where in the
graph child nodes can have many parents and, in fact, there is no
concept in a graph of a "parent" or "child" node.  This freedom from
constraint allows alternate genealogies to sit beside mainline
genealogies with no extra querying or modelling work required.  The
alternate genealogies are represented in IrishGen in the same way as
all other information within it is represented which makes it easy to
add to the database at any time without any more structural
considerations necessary.

There is, however, a drawback to this approach.  The more general a
data structure is the more computationally complex it is to execute
queries against that structure.  The reason that B+ trees have worked
so well for relational databases for decades is that they
can be well optimised and have very good computational complexity properties.

Once a graph is chosen as the underlying data structure, a database
and query language that can process graph data structures is
necessary.  In the case of IrishGen, RDF was chosen for reasons
outlined in [_IrishGen: RDF, Linked Data, and The Semantic Web_]({%
post_url 2020-06-17-IrishGen-RDF-Linked-Data-Semantic-Web %}) and
[_Triplestores, Ontologies, and Reasoning_]({% post_url
2020-06-22-Triplestores-Onotologies-Reasoning %}).

# Conclusion

This post covered some of the missing context as to why IrishGen
exists in the first place.  Additionally, the post covers some of the
data modelling considerations that went into the choice of a graph
data structure rather than a more mainstream technology like a
relational database.  Hopefully, the reader is now more informed as to
the reasoning behind IrishGen and not just what technology was chosen
and how that technology is used that has been covered in other posts.

