---
layout: post
date: 2020-06-26
title: "Making IrishGen SPARQL Part I: Selects"
author: cgy
---

{% assign author = site.data.authors[page.author]%}
{{author.name}}

ORCID: <a href="https://orcid.org/{{ author.orcid }}" title="{{author.name}}">{{author.orcid}}</a>

This post will begin to cover the [SPARQL query
language](https://www.w3.org/TR/sparql11-overview/) using IrishGen as
the example dataset.  This is the culmination of several posts that
guide the reader through the sometimes confusing world of Linked Data.
The post assumes that the reader is already familiar with [Linked
Data, Semantic Web]({% post_url
2020-06-17-IrishGen-RDF-Linked-Data-Semantic-Web %}), [Triplestores,
and logical reasoning using OWL 2]({% post_url
2020-06-22-Triplestores-Onotologies-Reasoning%}).  [Eystein
Thanisch](https://orcid.org/0000-0003-2819-5519) has already covered
some of this in [his post]({% post_url
2020-06-20-Some-Examples-of-Querying %}) which gives a practical and
useful introduction to using SPARQL with IrishGen.  This and following
posts is meant to deepen the reader's understanding and to give
examples of most forms of SPARQL so that the reader feels confident in
asking and answering their own questions of the IrishGen dataset.

## Báeth: A Useful Guide 

In the [Electronic Dictionary of the Irish
Language](http://www.dil.ie) (eDIL) defines
[báeth](http://www.dil.ie/5139) as "foolish, stupid, silly,
thoughtless, reckless" and also "in Laws applied to one not fully
responsible either through nonage or mental deficiency ...".  "Báeth"
as a name also appears in the genealogies and will serve as a guide
for many of the different kinds of queries that one can perform using
SPARQL.  It is unclear why this was used as a personal name in the
genealogies but its use could be connected to the dictionary
definition described above.

## SPARQL Query Forms: Select

SPARQL has several different kinds of
[queries](https://www.w3.org/TR/sparql11-query/#QueryForms):
[select](https://www.w3.org/TR/sparql11-query/#select),
[construct](https://www.w3.org/TR/sparql11-query/#construct),
[ask](https://www.w3.org/TR/sparql11-query/#ask), and
[describe](https://www.w3.org/TR/sparql11-query/#describe).  Select is
the most common and will be the focus of this post.  Construct queries
are the next most common but should have their own treatment as they
are more complex syntactically so will be deferred to another post.

We will begin our journey with Báeth with a very simple query (with
reasoning disabled for the moment):

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?x 
from <tag:stardog:api:context:all>
where {
    ?x irishRel:nomName "Baeth"
}
```

SPARQL resembles the [TRiG](https://www.w3.org/TR/trig/) file format
used in IrishGen in many ways.  However, there are differences between
them.  For the most part and in most practical situations, this does
not pose a problem and most tools that work with SPARQL that the user
will encounter will identify those for the user.

The `select` keyword tells the Triplestore that what is to follow is a
`select` query.  The `select`'s purpose is to return what is formally
termed a [_solution
sequence_](https://www.w3.org/TR/2013/REC-sparql11-query-20130321/#defn_sparqlSolutionSequence)
but for practical purposes can be understood as a list of terms which
match the query.  The `?x` is the query variable that the search is
interested in producing and will be reused in the query to indicate to
the Triplestore places in the query that will be filled by its search.
One can think of the variable `?x` as a hole in the RDF which the
computer is then asked to fill in.  There could be many different
solutions to filling the hole so each is returned as a valid way of
solving the problem of filling in the hole in the TRiG form presented
by the user.  The term `?x` is nothing special and it could very well
have been `?baeth` or any other set of characters.  [Query
variables](https://www.w3.org/TR/2013/REC-sparql11-query-20130321/#QSynVariables)
in SPARQL are prefixed with a `?` or a `$` so the above variable could
also be expressed as `$x`.  Naming variables is well-known as one of
the hard problems of Computer Science.  A rule of thumb is to make the
variable name meaningful unless the code is short as it is in this
case.

The `from` keyword controls what graph to create a default graph from.
In this instance, the `<tag:stardog:api:context:all>` is a Triplestore
dependent, in this case Stardog, variable which combines all available
graphs.  If, for instance, one only wanted to search for triples in
the Book of Leinster then the `from` statement would be `from
<http://example.com/LL>`.  This is a powerful way to constrain
searches to a particular MS or set of MSS and was one of the
motivating factors in choosing TRiG with its graph declarations rather
than continuing with [Turtle](https://www.w3.org/TR/turtle/) which
does not have the ability to constrain searches in this way.

The `where` keyword introduces the main section of the query which is
known as the _[basic graph
pattern](https://www.w3.org/TR/2013/REC-sparql11-query-20130321/#BasicGraphPatterns)_.
The basic graph pattern is basically a set of triples in Turtle form
where the `?x` is inserted to represent what part of the triple the
user is interested in being returned from the query.  The
`irishRel:nomName` is the predicate that we are interested in being
matched.  Finally, "Báeth" is the string literal that the user is
interested in matching.

To rewrite the above into more plain language: "Return to me the list
of all the subject URLs which have the predicate `irishRel:nomName`
and the object string "Báeth"" or, even more informally, "I want all
URLs where the person's nominative name is Báeth".

The results of the query using the current IrishGen are:

| ?x                                                                     |
| http://example.com/LL/forthart_fea.trig#Baeth                          |
| http://example.com/LL/ciarraige.trig#Baeth                             |
| http://example.com/LL/lagin.trig#Baeth                                 |
| http://example.com/LL/genelach_h_mugroin_i_m-maig_liphi.trig#Baeth     |
| http://example.com/LL/genelach_h_n-enechglais.trig#Baeth-fdda055e      |
| http://example.com/LL/eoganachta_casil.trig#Baeth                      |
| http://example.com/LL/de_genelach_dail_messi_corbb.trig#Baeth-a245b020 |
| http://example.com/LL/genelach_h_n-enechglais.trig#Baeth-50d79733      |
| http://example.com/LL/de_genelach_dáil_nia_corbb.trig#Baeth-a3e4de7a   |

A note about reasoning will be helpful here.  The reasoning
capabilities of Stardog, due to its backwards chaining strategy, can
be enabled or disabled at the time the query is sent to Stardog for
processing.  If it is turned off, the query will return only results
from the dataset as it stands in the files that are loaded into it.
GraphDB on the other hand has reasoning always enabled because it must
always pre-compute the entire graph before the query can be
processed. From this point onward, unless otherwise stated, assume
that Stardog is used and reasoning is enabled.

While this may be useful, someone knowledgeable concerning the
medieval Irish genealogies would question the overall usefulness as
there are many different spelling variations and other details to
consider.  In the instance of the query above, only instances of
"Baeth" that have a nominative name which _exactly_ matches will be
returned and all spelling variations will be ignored.

Additionally, Stardog elides some of the information as it will only
return one URL if they are all marked `owl:sameAs`, which is an
implementation detail which must be borne in mind by the user (see the
[note attached](https://www.stardog.com/docs/#_same_as_reasoning) to
the `owl:sameAs` reasoning in the Stardog manual for more
information).  Thus, Stardog will return all canonical URLs for Báeth
but not all _instances_ of Báeth in the Triplestore.  The difference
between individual and instance is a subtle one that would require a
more extended treatment elsewhere.  In this case, not all URLs which
match the query will be returned.

The next query will show how to have queries with multiple triples in
it:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?x ?y
from <tag:stardog:api:context:all>
where {
    ?x irishRel:nomName "Baeth";
       irishRel:genName ?y
}
```

This is a slight expansion of the above.  This query adds the `;`
which means "reuse the same subject" then gives the predicate
`irishRel:genName` then the variable `?y` which will capture any
genitive forms of the name that may appear.  More informally, this can
be translated: "Give me all subject URLs which have a nominative name
"Báeth" and also capture the genitive name and return it with the
subject URL".  One detail will need to be noted here but the query
will only match where there is an extact match of "Baeth" in the
nominative and the URL has a genative name.  Three results are
returned:

| ?x                                                                     | ?y      |
| http://example.com/LL/forthart_fea.trig#Baeth                          | "Baeth" |
| http://example.com/LL/genelach_h_n-enechglais.trig#Baeth-fdda055e      | "Baeth" |
| http://example.com/LL/de_genelach_dail_messi_corbb.trig#Baeth-a245b020 | "Báeth" |
| http://example.com/LL/genelach_h_n-enechglais.trig#Baeth-50d79733      | "Baeth" |

To create a more interesting query, let us ask who are the children of
Báeth?  To do this a new concept will need to be introduced: the
[blank
node](https://www.w3.org/TR/rdf11-concepts/#section-blank-nodes).
Blank nodes are URLs which have no subject or in terms of IrishGen,
there are people who are mentioned in the genealogies but they are not
given a name.  Since all URLs are predicated on the individual having
a name, a URL cannot be constructed for nameless individuals.  To
avoid this problem, as nameless individuals (most often women) are
genealogically important, RDF blank nodes are used to refer to these
individuals.  In SPARQL specification terms, [blank
nodes](https://www.w3.org/TR/2013/REC-sparql11-query-20130321/#QSynBlankNodes)
in graph patterns act as _variables_.  This, regrettably, can be
confusing to the beginner and expert alike but they serve different
purposes and is useful to keep in mind even if often the term variable
is used less formally to refer most often to query variables. The
usefulness of this will be shown in the query below:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 
prefix rel: <http://purl.org/vocab/relationship/>
prefix foaf:  <http://xmlns.com/foaf/0.1/>

select ?x
from <tag:stardog:api:context:all>
where {
   ?x rel:childOf [
      a foaf:Person;
      irishRel:nomName "Baeth"
   ]
}
```

The query can be translated as "give me all the people who are the
child of something which is a person and has the nominative name of
'Baeth'". Another way to think about the blank node in SPARQL is to
imagine that it is like `?x` but with some added constraints which
must be satisfied before the solution is added to the solution set.

In this case there are 7 results:

| ?x                                                                        |
| http://example.com/LL/forthart_fea.trig#Echdach                           |
| http://example.com/LL/genelach_h_mugroin_i_m-maig_liphi.trig#Thig         |
| http://example.com/LL/eoganachta_casil.trig#Butheni                       |
| http://example.com/LL/de_genelach_dail_messi_corbb.trig#Éitchen           |
| http://example.com/LL/genelach_h_n-enechglais.trig#Echach-faeed481        |
| http://example.com/Rawl_B502/genelach_úa_m-bairrche.trig#Breccán-f3d5ac80 |
| http://example.com/Rawl_B502/genelach_úa_m-bairrche.trig#Fóelchú          |

To return to the original objection, what about spelling variations?
To capture these kinds of variations we will need to find a way to
both broaden and filter our results.  This is possible in SPARQL using
the
[`filter`](https://www.w3.org/TR/2013/REC-sparql11-query-20130321/#expressions)
keyword.  The `filter`, according to the [SPARQL
specification](https://www.w3.org/TR/sparql11-query/#expressions),
"restrict[s] the solutions of a graph pattern match according to a
given constraint".  Essentially, `filter` allows the user to constrain
the kinds of results which are returned.  This can be very useful in a
variety of situations.  In the question above, the query will need to
be broadened but the user wants to restrict the kinds of solution sets
returned.  To wit:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?x ?y
from <tag:stardog:api:context:all>
where {
    ?x irishRel:nomName ?y.
    filter regex(?y, "^B[áa]eth$", "i")
}
```

This query returns 21 results.  The operative feature here is the line
`filter regex(?y, "^B[áa]eth$", "i")`.  This starts with the `filter`
keyword which tells the Triplestore that it should restrict the
solution set to the expression to follow.  `regex` is a function which
applies a _regular expression_ to the variable `?y` with the option of
`"i"` which means apply the regular expression with case insensitivity
(essentially, match upper and lower case letters with each other).
The term `regular` here comes from Latin _regula_ with the meaning "a
rule, pattern, model, example" (see Lewis and Short II
_[regula](http://www.perseus.tufts.edu/hopper/text?doc=Perseus%3Atext%3A1999.04.0059%3Aentry%3Dregula)_
and Oxford English Dictionary 3a
_[regular](https://www.oed.com/view/Entry/161414)_). Regular
expressions are a [very large
topic](https://en.wikipedia.org/wiki/Regular_expression) on their own
and are well worth the time to study as they appear in many computer
programming and query languages.  To quickly explain, regular
expressions are a string matching pattern language.  In the above
example, `^B[áa]eth$` can be translated into informal language as
"tell me if `?y` matches the following pattern: at the beginning of
`?y`, `B` followed by either `a` or `á` followed by `eth` and then end
of the string".  The option of case-insensitivity is active so `B`
will also match `b` and so on.
	
There is an [extensive set of
functions](https://www.w3.org/TR/2013/REC-sparql11-query-20130321/#SparqlOps)
available to build various filters and other constructs in SPARQL.  It
is another good use of time to look over the list to gain some
familiarity with what is available.

To return to the children of Báeth example, in order to gain a full
appreciation of the number of children, now that we have a method for
capturing more forms of Báeth, a merged query to see what that might
look like:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 
prefix rel: <http://purl.org/vocab/relationship/>
prefix foaf:  <http://xmlns.com/foaf/0.1/>

select ?x
from <tag:stardog:api:context:all>
where {
   ?x rel:childOf [
      a foaf:Person;
      irishRel:nomName ?y.
   ]
   filter regex(?y, "^B[áa]eth$", "i")
}
```

This query returns 22 results which is only one more than the original
but with this we can see how to combine blank nodes with a filter and
that single individual may be crucial to a researcher's interest so it
is well worth the extra complexity.

One of the great benefits of RDF and Linked Data is its flexibility.
In relational databases, due to the closed world assumption, all
columns must be filled with values.  Optional values must still have a
dummy value inserted and must be accounted for in SQL queries.  RDF
does not have this restriction.  In the context of IrishGen, this
means that not all individuals will have all predicates.  What this
means is that a URL may or may not have any predicate.  When a user
writes a query with a predicate, that predicate must exist for the
match to be made.  What if the user did not know in advance?  SPARQL
has a solution for this through the
[`optional`](https://www.w3.org/TR/2013/REC-sparql11-query-20130321/#optionals)
keyword.

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?x ?nominative ?genitive
from <tag:stardog:api:context:all>
where {
    ?x irishRel:nomName ?nominative.
    optional { ?x irishRel:genName ?genitive }
    filter regex(?nominative, "^B[áa]eth$", "i")
}
```

This query returns 19 results.  While in earlier queries, the results
were deferred so that the reader was not overwhelmed or distracted by
the table, it is instructive to reproduce the whole table here.

| ?x                                                                             | ?nominative | ?genitive |
| http://example.com/LL/forthart_fea.trig#Baeth                                  | "Baeth"     | "Baeth"   |
| http://example.com/LL/ciarraige.trig#Baeth                                     | "Baeth"     |           |
| http://example.com/LL/genelach_h_mugroin_i_m-maig_liphi.trig#Baeth             | "Baeth"     |           |
| http://example.com/Rawl_B502/mínugud_senchusa_laigin_and_so_sís.trig#Báeth     | "Baeth"     |           |
| http://example.com/LL/genelach_h_n-enechglais.trig#Baeth-fdda055e              | "Baeth"     | "Baeth"   |
| http://example.com/LL/genelach_h_n-enechglais.trig#Baeth-fdda055e              | "Báeth"     | "Baeth"   |
| http://example.com/LL/eoganachta_casil.trig#Buith                              | "Baeth"     |           |
| http://example.com/Rawl_B502/genelach_corco_m_druad.trig#Báeth                 | "Báeth"     |           |
| http://example.com/Rawl_B502/genelach_ceníuil_bóguine.trig#Báeth               | "Báeth"     |           |
| http://example.com/Rawl_B502/do_primforslointib_Lagen_inso.trig#Báeth-604282d0 | "Báeth"     | "Báeth"   |
| http://example.com/LL/de_genelach_dail_messi_corbb.trig#Baeth-a245b020         | "Baeth"     | "Báeth"   |
| http://example.com/LL/de_genelach_dail_messi_corbb.trig#Baeth-a245b020         | "Báeth"     | "Báeth"   |
| http://example.com/LL/genelach_h_n-enechglais.trig#Baeth-50d79733              | "Baeth"     | "Baeth"   |
| http://example.com/LL/genelach_h_falgi.trig#Báeth                              | "Báeth"     |           |
| http://example.com/Rawl_B502/genelach_benntraige.trig#Báeth                    | "Báeth"     | "Báeth"   |
| http://example.com/Rawl_B502/clann_aingeda.trig#Báeth                          | "Báeth"     | "Báeth"   |
| http://example.com/Rawl_B502/genelach_ceníuil_dalláin.trig#Báeth               | "Báeth"     |           |
| http://example.com/Rawl_B502/genelach_úa_m-bairrche.trig#Báeth                 | "Báeth"     | "Báeth"   |
| http://example.com/Rawl_B502/genelach_úa_m-bairrche.trig#Bóeth                 | "Baeth"     |           |

In this case though, there is another thing to note.  The query will
return all results which have a nominative name.  This could result in
the tens of thousands.  It will then filter those by the regular
expression and that will spawn tens or hundreds of thousands of
computations or more.  While in this case it took only 1816
millisecond (1.8 seconds), this is very inefficient.  Users will need
to have some awareness of the complexity of their queries when running
them and strive for the most specific query that will satisfy their
requirements.  Some queries, however, will just take time and the user
will need to keep in mind that their queries will not always be
instantaneous.

The next natural question is: what happens when both names could be
present?  As the reader can see from the example above the `optional`
keyword needs `{` and `}` which implies that more than one thing can
be optional at a time.  The general case can be re-specified as: the
subject can have either `irishRel:nomName` or `irishRel:genName` or
both.  In this case SPARQL has the
[`union`](https://www.w3.org/TR/2013/REC-sparql11-query-20130321/#alternatives)
keyword which combines the results of graph patterns.  Thus, in the
case where both `irishRel:nomName` or `irishRel:genName` can be
optional, the query can be rewritten:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?x ?y
from <tag:stardog:api:context:all>
where {
    optional { 
      {?x irishRel:nomName ?y }
      union
      { ?x irishRel:genName ?y } 
    }
    filter regex(?y, "^B[áa]eth$", "i")
}
```

The above query will return 24 results.  This will not distinguish
between nominative or genitive and will combine both if found.  Most
of the queries so far have not taken advantage of the reasoning
capabilities that were [discussed previously]({% post_url
2020-06-22-Triplestores-Onotologies-Reasoning %}).  Thus, there is an
alternative way for this query to be written so that it will do a
similar search, that is search for names irrespective of their
grammatical case, but rely instead on reasoning rather than
optionality.  If the reader consults the definition of `nomName`
predicate in the
[`earlyIrishRelationship.ttl`](https://github.com/cyocum/irish-gen/blob/master/earlyIrishRelationship.ttl)
file, the reader will notice that:

```turtle
:nomName
    a owl:DatatypeProperty ;
    rdfs:subPropertyOf oldIrish:nominative, foaf:name .
	
:genName
    a owl:DatatypeProperty ;
    rdfs:subPropertyOf oldIrish:genitive, foaf:name .
```

This means that the nominative name is a sub-property of `foaf:name`.
In other words, all `:nomName` are `foaf:name`.  Further, `:genName`
is similarly defined.  Taken as a whole, this means that, in the
presence of a Triplestore with a sufficiently powerful reasoner,
`foaf:name` can take the place of the other combination of names.
Thus:

```sparql
prefix foaf:  <http://xmlns.com/foaf/0.1/>

select ?x ?y
from <tag:stardog:api:context:all>
where {
    ?x foaf:name ?y
    filter regex(?y, "^B[áa]eth$", "i")
}
```

The result of this query is also 24 as there are very few dative and
accusative cases encoded in the dataset.

There is a final form of the `select` query that is useful to discuss
before moving to `construct` queries is
[aggregates](https://www.w3.org/TR/2013/REC-sparql11-query-20130321/#aggregates).
This kind of query generally count or add or generally combines or
aggregates information, hence the name.  We will need to part ways for
a moment from our useful guide, Báeth.  There is encoded in the
dataset the `:numChild` predicate.  In the medieval texts of the
genealogies, the number of children a person had would occasionally be
recorded.  Thus we can ask who had the most recorded children in the
genealogies:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?x (max(?numChild) as ?maxNumChild) 
from <tag:stardog:api:context:all>
where {
    ?x irishRel:numChild ?numChild
} 
group by ?x
order by desc (?maxNumChild)
```

This query takes all URLs that have a `irishRel:numChild` declared on
them then counts the `max` of them for each individual.  This means
that it will find the largest `irishGen:numChild` that an individual
has declared on them.  It orders them _descending_ (desc) and it
[groups them
by](https://www.w3.org/TR/2013/REC-sparql11-query-20130321/#groupby)
`?x` which means that the query will break each group represented by
the variable `?x` into a separate group then work on that group.  In
this case, it is `?x` so that each URL is counted as its own group.
The result is an order list going from most to least which shows each
individual and their `:numChild` which is more than 1000 results.
Thankfully, this is a very fast query at 68 milliseconds.  Evaluating
software performance, particularly speed of execution, is a
exhaustingly large subject.  There is a very high variance in what the
user may experience on their own computers due to _inter alia_ CPU L1
and L2 cache strategies, RAM clock rate, disk seek speed, query
optimiser, and even more complicated events outside the control of the
Triplestore.

As the result is over 1000, we will only concern ourselves with the
top five. We can do that progrmmatically by using the `limit` keyword.

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#> 

select ?x (max(?numChild) as ?maxNumChild) 
from <tag:stardog:api:context:all>
where {
    ?x irishRel:numChild ?numChild
} 
group by ?x
order by desc (?maxNumChild)
limit 5
```


| ?x                                                                                                      | ?maxNumChild |
| http://example.com/Laud_Misc_610/CGH/do_minigud_senchais_fer_muman.trig#Óengus                          | 48           |
| http://example.com/Rawl_B502/de_genelogia_síl_ébir_16.trig#Óengus-e74701aa                              | 48           |
| http://example.com/Rawl_B502/de_peritia_&_de_genealogis_dál_niad_cuirp_incipit.trig#CathairMár          | 33           |
| http://example.com/Rawl_B502/de_peritia_&_de_genealogis_dál_niad_cuirp_incipit.trig#CathairMár-82bed530 | 30           |
| http://example.com/LL/de_genelach_dáil_nia_corbb.trig#CathairMár                                        | 30           |

The results of this raise an interesting point.  The reader will
notice that Cathair Már appears three times in the results.  This
means that there are _three_ different declared number of children for
Cathair Már and they are not counted the same as `max` should return
only the largest for a single individual.  As explained above, the
difference between individual and instance is subtle and deserves its
own treatment.  To investigate this is outside the scope of this post
but a few hypothesises suggest themselves.  First, that Cathair Már
has differing numbers and the genealogists had differing accounts and
were taking from their own sources or constructing genealogies with
competing interests.  Second, these are all the same person but they
have not been linked together by `owl:sameAs` by the curators which
would then match what is expected, which, given that the same number
of children are reported, would suggest that this is the cause, at
least in this case.  Third, there is a discrepancy between LL and Rawl
B502 that is now apparent by looking at this query.  Whatever the
actual underlying cause, the query demonstrates something that would
need to be done painstakingly by hand with an even larger margin for
error than using a Triplestore to do the calculation for the user.

There is, of course, another method of counting by taking advantage of
the reasoner.  The alternative method is to count the children
separately and not to return the figures given explicitly in the
genealogies.  In fact, many do not have explicit numbers of children
mentioned so this is the method by which most counting may be done.
The query below demonstrates this:

```sparql
prefix irishRel: <http://example.com/earlyIrishRelationship.ttl#>
prefix rel: <http://purl.org/vocab/relationship/>

select ?x (count(distinct ?y) as ?numChildren)
from <tag:stardog:api:context:all>
where {
    ?x rel:parentOf ?y
}
group by ?x
order by desc(?numChildren)
limit 5
```

As a reader familiar with the IrishGen dataset will know
`rel:parentOf` is not encoded very often within the dataset.  The
reasoner knows that `rel:parentOf` is the inverse of `rel:childOf` and
thus can logically infer the number of children by applying this logic
to the dataset.  Thus, queries can be constructed based on data that
is not encoded within the dataset.  This does come at a cost that
shifts the burden of calculating this from human beings encoding it in
the dataset to the computer but this is a much less labour intensive
way of determining these things.  In this case, the query is runs for
~1022 milliseconds (about 1 second).  The outcome of this query for
the top five people is:

| ?x                                                                              | ?numChildren |
| http://example.com/Laud_Misc_610/CGH/do_minigud_senchais_fer_muman.trig#Óengus  | 34           |
| http://example.com/Laud_Misc_610/CGH/senchus_dáil_fíatach.trig#AilellaÁuluim    | 30           |
| http://example.com/LL/laigsi.trig#CathairMár                                    | 29           |
| http://example.com/Rawl_B502/úi_meic_h_eirc.trig#ConallClóen                    | 25           |
| http://example.com/Rawl_B502/do_forslointib_ulad_iar_coitchiund_in_so.trig#Buan | 25           |

While again outside the scope of the present post, it is interesting
to note that Cathair Már has 29 calculated children while the
genealogists have counted differently.  Again, there could be various
reasons why this is, including errors in the data introduced by the
curators during the translation process into RDF and it would bear
more investigation but the overall point is that there are now ways to
do these kinds of queries which were not possible or overly onerous
previously.

## Conclusion

This post has covered some of the basic select query forms that a
researcher will encounter when using SPARQL by using Báeth to explore
each in turn.  There is, of course, much more to discover and it takes
time, practice, experimentation, and experience to use a Triplestore
with SPARQL.  However, a researcher should now feel confident that
they have enough to get started asking their own research questions of
the IrishGen dataset and begin to explore the new possibilities
afforded by the system.

A note of caution is useful here.  The data which comprises IrishGen
is under review and will change as errors and omissions are corrected.
All queries results should be examined for correctness before relying
on them.  Over time, these errors and omissions will be rectified but
it is a time consuming process which will possibly never be fully
completed.

In the next post, the second most common form of SPARQL query will be
explored: the `construct` query form which will introduce the reader
to the real power of reasoning and how to extract sub-graphs from
IrishGen and create visualisations which will help in their own
research.
