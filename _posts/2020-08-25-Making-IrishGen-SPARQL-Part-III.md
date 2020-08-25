---
layout: post
date: 2020-08-25
title: "Making IrishGen SPARQL Part III: Ask and Describe"
author: cgy
---

{% assign author = site.data.authors[page.author]%}
{{author.name}}

ORCID: <a href="https://orcid.org/{{ author.orcid }}" title="{{author.name}}">{{author.orcid}}</a>

Over the last two posts ([part I]({% post_url
2020-06-26-Making-IrishGen-SPARQL-Part-I %}), [part II]({% post_url
2020-08-05-Making-IrishGen-SPARQL-Part-II %})), the two main query
forms of SPARQL (Select and Construct) have been shown with examples
from IrishGen.  These two query forms together comprise the bulk of
the operations a regular user of SPARQL will need to use on a regular
basis.  This post will explain the uses of the two less often used
query forms:
[Ask](https://www.w3.org/TR/2013/REC-sparql11-query-20130321/#ask) and
[Describe](https://www.w3.org/TR/2013/REC-sparql11-query-20130321/#describe).
These will be covered in the order in which they are most often used.

# Describe

The Describe query will return the RDF about a single URL supplied by
the user.  What is returned by a Triplestore is undefined by SPARQL
(and deliberately so) in the standard.  What is generally returned is
the RDF that the Triplestore has about a particular URL.  This is
useful since, if the user knows the URL, they can interrogate the
Triplestore about what is stored in the Triplestore about that URL
without needing to go through the ceremony of a construct query.
Moreover, Describe queries are often used in visual representations of
RDF graphs.

For instance, the screenshot below is from GraphDB's "Visual Graph"
feature and uses Finn mac Cumaill as the exemplar.  What happens is
that the user sends the URL in which they are interested, GraphDB then
makes a Describe query on the URL, and RDF returned is used by GraphDB
to create a visual representation.  This is very useful when
investigating single individuals within IrishGen.  Additionally, it is
good if a user wants to have a secure, single starting point to
explore the graph in various ways.

<img src="{{site.baseurl}}/assets/images/describe_part3.png" />

Using the "Expand" feature of GraphDB's Visual Graph will send further
Describe queries to the Triplestore.  The resulting RDF will then be
rendered in visual form and any connections will be presented to the
user.  Describe queries can also be done as plain SPARQL as well.  For
instance:

```sparql
describe <http://example.com/LL/lagin.trig#Find>
```

will return over 1000 results giving everything that the Triplestore
contains about Find mac Cumaill.  The result of this is that the
Visual Graph can be overwhelmed with information.  The best practice
here is to choose the predicates that the user is interested in rather
than attempting to sift through all the information.  While
manipulating the Visual Graph feature of GraphDB is outside the scope
of the present post, most often a user is only interested in one or
two predicates so using the Visual Graph settings to filter out all
results other than the ones that the user are interested in is
generally desirable.

# Ask

The Ask query has one use to inform the user if a basic graph pattern
has a solution or not.  No information other than a boolean that
indicates existence or otherwise of a solution is returned.

In terms of IrishGen, this query form is used to ask for a quick
indication if the information that a user is looking for exists within
IrishGen.  There is little point in using this in a SPARQL query when
using IrishGen.  A user will be better served by using a Select or
Construct query as usually a user will need the results of a query
rather than mere existence of a solution.  However, as an example:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

ask
from named <http://example.com/LL>
where {
    graph ?g {
    	?a irishRel:nomName "Find"
    }
}
```

This query asks if there are any URIs that have the predicate
`irishRel:nomName` and object literal "Find" or, more informally, if
there is someone with the nominative name Find in LL.  This will
return `YES` as there is, in fact, a solution to the given basic graph
pattern.

More broadly, the Ask query can be useful for what are called
[Federated Queries](https://www.w3.org/TR/sparql11-federated-query/),
which is an advanced SPARQL topic which will be covered in a future
post.  However, in terms of how IrishGen is currently configured,
incoming Federated Queries are not possible due to the fact that the
curators have not invested in server infrastructure that would allow
interested parties to interrogate IrishGen through a SPARQL endpoint
which is a URL that SPARQL queries can be send using HTTP without
mandating that the user load IrishGen on their own machines.  SPARQL
endpoints are the standard method for ask and answering SPARQL queries
across the Web.

# Conclusion

This concludes the series on basic SPARQL in terms of IrishGen.  While
not every possible combination was explored, these posts provide a
basis for further exploring the IrishGen database.  Query languages
like SPARQL, most famously [SQL](https://en.wikipedia.org/wiki/SQL),
form the backbone of data intensive systems.  Learning to manipulate
the four SPARQL query forms demonstrated in this series directly will
give a user more power than any interface to a database could because
an interface will always restrict the kinds of queries that can be run
where writing queries in the language that is native to the
Triplestore will allow a flexibility that a restrictive user interface
will not.

From here, the user will need to use their new found SPARQL skills to
explore IrishGen and other datasets.  For instance, Wikipedia has two
widely used datasets: [DBPedia](https://wiki.dbpedia.org/) and
[Wikidata](https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service/Wikidata_Query_Help).
Practising and using IrishGen will cement the core skills presented in
this series and give the user a sense of the possibilities for further
research.  For instance, it is now possible to ask and answer
statistical questions concerning the genealogies using SPARQL's
[Aggregates](https://www.w3.org/TR/2013/REC-sparql11-query-20130321/#aggregates)
or with genealogies from other sources, especially more narrative
focused works, ask questions such as: were these characters connected
to the secular genealogies and where and how?  This can make the
genealogies more accessible for studying narrative focused works and
expand the analysis of those works beyond their immediate MS context.
Questions such as the latter will become increasingly interesting as
data is added in addition to the information contained in _Corpus
Genealogiarum Hiberniae_.

Future posts will cover advanced SPARQL and RDF features such has as
[Federated Queries](https://www.w3.org/TR/sparql11-federated-query/)
and [Named Graphs](https://en.wikipedia.org/wiki/Named_graph).  These
future posts will facilitate advanced skills and deepen the user's
appreciation of the power of Linked Data and RDF.
