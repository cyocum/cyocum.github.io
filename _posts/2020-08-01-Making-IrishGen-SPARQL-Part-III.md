---
layout: post
date: 2020-08-01
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
from IrishGen.  These two query forms together form the bulk of the
operations a regular user of SPARQL will need to use on a regular
basis.  This post will explain the uses of the two less often used
query forms:
[Ask](https://www.w3.org/TR/2013/REC-sparql11-query-20130321/#ask) and
[Describe](https://www.w3.org/TR/2013/REC-sparql11-query-20130321/#describe).
These will be covered in the order in which they are most often used.

# Describe

The describe query will return the RDF about a single URL supplied by
the user.  What is returned by SPARQL is undefined (and deliberately
so) in the standard.  What is generally returned is the RDF that the
Triplestore has about a particular URL.  The usefulness of this is
that if the user knows the URL, they can interrogate the Triplestore
about what is stored in the Triplestore about that URL without needing
to go through the ceremony of a construct query.  Moreover, describe
queries are often used in visual representations of RDF graphs.

For instance, the below screenshot is from the GraphDB "Visual Graph"
feature and uses Finn mac Cumaill as the exemplar.  What happens is
that the user sends the URL in which they are interested and the
system renders the RDF returned as a visual representation.  This is
very useful when investigating single individuals within IrishGen.
Additionally, it is good if a user wants to have a secure, single
starting point to explore the graph in various ways.

<img src="{{site.baseurl}}/assets/images/describe_part3.png" />

Further describe queries can then be executed to investigate
connections further and the data returned can be integrated into the
visual representation of the graph.

# Ask

The ask query has one use to inform the user if a basic graph pattern
has a solution or not.  No other information than a boolean that
indicates existence of a solution is returned.

In terms of IrishGen, the use of this query form is to ask for a quick
indication if the information that a user is looking for exists within
IrishGen.  This can be useful for what are called [Federated
Queries](https://www.w3.org/TR/sparql11-federated-query/), which is an
advanced SPARQL topic which will be covered in a future post.
Although, in terms of how IrishGen is currently configured, incoming
Federated Queries are not possible and outgoing Federated Queries are
of minimal usefulness while Linked Data in Celtic Studies is not
ubiquitous.

# Conclusion

This concludes the series on SPARQL in terms of IrishGen.  While not
every possible combination was explored, these posts provide a basis
for further exploring the IrishGen database.  Query languages like
SPARQL, most famously [SQL](https://en.wikipedia.org/wiki/SQL), form
the backbone of data intensive systems.  Learning to manipulate these
directly will give a user more power than any interface to a database
could.
