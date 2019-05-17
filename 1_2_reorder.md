# Spaltenreihenfolge

Inhalt: 
[Erläuterungen allgemein](#intro) || 
[JSON: Reorder nach MD1](#json-reorder-md1) || 
[JSON: Reorder nach MD2](#json-reorder-md2)

## Intro

* Ziel dieses Schrittes ist, die Spalten so anzuordnen, wie es für eine weitere manuelle Bearbeitung von Vorteil ist. Im Projekt vorhandene Spalten, die in diesem Schritt nicht aufgeführt werden, werden beim Reorder gelöscht. Wichtig ist, dass die Spaltennamen exakt übereinstimmen (Groß-/Kleinschreibung, [Whitespace](https://de.wikipedia.org/wiki/Leerraum) etc.); andernfalls werden die Spalten gelöscht.
* In OpenRefine gibt es verschiedene Möglichkeiten, Spalten neu anzuordnen &ndash; siehe dazu:
  * [OpenRefine Guide: *Column Editing*](https://github.com/OpenRefine/OpenRefine/wiki/Column-Editing)
  * [RefinePro: *Refine Tutorials: How to Reorder Columns* (YouTube)](https://www.youtube.com/watch?v=EpxvbRHLVvk)
  * [University Library at the University of Illinois: *LibGuides, OpenRefine, Reordering Columns*](https://guides.library.illinois.edu/openrefine/reordercolumn)
* Alle Möglichkeiten lassen sich über JSON reproduzieren, d.h. die Bearbeitungshistorie kann als Skript abgelegt und später nachgenutzt werden. Die Praxis zeigt, dass ab einer gewissen Anzahl von zu ändernden Spalten die Option b), die JSON-Anweisungen in einem Editor zu öffnen und zu editieren, am einfachsten zu handhaben ist.

**Option a)** Eine einzelne Spalte verschieben: 

**$Spalte** -> Edit column

=> Der unterste Block in dem Menü bietet vier Möglichkeiten, die Position einer Spalte zu verändern:

* Move column to beginning
* Move column to end
* Move column left
* Move column right

**Option b)** Mehrere Spalten neu anordnen: 

**All** -> Edit columns -> Re-order / remove columns...

=> **All** befindet sich links neben der ersten Spalte. Es öffnet sich ein Menü, in dem auf der linken Seite die Spaltenreihenfolge über Drag and Drop festgelegt werden kann. Nicht mehr benötigte Spalten können auf die rechte Seite verschoben werden, diese werden dann gelöscht.


## Erläuterungen zum Code

* Das JSON-Array besteht aus zwei JSON-Objekten:
  * Das erste Objekt (`"op": "core/column-removal"`) löscht nicht mehr benötigte Spalten, diese werden in Anführungsstrichen mit Kommata getrennt in das Feld `columnName` geschrieben. Dies ist sauber, aber nicht unbedingt nötig: Spalten, deren Name nicht in dem zweiten JSON-Objekt auftauchen, werden ebenfalls verworfen.
  * Das zweite Objekt (`"op": "core/column-reorder"`) ordnet die verbleibenden Spalten neu: Die Spalten erscheinen danach in der Reihenfolge, in der sie in dem Array `columnName` stehen (auch sie sind in Anführungszeichen mit Kommata getrennt aufzuzählen). Stehen in `columnName` Spaltennamen, die in dem Projekt gar nicht auftauchen, werden die Einträge ohne weitere Fehlermeldung ignoriert.
* Ein korrektes Ergebnis wird nur erzielt, wenn Spaltennamen exakt angegeben sind &ndash; Groß-/Kleinschreibung wird dabei unterschieden, auch abweichender [Whitespace](https://de.wikipedia.org/wiki/Leerraum) u.ä. führt dazu, dass Spalten nicht berücksichtigt werden. Alle Spalten, die nicht korrekt im Schritt "Reorder columns" angegeben werden, werden gelöscht.
* In beiden JSON-Objekten kann das Feld `description` bei einer späteren Ergänzung ignoriert werden.


```
[
  {
    "op": "core/column-removal",
    "description": "Remove column CITE_AUTHORS",
    "columnName": "CITE_AUTHORS"
  },
  {
    "op": "core/column-reorder",
    "description": "Reorder columns",
    "columnNames": [
      "Projekt Nr.",
      "to do",
      "Notiz OA Team",
      ...
    ]

  }
]
```


## JSON: Reorder MD1

```
[
  {
    "op": "core/column-reorder",
    "description": "Reorder columns",
    "columnNames": [
      "Projekt Nr.",
      "to do",
      "Notiz OA Team",
      "Notiz2",
      "DEPO_DOI",
      "DEPO_TITLE",
      "DOI",
      "DOI-Link",
      "doiRA",
      "URL",
      "Author",
      "Title",
      "dc.title.subtitle[en]",
      "dc.title[de]",
      "dc.title.subtitle[de]",
      "Type",
      "Date",
      "Publisher",
      "Journal",
      "ContainerTitle",
      "CR_ContainerTitle",
      "dcterms.bibliographicCitation.journaltitle[en]",
      "dcterms.bibliographicCitation.booktitle[en]",
      "dcterms.bibliographicCitation.proceedingstitle[en]",
      "ISSN",
      "eISSN",
      "CR_ISSN_ALL",
      "ISBN",
      "UPW: OA-Repos",
      "UPW: bestOA",
      "UPW: J DOAJ?",
      "UPW: IS OA?",
      "RP Ergebnis",
      "TU-Affiliation",
      "dc.rights.uri",
      "Datum Ablauf Embargo",
      "RP Version",
      "RP Embargo",
      "RP Phrase",
      "OAEZB_STATUS",
      "OAEZB_RECHTSGRUNDLAGE",
      "OAEZB_ANMERKUNGEN",
      "SR AOM",
      "SR AOM restriction",
      "SR AAM",
      "SR AAM restriction",
      "SR VOR",
      "SR VOR restriction",
      "SR Links Policy",
      "CR_LICENSE_URLs",
      "CR_FUNDER",
      "DATE PUBLISHED PRINT",
      "DATE PUBLISHED ONLINE",
      "CR",
      "UPW",
      "SR",
      "OAEZB"
    ]
  }
]
```

## JSON: Reorder MD2

```
[
    {
    "op": "core/column-reorder",
    "description": "Reorder columns",
    "columnNames": [
      "Projekt Nr.",
      "to do",
      "Notiz OA Team",
      "Notiz2",
      "DEPO_DOI",
      "DEPO_TITLE",
      "DOI-Link",
      "URL",
      "CORE_downloadUrl",
      "UPW: OA-Repos",
      "RP Ergebnis",
      "Datum Ablauf Embargo",
      "RP Phrase",
      "CITE_STRING",
      "TU-Affiliation",
      "OAEZB_RECHTSGRUNDLAGE",
      "OAEZB_STATUS",
      "OAEZB_ANMERKUNGEN",
      "id",
      "collection",
      "dcterms.bibliographicCitation.doi",
      "dc.type[en]",
      "dc.type.version[en]",
      "dc.contributor.author",
      "dc.language.iso",
      "dc.title[en]",
      "dc.title.subtitle[en]",
      "dc.title[de]",
      "dc.title.subtitle[de]",
      "dc.date.issued",
      "DATE PUBLISHED PRINT",
      "DATE PUBLISHED ONLINE",
      "dcterms.bibliographicCitation.journaltitle[en]",
      "Journal",
      "ContainerTitle",
      "CR_ContainerTitle",
      "dcterms.bibliographicCitation.booktitle[en]",
      "dcterms.bibliographicCitation.proceedingstitle[en]",
      "dcterms.bibliographicCitation.editor",
      "dcterms.bibliographicCitation.originalpublishername[en]",
      "dcterms.bibliographicCitation.originalpublisherplace[en]",
      "dc.subject.ddc[de]",
      "dc.identifier.issn",
      "dc.identifier.eissn",
      "dc.identifier.isbn",
      "ARXIV_abstract",
      "CORE_abstract",
      "PM_abstract",
      "dc.description.abstract[de]",
      "dc.description.abstract[en]",
      "dc.subject.other[de]",
      "dc.subject.other[en]",
      "dc.description.sponsorship[en]",
      "dcterms.bibliographicCitation.pagestart",
      "dcterms.bibliographicCitation.pageend",
      "dcterms.bibliographicCitation.articlenumber",
      "dcterms.bibliographicCitation.volume",
      "dcterms.bibliographicCitation.issue",
      "tub.publisher.universityorinstitution[de]",
      "tub.accessrights.openaire",
      "tub.accessrights.dnb",
      "dc.rights.uri",
      "dc.relation.ispartof",
      "dc.identifier.pmid[en]",
      "dc.description[de]",
      "dc.description[en]",
      "CR",
      "UPW",
      "SR",
      "OAEZB",
      "CORE",
      "SPRINGER",
      "ARXIV",
      "BASE",
      "PM_CONVERTER_API",
      "PM_MD_API",
      "doiRA",
      "CR_ISSN_ALL",
      "UPW: bestOA",
      "UPW: J DOAJ?",
      "UPW: IS OA?",
      "RP Embargo",
      "SR AOM",
      "SR AOM restriction",
      "SR AAM",
      "SR AAM restriction",
      "SR VOR",
      "SR VOR restriction",
      "SR Links Policy",
      "CR_LICENSE_URLs"
    ]
  }
]
```
