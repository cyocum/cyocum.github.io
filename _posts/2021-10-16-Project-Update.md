---
layout: post
date: 2021-10-16
title: Project Update
author: cgy
---

{% assign author = site.data.authors[page.author]%}
{{author.name}}

ORCID: <a href="https://orcid.org/{{ author.orcid }}" title="{{author.name}}">{{author.orcid}}</a>

I have been thinking about various aspects of this project for a
while.  As you may have noticed, it went on hiatus for the last nine
months due to various life events as often happens.  This also gave me
some time to think about various aspects of the project.  I have come
to a few "executive decisions".

First, in line with the [Data on the Web Best
Practices](https://www.w3.org/TR/dwbp/) [Recommendation
8.6](https://www.w3.org/TR/dwbp/#dataVersioning), I will be versioning
the data from now on.  The way I will do this is to use git tags to
tag each merge into the master branch.  The format will be the [ISO
8601 date](https://en.wikipedia.org/wiki/ISO_8601) plus a dot then the
number of the merge that day starting at 1 and incremented from there.
For instance, if I were to release for the first time today, the tag
would be 20211016.1 and the second would be 20211016.2 and so on.

This versioning scheme will allow anyone who wishes to reference
IrishGen to show where in the history of IrishGen they got their
information.  This is in contrast to needing to use "Accessed on" or
the bare git commit hash.

Second, I will be removing the '#' from many of the entries and
putting that in the `@base` for each file.  This is to cut down on
repetition and visual "noise" when looking at the files.  I may in
the future swap from using `.trig#` to using `/` (the hash versus
slash debate).  By centralising the `#`, it will be easier to swap in
the future.

Third, I will be auditing all the files.  This for the sake of
accuracy and as my understanding of the medieval Irish genealogies
grows, the audit will reflect that growth.  This may never actually be
finished as I am doing the audit on my own in whatever free time I
feel like devoting to it.

Fourth, as I am auditing, I will be adding
[Agrelon](https://d-nb.info/standards/elementset/agrelon) predicates
to the entries.  This is to allow people to use Agrelon rather than
the Relationship Ontology.  I may in the future remove the
Relationship Ontology completely.  Some of the reasons for doing this
are outlined in this [post]({% post_url 2021-01-25-Agrelon %}).

Fifth, I am considering removing [Named Graphs]({% post_url
2020-10-18-Named-Graphs %}) from the database.  Named Graphs did not,
in the end, really serve the namespacing purpose that I wanted to use
them for.  Additionally, I feel that anyone using this database will
want to interrogate particular manuscript sources rather than all the
manuscripts mashed together in one search.  I thought that Named
Graphs would serve the purpose of namespacing the manuscripts and the
triples contained within but it seems to have caused more confusion
and frustration than clarity.  Strict namespacing seems to be better
controlled at the repository level within whatever RDF graph database
that you are using.

Sixth, I am also considering purchasing a domain name for the
database.  This will allow other people to reference the database in
various ways.  The cost is fairly minimal at around £10 per year so it
should not be too much of a burden.

The project will continue and will be trundling along as my energy
levels allow.  If you use IrishGen, please let me know.  I am always
happy to hear from anyone using it or interested in it.
