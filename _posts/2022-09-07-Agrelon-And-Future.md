---
layout: post
date: 2022-09-07
title: "Agrelon and the Future Direction of IrishGen"
author: cgy
---

{% assign author = site.data.authors[page.author]%}
{{author.name}}

ORCID: <a href="https://orcid.org/{{ author.orcid }}" title="{{author.name}}">{{author.orcid}}</a>

With the newest release of IrishGen
[20220907.1](https://github.com/cyocum/irish-gen/releases/tag/20220907.1),
the [Agrelon](https://d-nb.info/standards/elementset/agrelon) ontology
is fully incorporated in the IrishGen database.  This will allow users
to query IrishGen based on either the [Relationship
Ontology](http://purl.org/vocab/relationship/) or Agrelon.  I would
recommend using Agrelon from 20220709.1 onward for reasons set out in
[this post]({% post_url 2021-01-25-Agrelon %}).

At this point there is not much more than can be done with IrishGen in
its current form.  To get where it is, I used
[CELT](https://celt.ucc.ie/) transcriptions of LL and Rawl. B502.
Other information was transcribed from published genealogical sources
which were easily transcribed.  For the medieval Irish genealogies,
that is not the whole corpus.  There are many other genealogical
corpora.  Currently, there are 38 open MS corpora as recorded in the
IrishGen [project
page](https://github.com/cyocum/irish-gen/projects/1).  To get these
into IrishGen, I would need to transcribe them into a file then
translate them into RDF.  This is a laborious process that, while
valuable in its own right, will take time and money and is
overwhelming for a single individual to do.  I started on NLS Adv MS
72.1.6 so I may return to that.  In the long run, IrishGen may not
grow very quickly without more volunteers or some other solution
coming to light.

One last thing that IrishGen needs that may be easily done is to swap
the hash for the slash in the URLs that are created for each
individual.  The reason for this is that the hash leaks that the
database is based on files as it includes the `.trig` in the URL.  If
this were transferred to the web and put on a server, it would be much
cleaner for access by browser and easier to remember.

For instance, currently to link to an individual instance in an item,
someone would need to use this URL
`http://example.com/LL/aisneidem_di_araill.trig#Conchobuir`.  This is
sometimes called a "leaky abstraction" as it leaks that there exists a
file called `aisneidem_di_araill.trig` which contains the item
entitled "Aisneidem Di Araill" and that it is in the TRiG file format.
To hide this from someone looking at a URL, swapping the `#` for a `/`
will do that and make a few other things possible.  A URL that uses a
slash will look like
`http://example.com/LL/aisneidem_di_araill/Conchobuir`.  In this
situation you can easily parse the URL this way `http://example.com` +
MS + Item + Individual Instance.  Thus if a user requests
`http://example.com/LL/aisneidem_di_araill/`, they will get the
metadata back about that item from the server.  If they request
`http://example.com/LL`, they can get the metadata about the
manuscript.  This hides the fact that the entire database is a set of
flat files on Github as the user will likely never care about that and
indeed should not care.  Moreover, the underlying database (if there
were to be one online in the future), would only need to run a SPARQL
`describe` query to return all known information about a requested URL.

I will concentrate on the hash/slash swap which will be a bit tricky
as I will need to keep the `owl:sameAs` links that are in the files
consistent if they change.  I have scripts which will help me detect
this so it is only a minor annoyance.  The next thing after that is
much more daunting which is transcribing genealogical corpora which
have not been published.  I will probably pick up NLS Adv MS 72.1.6
again which has some of its contents published
[here](https://github.com/cyocum/irish-gen-mss/blob/master/xml/nls_ms_72_1_6.xml)
as TEI XML.  I am not sure if it would be worth doing all the context
work to attempt a proper publication out of it (or any of the other
genealogical corpora) but I may explore what the options are once I
get that far.
