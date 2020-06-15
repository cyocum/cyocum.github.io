---
layout: post
date: 2020-06-14 14:00:00 +0100
title: Emendations to Source Editions on IrishGen
author: et
---

{% assign author = site.data.authors[page.author]%}
{{author.name}}

ORCID: <a href="https://orcid.org/{{ author.orcid }}" title="{{author.name}}">{{author.orcid}}</a>

## Introduction
When adding genealogies to IrishGen, as Christopher Yocum has [already noted](https://cyocum.github.io/2020/06/07/Human-Curation-and-Digital-Datasets.html), we generally use existing printed editions rather than making our own transcriptions. This is due to both limitations on our time and the need to remain accountable when encoding data. The printed editions we use need to be based on a single manuscript, as we want to be able to accurately encode data from each manuscript and thus track all the possible variants in the tradition. Fortunately, several medieval genealogical collections are available in such editions and these are available on [CELT](https://celt.ucc.ie/), including those in the [Book of Leinster](https://celt.ucc.ie//published/G800011F/index.html), [Rawl. B.502](https://celt.ucc.ie//published/G105003/index.html), and [Laud Misc. 610](https://celt.ucc.ie//published/G105005/index.html).

Occasionally, however, we encounter incongruities in existing editions and clarify the situation by checking online images of the manuscript. These images are available via repositories like [ISOS](https://www.isos.dias.ie) or [Digital.Bodleian](https://digital.bodleian.ox.ac.uk/). If the edition turns out to be in serious error, we follow the original manuscript in our encoding. This blogpost describes one such eventuality.

## The Anachronistic Ancestor of Dál nAraide
Cenél Lóegaire were a minor southern Uí Néill dynasty based around modern-day [Trim](https://www.logainm.ie/1416699.aspx ), Co. Meath. The genealogists traced their ancestry back to [Lóegaire](https://doi.org/10.1093/ref:odnb/15872), son of Níall Noígiallach, the pagan king of Tara whose druids confronted St. Patrick.

In the CELT edition of the Book of Leinster, [the pedigree](https://celt.ucc.ie//published/G800011F/index.html) of their royal line contains an account of Fiacha Araide. He is the ancestor figure of the Ulster polity of Dál nAraide, based in Mag nEilne, the area just east of [Coleraine](https://www.logainm.ie/1416661.aspx).

> Cu Ulad m Domnaill m Gillai Ultain m Oengusa m Cainelbain m Mael Chróin m Domnaill m Con Rai m Oengusa m Fheradaig Araide Bibre in cáinte de Mumnechaib ba sé ba rectaire do Chormac .h. Chuind. Cairech a ben. Is i ro anacht Fiacha mc Oengusa. inde dicitur Fiacha Araide a quo Dal Araide. m Mael Duin Dergainig m Colmáin m Aeda m Libuir m Dalleni m Ennai m Loegaire m Neill Noigiallaig.
> > Cú Ulad, son of Domnall, son of Gilla Ultain, son of Óengus, son of Caindelbán, son of Mael Chróin, son of Domnall, son of Cú Rai, son of Óengus, son of Feradach (Araide Bibre. The satirist from among the Munstermen. He was steward to Cormac, grandson of Conn. Cairech was his wife. It was she who cared for Fiacha, son of Óengus. Thence he is called Fiacha Araide, whence are called the Dál nAraide), son of Máel Duin Dergainig, son of Colmán, son of Áed, son of Libur, son of Dallene, son of Enna, son of Lóegaire, son of Níall Noígiallach.

This is impossible. Cormac, grandson of Conn, is Cormac mac Airt. He is Níall Noígiallach's legendary ancestor, placed five generations previous to him in the genealogies. Cormac's steward cannot be Níall's descendant, especially not by nine generations.

## The Edition
The passage quoted above is [the entry on Cenél Lóegaire](https://celt.ucc.ie//published/G800011F/index.html) from the edition of the Book of Leinster as digitised on CELT (the translation is my own). In [the printed edition](https://www.vanhamel.nl/codecs/Best,_et_al._1954-1983) (p. 1467), however, it is made clear that passage contains a section that has been added in the margin of the manuscript page. I have emphasised the inserted material **in bold**; in the printed edition, it is presented in a footnote but the reference number for the footnote is still placed after "m Fheradaig".

> Cu Ulad m Domnaill m Gillai Ultain m Oengusa m Cainelbain m Mael Chróin m Domnaill m Con Rai m Oengusa m Fheradaig **Araide Bibre in cáinte de Mumnechaib ba sé ba rectaire do Chormac .h. Chuind. Cairech a ben. Is i ro anacht Fiacha mc Oengusa. inde dicitur Fiacha Araide a quo Dal Araide.** m Mael Duin Dergainig m Colmáin m Aeda m Libuir m Dalleni m Ennai m Loegaire m Neill Noigiallaig.
> > Cú Ulad, son of Domnall, son of Gilla Ultain, son of Óengus, son of Caindelbán, son of Mael Chróin, son of Domnall, son of Cú Rai, son of Óengus, son of Feradach **(Araide Bibre. The satirist from among the Munstermen. He was steward to Cormac, grandson of Conn. Cairech was his wife. It was she who cared for Fiacha, son of Óengus. Thence he is called Fiacha Araide, whence are called the Dál nAraide)**, son of Máel Duin Dergainig, son of Colmán, son of Áed, son of Libur, son of Dallene, son of Enna, son of Lóegaire, son of Níall Noígiallach.

## The Manuscript
Turning to the manuscript, the insertion (now difficult to read) appears in the lower margin of p. 335 (full page available [here](https://www.isos.dias.ie/libraries/TCD/TCD_MS_1339/small_jpgs/335.jpg)), with a special symbol (an upward pointing angle containing three dots) that will mark its point of insertion in the main text:

<img src="{{site.baseurl}}/assets/images/post2_img1.jpg" />

*Detail from The Book of Leinster (TCD MS 1339), p.335, lower margin. Source: [ISOS](https://www.isos.dias.ie). Reproduced with permission of the Board of Trinity College Dublin.*

Within the pedigrees, this symbol is indeed placed next to "m Fheradaig" in the Cenél Lóegaire pedigree:

<img src="{{site.baseurl}}/assets/images/post2_img2.jpg" />

*Detail from The Book of Leinster (TCD MS 1339), p.335, cols d-h. Source: [ISOS](https://www.isos.dias.ie). Reproduced with permission of the Board of Trinity College Dublin.*

However, two columns to the right of "m Fheradaig" is [a Dál nAraide royal pedigree](https://celt.ucc.ie//published/G800011F/text063.html) and, pretty much aligned on the page with "m Fheradaig" is Fiacha Araide, the subject of the insertion. This makes for a much more cogent place for the insertion, in terms of both chronology and subject-matter. Scribal error seems to be behind the association of the insertion with Cenél Lóegaire, as there is plenty of space next to "m Fhiachach Araide" for the insertion mark, so there was no need to place it so far away.

## Course of Action on IrishGen

We have therefore encoded the insertion within [the RDF rendering of the Dál nAraide pedigree](https://github.com/cyocum/irish-gen/blob/master/LL/dail_araide.trig), which we read as follows (insertion **in bold**):

> Dondchad m Aeda m Longsig m Etig m Lethlobair m Longsig m Thommaltaig m Indrechtaig m Lethlobair m Echach Iarlathi m Fhiachnai m Boetain m Echach m Condlai m Coelbaith m Cruind Ba Drúi. m Echach m Lugdach m Rosa m Imchada m Fheidlimthi m Meic Caiss m Fhiachach Araide. **Araide Bibre in cáinte de Mumnechaib ba sé ba rectaire do Chormac .h. Chuind. Cairech a ben. Is i ro anacht Fiacha mc Oengusa. inde dicitur Fiacha Araide a quo Dal Araide.** m Oengusa Gobnenn m Fhergusa m Tipraite (qui occidit Cond) m Bresail m Fheirbb m Máeil m Rochride m Cathbath m Aillchatho m Findchada m Muridaig m Fhiachach Find m Amnais m Iriel Glúnmair m Conaill Cernaig.
