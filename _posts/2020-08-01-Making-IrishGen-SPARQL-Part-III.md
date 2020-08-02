---
layout: post
date: 2020-08-01
title: "Making IrishGen SPARQL Part III: Ask and Describe"
author: cgy
---

{% assign author = site.data.authors[page.author]%}
{{author.name}}

ORCID: <a href="https://orcid.org/{{ author.orcid }}" title="{{author.name}}">{{author.orcid}}</a>

Over the last two posts, the two main query forms of SPARQL (Select
and Construct) have been shown with examples from IrishGen.  These two
query forms form the bulk of the operations a regular user of SPARQL
will need to use on a regular basis.  This post will explain the uses
of the two less often used query forms:
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
and uses Finn mac Cumaill as the exemplar.  What happens is that the
user sends the URL in which they are interested and the system renders
the return as a visual representation.  This is very useful when
investigating single individuals within IrishGen.

[insert screen shot here]

Further describe queries can then be executed to investigate
connections further and the data returned can be integrated into the
visual representation of the graph.

# Ask

The ask query has one use to inform the user if a basic graph pattern
has a solution or not.  No other information than a boolean that
indicates existance of a solution is returned.

In terms of IrishGen, the use of this query form is to ask a quick
indication if the information that a user is looking for exists within
IrishGen.

# Conclusion

This concludes 
