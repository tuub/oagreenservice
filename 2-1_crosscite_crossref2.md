# Crosscite und Crossref 2

Inhalt: 
[Erläuterungen Crosscite](#crosscite) || 
[Erläuterungen Spalten und Crossref 2](#spalten-und-crossref-2) || 
[JSON](#json)


## Crosscite

### Intro

* [Crosscite](https://crosscite.org/) ist ein gemeinsamer Service der vier DOI-Registrierungsagenturen Crossref, DataCite, mEDRA und ISTIC. Mithilfe von Crosscite können bibliografische Daten in verschiedenen Datenformaten (unter anderem RIS, Bibtex, JSON, XML) oder als formatierte Zitation abgerufen werden, wobei zahlreiche Zitierstile sowie Gebietsschemata (Sprache und Land) ausgewählbar sind.
* [Dokumentation](https://citation.crosscite.org/docs.html#sec-4-1)
* Mit der Nutzung von Crosscite kann der Formatierungsaufwand für eine einheitliche Zitation reduziert werden; diese kann z.B. in einem Titelblatt eingebunden werden, das dem Volltext vorangestellt wird. 


### Dokumentation zum Code

#### API Call

* Eine Anfrage an die Schnittstelle erfolgt, wenn eine DOI vorhanden ist (`if(isNonBlank(value)`).
* Bei der folgenden Anfrage wird im Header übermittelt, dass (1) der Rückgabewert Text (nicht etwa BibTex oder RIS), (2) der Zitierstil "apa" und (3) das Gebietsschema (locale) "en-GB" sein soll. 
* Ist ein anderer Aufbau für den Zitationsstring gewünscht, müssen entsprechend die Parameter für "style" und/oder "locale" angepasst werden. Am einfachsten identifiziert man geeignete Parameter über die [grafische Oberfläche von Crosscite](https://crosscite.org). Dabei ist zu beachten, dass ggf. die Schreibweise des DOI-Links abweicht und die unten beschriebene Transformation (ersetzen von `doi:` mit `https://doi.org/`) erforderlich ist.
* Ist die Anfrage bei Crosscite erfolgreich, wird der Wert in der neu anzulegenden Spalte **CITE_STRING** gespeichert.
* Für den Fall, dass keine DOI vorliegt und Crosscite keine Zitation zurückliefern kann, wird die Zelle mit einem leeren Wert angelegt.

**DOI** -> Edit column -> Add column by fetching URLs... -> **CITE_STRING**

* **Wichtig**: `Throttle delay: 1000 ms`
* **Wichtig**: `HTTP header...` -> show -> `Accept: text/x-bibliography; style=apa; locale=en-GB`
* **Wichtig**: `On error: set to blank` 

```
if(isNonBlank(value),
  'https://doi.org/' + value,
  null
)
```

#### CITE_STRING formatieren

* Der von Crosscite zurückgelieferte Wert wird transformiert:
  * Autor\*innen: Da Crosscite ab einer bestimmten Anzahl einzelne Namen unterdrückt (stattdessen Angabe "..."), wir die Namensverknüpfung mit "&" nicht wünschen und zudem normalisierte Autor\*innennamen (Umlaute, Sonderzeichen) nutzen wollen, werden die Namen ersetzt mit den in [MDK1](/README.md#metadatenkontrolle-1-mdk1) vorbereiteten Daten. (Die Namen liegen im Format `Nachname1, V1., Nachname2, V2., ..., & Nachname-n, V.-n` vor.)
  * DOI als Link: "doi:" wird ersetzt mit "https://doi.org/"
* Da verschiedene Werte in einem Schritt transformiert werden sollen, ist der GREL-Befehl etwas komplexer aufgebaut und folgt der Logik: `if` -> Unterscheidung, ob in der Spalte **Author** ein Wert vorhanden ist: Vorhandene Werte werden genutzt (Fall 1); ist die Spalte leer, werden die von der API gelieferten Werte genutzt und transformiert (Fall 2).
  * **Fall 1**: Es sind Werte in der Spalte **Author** vorhanden:
    * `cells.Author.value.replace('||', '; ')` -> Der Wert aus der Spalte **Author** wird übernommen und es werden die Trennzeichen `||` ersetzt durch `; ` (Semikolon + Leerzeichen).
    * `+ ' (' +` -> Hinter die Autor\*innen wird ein Leerzeichen und eine öffnende Klammer geschrieben, denn diese werden im kommenden Befehlsteil als Trennzeichen genutzt.
    * `value.split(' (')[1,value.split(' (').length()].join(' (')` -> Der Wert aus der Spalte **CITE_STRING** (= Wert der Crosscite-API) wird anhand des Trennzeichens ` (` (Leerzeichen + öffnende Klammer) getrennt. Infolge werden die Autor\*innen aus der Zeichenkette von Crosscite entfernt.
    * `.replace('doi:', 'https://doi.org/')` -> Die Buchstabenfolge "doi:" wird ersetzt durch "https://doi.org/", da die Angabe eines funktionierenden DOI-Links gwünscht ist.
  * **Fall 2**: Es sind *keine* Werte in der Spalte **Author** vorhanden: 
    * `value.split('. (')[0].replace('& ', '').replace('., ', '.; ')` ->  Der Wert aus der Spalte **CITE_STRING** (= Wert der Crosscite-API) wird anhand des Trennzeichens `. (` (Punkt + Leerzeichen + öffnende Klammer) getrennt. Es wird das erste Element (die Autor\*innen) ausgewählt (`[0]`) und das Zeichen "&" gelöscht (`.replace('& ', '')`). Die als Trennzeichen für Autor\*innen genutzten Kommas werden ersetzt gegen Semikolons (`.replace('., ', '.; ')`).
    * `+ '. (' +` -> Die Zeichenfolge `. (` (Punkt + Leerzeichen + öffnende Klammer) wurde bei der ersten Transformation als Trennzeichen genutzt; diese Zeichenfolge `. (` (Punkt + öffnende Klammer) wird nun wieder nach den Autor\*innen eingefügt.
    * `value.split('. (')[1,value.split('. (').length()].join('. (')` -> Der Wert aus der Spalte **CITE_STRING** (= Wert der Crosscite-API) wird anhand des Trennzeichens `. (` (Punkt + Leerzeichen + öffnende Klammer) getrennt. Es werden die Elemente ab dem zweiten (die restlichen bibliografischen Angaben) ausgewählt (`[1,value.split('. (').length()]`) und eingefügt.
    * `.replace('doi:', 'https://doi.org/')` -> Die Buchstabenfolge "doi:" wird ersetzt durch "https://doi.org/", da die Angabe eines funktionierenden DOI-Links gwünscht ist.


**CITE_STRING** -> Edit cells -> Transform...

```
if(isNonBlank(cells.Author.value),
  cells.Author.value.replace('||', '; ') + ' (' + value.split(' (')[1,value.split(' (').length()].join(' (').replace('doi:', 'https://doi.org/'),
  value.split('. (')[0].replace('& ', '').replace('., ', '.; ') + '. (' + value.split(' (')[1,value.split(' (').length()].join(' (').replace('doi:', 'https://doi.org/')
)
```


## Spalten und Crossref 2

### Intro

* Sind die Publikationen identifiziert, für die eine Zweitveröffentlichung auf DepositOnce möglich ist, kann im Folgenden der Import vorbereitet werden. Nicht für alle Felder, die für DepositOnce erforderlich sind, können Fremddaten nachgenutzt werden; einige sind manuell zu erfassen (in [MDK2](/README.md##metadatenkontrolle-2-mdk2)). Zur Unterstützung werden zusätzliche Spalten angelegt und Spalten umbenannt, so dass die Spaltennamen den internen DepositOnce-Feldern entsprechen.
* Bei der ersten Abfrage an Crossref wurde nur ein Teil der für das Repositorium benötigten bibliografischen Daten erfasst (vgl. [MD1](/1-1_doiRA_crossref.md#crossref)); diese sollen nun ergänzt werden.
* Ein erneuter Abruf der Crossref-API ist nicht erforderlich, denn das Crossref-JSON liegt bereits in der Spalte **CR** vor; die Daten können nun ausgelesen werden.


### Dokumentation zum Code

#### Neue Spalten

* Für den fehlerfreien DSpace-Import (s. auch [DSpace-Dokumentation](https://wiki.duraspace.org/display/DSDOC6x/Batch+Metadata+Editing)) muss die CSV einen bestimmten Aufbau haben:
  * Die erste Spalte **id** steuert das Anlegen neuer Datensätze (durch Eintrag `+`) oder das Überschreiben vorhandener Datensätze (durch Angabe der internen item ID).
  * In der zweiten Spalte **collection** ist die Collection einzutragen, der das neue item zugeordnet werden soll.
  * Daran anschließen können die Angaben zu Metadaten, wobei für jedes Feld, das ggf. mit einer Sprachangabe zu qualifizieren ist, eine Spalte anzulegen ist. Dabei ist die Reihenfolge der Spalten irrelevant (außer bei den ersten beiden Spalten **id** und **collection**).

##### id

* Es wird eine neue Spalte **id** angelegt und mit dem Wert `+` belegt, was beim CSV-Import das Anlegen eines neuen Datensatzes steuert.

**DOI** -> Edit column -> Add column based on this column -> **id**

```
'+'
```

##### collection

* DSpace-Logik: Jeder Datensatz ("item") muss einer Collection zurgeordnet sein.
* DepositOnce-Logik: Der Community- und Collection-Baum bildet die Organisationsstruktur der TU Berlin ab, d.h. es gibt pro Fakultät/Institut/Fachgebiet eine Community, welche jeweils zwei Collections hat ("Publications" für Textpublikationen, "Research Data" für Forschungsdaten)
* Es wird eine neue, leere Spalte **collection** angelegt, in die später der/die Handle(s) der Collection(s) eingetragen wird, um neue Datensätze einer bzw. mehreren Collection(s) zuzuordnen.

**DOI** -> Edit column -> Add column based on this column -> **collection**

```
null
```

##### dc.description.abstract[de]

* Es wird eine neue, leere Spalte **dc.description.abstract[de]** für das deutsche Abstract angelegt.

**DOI** -> Edit column -> Add column based on this column -> **dc.description.abstract[de]**

```
null
```

##### dc.description.abstract[en]

* Es wird eine neue, leere Spalte **dc.description.abstract[en]** für das englische Abstract angelegt.

**DOI** -> Edit column -> Add column based on this column -> **dc.description.abstract[en]**

```
null
```

##### dc.subject.other[de]

* Es wird eine neue, leere Spalte **dc.subject.other[de]** für deutsche Schlagwörter angelegt.

**DOI** -> Edit column -> Add column based on this column -> **dc.subject.other[de]**

```
null
```

##### dc.subject.other[en]

* Es wird eine neue, leere Spalte **dc.subject.other[en]** für englische Schlagwörter angelegt.

**DOI** -> Edit column -> Add column based on this column -> **dc.subject.other[en]**

```
null
```

##### dc.subject.ddc[de]

* Es wird eine neue, leere Spalte **dc.subject.ddc[de]** für die Dewey Decimal Classification (DDC) angelegt.

**DOI** -> Edit column -> Add column based on this column -> **dc.subject.ddc[de]**

```
null
```

##### tub.publisher.universityorinstitution[de]

* Es wird eine neue Spalte **tub.publisher.universityorinstitution[de]** angelegt und mit dem Standardwert belegt. Dieses interne Feld wird primär für die Datenlieferung an die Deutsche Nationalbibliothek (DNB) benötigt.

**DOI** -> Edit column -> Add column based on this column -> **tub.publisher.universityorinstitution[de]**

```
'Technische Universität Berlin'
```

##### tub.accessrights.openaire

* Es wird eine neue, leere Spalte **tub.accessrights.openaire** angelegt, in der später Angaben zum Status des Volltextzugriffs in DepositOnce eingetragen werden. Dieses interne Feld wird primär für die Datenlieferung an OpenAire benötigt.

**DOI** -> Edit column -> Add column based on this column -> **tub.accessrights.openaire**

```
null
```

##### tub.accessrights.dnb

* Es wird eine neue, leere Spalte **tub.accessrights.dnb** angelegt, in der später Angaben zu Zugriffsrechten für die DNB zu erfassen sind (d.h. zu welchen Bedingungen darf die DNB Zugriff auf Volltexte gewähren).

**DOI** -> Edit column -> Add column based on this column -> **tub.accessrights.dnb**

```
null
```

##### dc.relation.ispartof

* Es wird eine neue, leere Spalte **dc.relation.ispartof** angelegt, in der später durch Angabe einer DOI Beziehungen zu anderen Objekten eingetragen werden können.

**DOI** -> Edit column -> Add column based on this column -> **dc.relation.ispartof**

```
null
```

##### dc.description[de]

* Es wird eine neue, leere Spalte **dc.description[de]** angelegt &ndash; ein Freitextfeld in DepositOnce, welches primär für die von der DFG vorgegebenen (deutschen) Phrase zur Kennzeichnung von OA-Rechten aus Allianz- und Nationallizenzen genutzt wird.

**DOI** -> Edit column -> Add column based on this column -> **dc.description[de]**

```
null
```

##### dc.description[en]

* Es wird eine neue, leere Spalte **dc.description[en]** angelegt &ndash; ein Freitextfeld in DepositOnce, welches primär für die von der DFG vorgegebenen (englischen) Phrase zur Kennzeichnung von OA-Rechten aus Allianz- und Nationallizenzen genutzt wird.

**DOI** -> Edit column -> Add column based on this column -> **dc.description[en]**

```
null
```

#### Spalten umbenennen

* Für die ersten Schritte waren kurze und sprechende Spaltenbezeichnungen hilfreichen. Als Vorbereitung für den CSV-Import werden vorhandene Spalten nun umbenannt, so dass sie den internen DepositOnce-Feldern entsprechen.
* Folgende Spalten sind umzubenennen:
  * **DOI** -> **dcterms.bibliographicCitation.doi**
  * **Author** -> **dc.contributor.author**
  * **Title** -> **dc.title[en]**
  * **Type** -> **dc.type[en]**
  * **Date** -> **dc.date.issued**
  * **Publisher** -> **dcterms.bibliographicCitation.originalpublishername[en]**
  * **ISSN** -> **dc.identifier.issn**
  * **eISSN** -> **dc.identifier.eissn**
  * **ISBN** -> **dc.identifier.isbn**
  * **RP Version** -> **dc.type.version[en]**
  * **CR_FUNDER** -> **dc.description.sponsorship[en]**
* Dabei ist jeweils folgende Aktion durchzuführen:

**$Spalte alt** -> Edit column -> Rename this column -> **$Spalte neu**


#### Crossref-Daten auslesen

* Es werden weitere Metadaten von Crossref ausgelesen und den jeweiligen DepositOnce-Feldern zugeordnet.
* Ein erneuter Abruf der Crossref-API ist nicht erforderlich, denn das Crossref-JSON liegt bereits in der Spalte **CR** vor.

##### dc.language.iso

* Neue Spalte für Sprache der Publikation anlegen, ggf. vorhandenen Wert des Crossref-Feldes `language` übernehmen.

**CR** -> Add column based on this column -> **dc.language.iso**

```
value.parseJson().message.language
```

##### dcterms.bibliographicCitation.pagestart

* Neue Spalte für Seitenzählung anlegen, ggf. vorhandenen Wert des Crossref-Feldes `page` aufsplitten und Angabe für *erste Seite* übernehmen.

**CR** -> Add column based on this column -> **dcterms.bibliographicCitation.pagestart**

```
value.parseJson().message.page.split('-')[0]
```

##### dcterms.bibliographicCitation.pageend

* Neue Spalte für Seitenzählung anlegen, ggf. vorhandenen Wert des Crossref-Feldes `page` aufsplitten und Angabe für *letzte Seite* übernehmen.

**CR** -> Add column based on this column -> **dcterms.bibliographicCitation.pageend**

```
value.parseJson().message.page.split('-')[1]
```

##### dcterms.bibliographicCitation.articlenumber

* Neue Spalte für Artikel-ID (ggf. anstelle einer Seitenzählung vorhanden) anlegen, ggf. vorhandenen Wert des Crossref-Feldes `article-number` übernehmen.

**CR** -> Add column based on this column -> **dcterms.bibliographicCitation.articlenumber**

```
value.parseJson().message['article-number']
```

##### dcterms.bibliographicCitation.volume

* Neue Spalte für Bandangabe der Zeitschrift anlegen, ggf. vorhandenen Wert des Crossref-Feldes `volume` übernehmen.

**CR** -> Add column based on this column -> **dcterms.bibliographicCitation.volume**

```
value.parseJson().message.volume
```

##### dcterms.bibliographicCitation.issue

* Neue Spalte für Heftangabe der Zeitschrift anlegen, ggf. vorhandenen Wert des Crossref-Feldes `journal-issue` übernehmen.

**CR** -> Add column based on this column -> **dcterms.bibliographicCitation.issue**

```
value.parseJson().message['journal-issue'].issue
```

##### dcterms.bibliographicCitation.editor

* Es wird eine neue Spalte für Angaben zu Herausgeber\*innen angelegt und ggf. vorhandene Werte des Crossref-Feldes `editor` ausgelesen. Subfelder werden in Reihenfolge "Nachname, Vorname" erfasst.
* Gibt es mehrere Einträge, werden sie durch `||` getrennt.

**CR** -> Add column based on this column -> **dcterms.bibliographicCitation.editor**

```
forEach(value.parseJson().message.editor,v,
  v.family + ', ' + v.given
).join('||')
```

##### dcterms.bibliographicCitation.originalpublisherplace[en]

* Neue Spalte für Verlagsort anlegen, ggf. vorhandenen Wert des Crossref-Feldes `publisher-location` übernehmen.

**CR** -> Add column based on this column -> **dcterms.bibliographicCitation.originalpublisherplace[en]**

```
value.parseJson().message['publisher-location']
```


## JSON

```
[
  {
    "op": "core/column-addition-by-fetching-urls",
    "description": "Create column CITE_STRING at index 1 by fetching URLs based on column DOI using expression grel:if(isNonBlank(value),\n  'https://doi.org/' + value,\n  null\n)",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "CITE_STRING",
    "columnInsertIndex": 1,
    "baseColumnName": "DOI",
    "urlExpression": "grel:if(isNonBlank(value),\n  'https://doi.org/' + value,\n  null\n)",
    "onError": "set-to-blank",
    "delay": 1000,
    "cacheResponses": true,
    "httpHeadersJson": [
      {
        "name": "authorization",
        "value": ""
      },
      {
        "name": "user-agent",
        "value": "OpenRefine 3.0-beta [TRUNK]"
      },
      {
        "name": "accept",
        "value": "text/x-bibliography; style=apa; locale=en-GB"
      }
    ]
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column CITE_STRING using expression grel:if(isNonBlank(cells.Author.value),\n  cells.Author.value.replace('||', '; ') + ' (' + value.split(' (')[1,value.split(' (').length()].join(' (').replace('doi:', 'https://doi.org/'),\n  value.split('. (')[0].replace('& ', '').replace('., ', '.; ') + '. (' + value.split(' (')[1,value.split(' (').length()].join(' (').replace('doi:', 'https://doi.org/')\n)",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "CITE_STRING",
    "expression": "grel:if(isNonBlank(cells.Author.value),\n  cells.Author.value.replace('||', '; ') + ' (' + value.split(' (')[1,value.split(' (').length()].join(' (').replace('doi:', 'https://doi.org/'),\n  value.split('. (')[0].replace('& ', '').replace('., ', '.; ') + '. (' + value.split(' (')[1,value.split(' (').length()].join(' (').replace('doi:', 'https://doi.org/')\n)",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/column-addition",
    "description": "Create column id at index 7 based on column DOI using expression grel:'+'",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "id",
    "columnInsertIndex": 7,
    "baseColumnName": "DOI",
    "expression": "grel:'+'",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column collection at index 7 based on column DOI using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "collection",
    "columnInsertIndex": 7,
    "baseColumnName": "DOI",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dc.description.abstract[de] at index 7 based on column DOI using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dc.description.abstract[de]",
    "columnInsertIndex": 7,
    "baseColumnName": "DOI",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dc.description.abstract[en] at index 7 based on column DOI using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dc.description.abstract[en]",
    "columnInsertIndex": 7,
    "baseColumnName": "DOI",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dc.subject.other[de] at index 7 based on column DOI using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dc.subject.other[de]",
    "columnInsertIndex": 7,
    "baseColumnName": "DOI",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dc.subject.other[en] at index 7 based on column DOI using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dc.subject.other[en]",
    "columnInsertIndex": 7,
    "baseColumnName": "DOI",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dc.subject.ddc[de] at index 7 based on column DOI using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dc.subject.ddc[de]",
    "columnInsertIndex": 7,
    "baseColumnName": "DOI",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column tub.publisher.universityorinstitution[de] at index 7 based on column DOI using expression grel:'Technische Universität Berlin'",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "tub.publisher.universityorinstitution[de]",
    "columnInsertIndex": 7,
    "baseColumnName": "DOI",
    "expression": "grel:'Technische Universität Berlin'",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column tub.accessrights.openaire at index 7 based on column DOI using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "tub.accessrights.openaire",
    "columnInsertIndex": 7,
    "baseColumnName": "DOI",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column tub.accessrights.dnb at index 7 based on column DOI using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "tub.accessrights.dnb",
    "columnInsertIndex": 7,
    "baseColumnName": "DOI",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dc.relation.ispartof at index 7 based on column DOI using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dc.relation.ispartof",
    "columnInsertIndex": 7,
    "baseColumnName": "DOI",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dc.description[de] at index 7 based on column DOI using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dc.description[de]",
    "columnInsertIndex": 7,
    "baseColumnName": "DOI",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dc.description[en] at index 7 based on column DOI using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dc.description[en]",
    "columnInsertIndex": 7,
    "baseColumnName": "DOI",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dc.language.iso at index 7 based on column CR using expression grel:value.parseJson().message.language",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dc.language.iso",
    "columnInsertIndex": 7,
    "baseColumnName": "CR",
    "expression": "grel:value.parseJson().message.language",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dcterms.bibliographicCitation.pagestart at index 7 based on column CR using expression grel:value.parseJson().message.page.split('-')[0]",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dcterms.bibliographicCitation.pagestart",
    "columnInsertIndex": 7,
    "baseColumnName": "CR",
    "expression": "grel:value.parseJson().message.page.split('-')[0]",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dcterms.bibliographicCitation.pageend at index 7 based on column CR using expression grel:value.parseJson().message.page.split('-')[1]",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dcterms.bibliographicCitation.pageend",
    "columnInsertIndex": 7,
    "baseColumnName": "CR",
    "expression": "grel:value.parseJson().message.page.split('-')[1]",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dcterms.bibliographicCitation.articlenumber at index 7 based on column CR using expression grel:value.parseJson().message['article-number']",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dcterms.bibliographicCitation.articlenumber",
    "columnInsertIndex": 7,
    "baseColumnName": "CR",
    "expression": "grel:value.parseJson().message['article-number']",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dcterms.bibliographicCitation.volume at index 7 based on column CR using expression grel:value.parseJson().message.volume",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dcterms.bibliographicCitation.volume",
    "columnInsertIndex": 7,
    "baseColumnName": "CR",
    "expression": "grel:value.parseJson().message.volume",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dcterms.bibliographicCitation.issue at index 7 based on column CR using expression grel:value.parseJson().message['journal-issue'].issue",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dcterms.bibliographicCitation.issue",
    "columnInsertIndex": 7,
    "baseColumnName": "CR",
    "expression": "grel:value.parseJson().message['journal-issue'].issue",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dcterms.bibliographicCitation.editor at index 7 based on column CR using expression grel:forEach(value.parseJson().message.editor,v,\n  v.family + ', ' + v.given\n).join('||')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dcterms.bibliographicCitation.editor",
    "columnInsertIndex": 7,
    "baseColumnName": "CR",
    "expression": "grel:forEach(value.parseJson().message.editor,v,\n  v.family + ', ' + v.given\n).join('||')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dcterms.bibliographicCitation.originalpublisherplace[en] at index 7 based on column CR using expression grel:value.parseJson().message['publisher-location']",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dcterms.bibliographicCitation.originalpublisherplace[en]",
    "columnInsertIndex": 7,
    "baseColumnName": "CR",
    "expression": "grel:value.parseJson().message['publisher-location']",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-rename",
    "description": "Rename column DOI to dcterms.bibliographicCitation.doi",
    "oldColumnName": "DOI",
    "newColumnName": "dcterms.bibliographicCitation.doi"
  },
  {
    "op": "core/column-rename",
    "description": "Rename column CR_FUNDER to dc.description.sponsorship[en]",
    "oldColumnName": "CR_FUNDER",
    "newColumnName": "dc.description.sponsorship[en]"
  },
  {
    "op": "core/column-rename",
    "description": "Rename column Author to dc.contributor.author",
    "oldColumnName": "Author",
    "newColumnName": "dc.contributor.author"
  },
  {
    "op": "core/column-rename",
    "description": "Rename column Title to dc.title[en]",
    "oldColumnName": "Title",
    "newColumnName": "dc.title[en]"
  },
  {
    "op": "core/column-rename",
    "description": "Rename column Type to dc.type[en]",
    "oldColumnName": "Type",
    "newColumnName": "dc.type[en]"
  },
  {
    "op": "core/column-rename",
    "description": "Rename column Date to dc.date.issued",
    "oldColumnName": "Date",
    "newColumnName": "dc.date.issued"
  },
  {
    "op": "core/column-rename",
    "description": "Rename column Publisher to dcterms.bibliographicCitation.originalpublishername[en]",
    "oldColumnName": "Publisher",
    "newColumnName": "dcterms.bibliographicCitation.originalpublishername[en]"
  },
  {
    "op": "core/column-rename",
    "description": "Rename column ISSN to dc.identifier.issn",
    "oldColumnName": "ISSN",
    "newColumnName": "dc.identifier.issn"
  },
  {
    "op": "core/column-rename",
    "description": "Rename column eISSN to dc.identifier.eissn",
    "oldColumnName": "eISSN",
    "newColumnName": "dc.identifier.eissn"
  },
  {
    "op": "core/column-rename",
    "description": "Rename column ISBN to dc.identifier.isbn",
    "oldColumnName": "ISBN",
    "newColumnName": "dc.identifier.isbn"
  },
  {
    "op": "core/column-rename",
    "description": "Rename column RP Version to dc.type.version[en]",
    "oldColumnName": "RP Version",
    "newColumnName": "dc.type.version[en]"
  }
]
```
