# Abstracts, Keywords, DDC

Inhalt:
[Erläuterung allgemein](#intro) || 
[Crossref](#crossref) || 
[PubMed](#pubmed) || 
[BASE](#base) || 
[CORE](#core) || 
[arXiv](#arxiv) || 
[Springer Nature](#springer-nature) || 
[JSON für alle APIs](#json)


Es gibt nicht den einen Webservice, über den &ndash; möglichst für alle Publikationen &ndash; Abstracts, Keywords und Vorschläge für eine [DDC-Klassifikation](https://www.ddc-deutsch.de/) bezogen werden können. Daher werden Schnittstellen verschiedener Datenquellen abgefragt:

1. Crossref -> Abstract
2. PubMed -> Abstract, Keywords
3. BASE -> DDC
4. Core -> Abstract, Download-URL
5. arXiv -> Abstract
6. Springer -> Abstract, Keywords

Die Reihenfolge dieser Abfragen reflektiert Beobachtungen aus Tests: Datenquellen, die als aussichtsreicher bzw. zuverlässiger eingeschätzt werden, werden zuerst abgefragt. Zudem werden nicht aus allen Abfragen alle vorhandenen Felder extrahiert &ndash; Tests haben gezeigt, dass etwa trotz ähnlicher Benennungen die Feldinhalte nicht dem gesuchten Inhalt entsprechen. Für das Ergebnis (möglichst vollständige Metadaten aus Fremdquellen) spielen die Abfragen aufgrund der bisher geringen Erfolgsquote allerdings noch eine untergeordnete Rolle.


## Crossref

### Intro

* Ein erneuter Abruf der Crossref-API ist nicht erforderlich, denn das Crossref-JSON liegt bereits in der Spalte **CR** vor (vgl. [MD1](/1-1_doiRA_crossref.md#crossref)); die Daten können nun ausgelesen werden.

### Erläuterungen zum Code

#### Abstract

* In [MD2](/2-1_crosscite_crossref2.md) wurden zwei leere Spalten für das Abstract angelegt (**dc.description.abstract[en]** sowie **dc.description.abstract[de]**).
* Sofern in Crossref-Daten ein Abstract vorhanden ist, wird er nun in die Spalte **dc.description.abstract[en]** übernommen. In [MDK2](/README.md#metadatenkontrolle-2-mdk2) ist später zu prüfen, ob es sich ggf. um ein Abstract in anderer Sprache als Englisch handelt und der Inhalt daher in eine andere Spalte kopiert werden muss.
* Hinweis: Der von Crossref ermittelte Wert wird später überschrieben, wenn die Springer Nature-API ebenfalls ein Abstract liefert.

**dc.description.abstract[en]** -> Edit cells -> Transform...

```
cells.CR.value.parseJson().message.abstract
```


## PubMed

### Intro

* [PubMed](https://www.ncbi.nlm.nih.gov/pubmed/) ist die einschlägige Fach- und Literaturdatenbank für Publikationen aus dem Bereich (Bio-)Medizin; sie wird von der US-amerikanischen National Library of Medicine betrieben.
* PubMed stellt [verschiedene Schnittstellen](https://www.ncbi.nlm.nih.gov/home/develop/api/) zur Verfügung; es liegt jeweils eine ausführliche Dokumentation vor.
  * ID Converter API: PubMed-ID (PMID) für DOI ermitteln (vgl. [Dokumentation](https://www.ncbi.nlm.nih.gov/pmc/tools/id-converter-api/))
  * eSummary API: Metadaten (Abstract, Keywords usw.) auf Basis der PubMed-ID ermitteln (vgl. [Dokumentation](https://www.ncbi.nlm.nih.gov/books/NBK25499/#chapter4.ESummary))
* Weiterführende Literatur siehe [Fred Trotter (2014): Hacking on the Pubmed API](https://fredtrotter.com/2014/11/14/hacking-on-the-pubmed-api/)
* Um Metadaten für eine DOI zu beziehen, sind also mehrere Schritte erforderlich:
  1. Anfrage an ID Converter API auf Basis der DOI
  2. PMID auslesen und in Spalte **dc.identifier.pmid[en]** schreiben
  3. Anfrage an eSummary API auf Basis der PMDI
  4. Metadaten auslesen und in entsprechende Spalten schreiben


### Erläuterungen zum Code

#### Aufruf ID Converter API

* Es wird eine Anfrage an die ID Converter API auf Basis der DOI gestellt und die Antwort in der (neuen) Spalte **PM_CONVERTER_API** gespeichert.
* Die ID Converter API gibt Daten in verschiedenen Formaten aus: der Standard ist XML, sofern keine anderes Format angefragt wird, mit Parameter `&format=json` erfolgt die Rückgabe im JSON-Format.
* Eine Registrierung ist nicht erforderlich, jedoch sollen bei jeder Abfrage der Name der Applikation, für die die Schnittstelle genutzt wird, und die E-Mail-Adresse übermittelt werden.
* **Wichtig**: In der URL den Parameter `?tool=oagreenserviceTUberlin` durch Namen des eigenen 'Tools' ersetzen
* **Wichtig**: E-Mail-Adresse anpassen
* **Wichtig**: `Throttle delay	1000 milliseconds`

**dcterms.bibliographicCitation.doi** -> edit column -> Add column by fetching URLs -> **PM_CONVERTER_API**

```
'https://www.ncbi.nlm.nih.gov/pmc/utils/idconv/v1.0/?tool=oagreenserviceTUberlin&email=INSERT@YOUR.EMAIL&format=json&ids=' + value
```

#### PubMed-ID (PMID)

* Es wird das Feld `pmid` der Rückgabe der ID Converter API ausgelesen und die PubMed-ID in einer neuen Spalte **dc.identifier.pmid[en]** gespeichert.

**PM_CONVERTER_API** -> Edit column -> Add column based on this column -> **dc.identifier.pmid[en]**

```
forEach(value.parseJson().records,v,
  v.pmid
).uniques().join('||')
```

#### Aufruf eSummary API

* Die eSummary API ist Teil der [Entrez Programming Utilities (E-utilities)](https://www.ncbi.nlm.nih.gov/books/NBK25501/).
* Es wird eine Anfrage an die eSummary API auf Basis der PubMed-ID gestellt und die Antwort in der (neuen) Spalte **PM_MD_API** gespeichert.
* Die Schnittstelle gibt Daten prinzipiell in zwei Formaten aus (XML oder Klartext). Da die gewünschten Keywords nur im XML-Format enthalten sind, soll die Rückgabe in XML erfolgen.
* **Wichtig**: `Throttle delay	1000 milliseconds`

**dc.identifier.pmid[en]** -> Edit column -> Add column by fetching URLs -> **PM_MD_API**

```
if(isNonBlank(value),
  'https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&retmode=xml&rettype=abstract&id=' + value.replace('||', ','),
  null
)
```

#### Abstract

* Es wird das Feld `abstracttext` der Rückgabe der eSummary API ausgelesen und das Abstract in einer neuen Spalte **PM_abstract** gespeichert.

**PM_MD_API** -> Edit column -> Add column based on this column -> **PM_abstract**

```
forEach(value.parseHtml().select('abstracttext'),v,
  v.htmlText()
).uniques().join('||')
```

#### Keywords

* Es wird das Feld `keyword` der Rückgabe der eSummary API ausgelesen und Keywords werden in der vorhandenen Spalte **dc.subject.other[en]** gespeichert.

**dc.subject.other[en]** -> Edit cells -> Transform...

```
forEach(cells.PM_MD_API.value.parseHtml().select('keyword'),v,
  v.htmlText()
).uniques().join('||')
```


## BASE

### Intro

* Die wissenschaftliche Suchmaschine [BASE](https://www.base-search.net) aggregiert bibliografische Daten verschiedener Services mittels deren jeweiliger OAI-Schnittstelle (Repositorien, Open-Access-Zeitschriften, Forschungsinformationssysteme, Digitale Sammlungen und so weiter). Die Daten werden jedoch nicht zu einem Datensatz zusammengefügt, weshalb in dem Feld `docs` ein Array mit potentiell mehreren Dictonaries vorliegt.
* [Dokumentation der BASE-REST-API](https://www.base-search.net/about/download/base_interface.pdf)
* Prinzipiell ließen sich aus BASE neben Vorschlägen für die DDC auch Abstract und Keywords beziehen; allerdings sind die BASE-Felder nicht so granular, dass sich Angaben zu Abstract eindeutig von Angaben zu Funding o.Ä. (die von Repositorienbetreibern über die OAI-Schnittstelle im Format Dublic Core Simple im Feld "dc.description" geliefert werden) unterscheiden ließen. Aus diesem Grund lesen wir zunächst nur Angaben zur DDC aus den BASE-Daten aus.
* Die Schnittstelle gibt Daten prinzipiell in zwei Formaten aus. Der Standard ist XML, durch Angabe des Parameters `&format=json` erfolgt die Rückgabe im JSON-Format.
* **WICHTIG**: Um die Schnittstelle nutzen zu können, muss die [IP-Adresse freigeschaltet](https://www.base-search.net/about/en/contact.php) werden. Zudem sollte nicht mehr als eine Anfrage pro Sekunde gestellt werden.

### Erläuterungen zum Code

#### Aufruf BASE API

* Es wird eine Anfrage an die BASE-API auf Basis der DOI gestellt und die Antwort in der (neuen) Spalte **BASE** gespeichert.
* Bei der Anfrage wird übermittelt, welche Felder zurückgegeben werden sollen (hier: `dcautoclasscode`, `dcclasscode` und `dcdeweyfull`).

**dcterms.bibliographicCitation.doi** -> Edit column -> Add column by fetching URLs -> **BASE**

* **Wichtig**: `Throttle delay 1000 ms`

```
'https://api.base-search.net/cgi-bin/BaseHttpSearchInterface.fcgi?func=PerformSearch&query=dcdoi:'
+ value +
'&fields=dcautoclasscode,dcclasscode,dcdeweyfull&format=json'
```

#### DDC 

* BASE liefert zur DDC mehrere Angaben:
  * `dcclasscode`: DDC-Klasse, wie sie etwa von Repositorienbetreibern über die OAI-Schnittstelle ausgeliefert wird
  * `dcautoclasscode`: DDC-Klasse, wie sie mithilfe der [automatic classification toolbox for Digital Libraries (ACT-DL)](http://act-dl.base-search.net/) automatisch ermittelt wurde
* An dieser Stelle werden beide Werte (sofern vorhanden) ausgelesen und in folgenden Form ausgegeben: "BASE-Classifier: `dcautoclasscode` -- BASE-Repo-Daten: `dcclasscode`". 
* Angaben aus mehreren Quellen werden nicht aggregiert, sondern fortlaufend aufgeführt. Mehrere Einträge werden durch `||` getrennt; identische Einträge werden nur einmal ausgegeben (`set(...)`).

**dc.subject.ddc[de]** -> Edit cells -> Transform...
 
```python
import json

j = json.loads(cells.BASE.value)
response = j.get('response', {})
docs = response.get('docs', {})

return '||'.join(set(
                    ['BASE-Classifier: {}\nBASE-Repo-Daten: {}'.format(
                        '||'.join(i.get('dcautoclasscode', ['--'])),
                        '||'.join(i.get('dcclasscode', ['--'])))
                    for i in docs]
                    ))
```

* Beispiel 1 für Ausgabe: `BASE-Classifier: 621 BASE-Repo-Daten: --`
* Beispiel 2 für Ausgabe: `BASE-Classifier: 600 BASE-Repo-Daten: --||BASE-Classifier: -- BASE-Repo-Daten: 534`


## CORE

### Intro

* [CORE](https://core.ac.uk/about) aggregiert Metadaten und Volltexte von Repositorien und Journalen. Bibliografische Daten stellt CORE auch über eine Schnittstelle zur Verfügung. 
* Um die Schnittstelle abfragen zu können, muss ein [API-key registriert](https://core.ac.uk/searchAssets/api-keys/register/) werden, dessen Angabe bei der Abfrage verpflichtend ist.
* Daten werden im JSON-Format ausgegeben.
* [Dokumentation CORE-API.](https://core.ac.uk/documentation/api)


### Dokumentation zum Code

#### Aufruf CORE API

* Es wird eine Anfrage an die CORE-API auf Basis der DOI gestellt und die Antwort in der neu angelegten Spalte **CORE** gespeichert.

**dcterms.bibliographicCitation.doi** -> Edit column -> Add column by fetching URLs -> **CORE**

* **Wichtig**: API-Key für CORE eintragen (`CORE_API_KEY` mit [eigenem API-Key](https://core.ac.uk/searchAssets/api-keys/register/) ersetzen)

```
'https://core.ac.uk/api-v2/articles/search/doi:"' + value + '"?apiKey=CORE_API_KEY'
```

#### Abstract

* Es wird das CORE-Feld `description` ausgelesen und Abstracts werden in der neuen Spalte **CORE_abstract** gespeichert.

**CORE** -> Edit column -> Add column based on this column -> **CORE_abstract**

```
forEach(value.parseJson().data, v,
  v.description
).uniques().join('||')
```

#### Download-URL

* Es wird das CORE-Feld `downloadUrl` ausgelesen und Abstracts werden in der neuen Spalte **CORE_downloadUrl** gespeichert.
* Das Feld enthält URLs zum Direktdownload von Volltexten bei CORE, denn CORE harvestet Metadaten und frei zugängliche Volltexte von Repositorien und Journalen.

**CORE** -> Edit column -> Add column based on this column -> **CORE_downloadUrl**

```
forEach(value.parseJson().data, v,
  v.downloadUrl
).uniques().join('||')
```


## arXiv

### Intro

* arXiv ist ein Preprint-Server mit fachlichem Schwerpunkt in den Bereichen Physik, Mathematik, Informatik, Statistik, Finanzmathematik und Biologie. 
* arXiv stellt Daten über eine Schnittstelle zur Verfügung, für die keinerlei Registrierung oder Authentifizierung erforderlich ist.
* Es liegen verschiedene Dokumentationen zur Schnittstelle vor:
  * [Kurzbeschreibung arXiv API](https://arxiv.org/help/api)
  * [arXiv API User's Manual](https://arxiv.org/help/api/user-manual)
  * [FAQ arXiv API](https://arxiv.org/help/api/faq) 
* Daten werden im XML-Format (genauer: in Atom 1.0) ausgegeben.


### Dokumentation zum Code

#### Aufruf arXiv API

* Es wird eine Anfrage an die arXiv-API auf Basis der DOI gestellt und die Antwort in der (neuen) Spalte **ARXIV** gespeichert.


**dcterms.bibliographicCitation.doi** -> edit column -> Add column by fetching URLs -> **ARXIV**

```
'http://export.arxiv.org/api/query?search_query=doi:' + value
```

#### Abstract

* Es wird das arXiv-Feld `summary` ausgelesen und Abstracts werden in der neuen Spalte **ARXIV_abstract** gespeichert.

**ARXIV** -> Edit column -> Add column based on this column -> **ARXIV_abstract**

```
forEach(value.parseHtml().select('summary'),v,
    v.htmlText()
).uniques().join('||')
```


## Springer Nature

### Intro

* Springer Nature stellt für eigene Publikationen bibliografische Daten über verschiedene Schnittstellen zur Verfügung. Über die "Springer Nature Metadata API" etwa können Metadaten zu Zeitschriftenartikeln, Buchkapiteln etc. abgefragt werden. 
* Es liegen eine ausführliche Dokumentation und Beispiele für Abfragen vor:
  * [RESTful Operations](https://dev.springernature.com/restfuloperations)
  * [Example API Responses](https://dev.springernature.com/example-metadata-response)
  * [Documentation](https://dev.springernature.com/documentation)
* Um die Schnittstelle abfragen zu können, muss zunächst ein [Konto und dann ein API-Key](https://dev.springernature.com/signup) registriert werden; die Angabe des API-Keys ist bei der Abfrage verpflichtend.
* Die Schnittstelle gibt Daten prinzipiell in zwei Formaten (XML und JSON) aus. Bei der Abfrage ist anzugeben, in welchem Format die Rückgabe erfolgen soll (wir nutzen JSON, siehe `/json`).
* Wurde ein Abstract bereits über andere Datenquellen ermittelt, wird dieser Wert nun überschrieben, wenn die Springer Nature-API ebenfalls ein Abstract liefert.

### Dokumentation zum Code

#### Aufruf Springer API

* Die Springer Nature-API soll nur für Publikationen, die bei Springer erschienen sind, abgefragt werden. Daher wurde vorab eine Liste aller [DOI-Präfixe von Springer](http://api.crossref.org/members/297) auf Basis einer Crossref-Abfrage erstellt und diese in einen String geschrieben.
* Vor dem API-Aufruf wird der DOI-Präfix der jeweiligen Publikation (`value.split('/')[0])`) mit dieser Liste abgeglichen (`.contains(value.split('/')[0])`). `contains` gibt einen Boolschen Wert zurück: `true` führt den API-Call aus, `false` nicht.

**dcterms.bibliographicCitation.doi** -> Edit cloumn -> Add column by fetching URLs... -> **SPRINGER**

* **Wichtig**: API-Key für Springer Nature eintragen (`SPRINGER_API_KEY` mit [eigenem API-Key](https://dev.springernature.com/signup) ersetzen)

```
if('10.26778 10.26777 10.7603 10.33283 10.1617 10.1245 10.1251 10.3858 10.1208
    10.1114 10.3758 10.1186 10.1140 10.1361 10.1379 10.1381 10.1385 10.2165 
    10.19150 10.1007 10.1013 10.1065 10.1023 10.1038 10.1057 10.4333 10.5822
    10.5819 10.5052 10.4056 10.17269'.contains(value.split('/')[0]),
  'http://api.springernature.com/metadata/json?q=doi:' + value + '&api_key=SPRINGER_API_KEY',
  null
)
```

#### Keywords

* Sofern in den Springer Nature-Daten Keywords vorhanden sind, werden diese in der Spalte **dc.subject.other[en]** gespeichert und ggf. bereits vorhandene Werte (via PubMed) überschrieben.
* Der Befehl folgt dieser Logik: Prüfen, ob Keywords enthalten sind (`if(isBlank(forEach(cells.SPRINGER.value.parseJson().facets,v, [...] ).join('')),`) -> sind Keywords *nicht* vorhanden (`isBlank`):
  * TRUE (*keine* Keywords vorhanden): Belasse den Wert des Feldes
  * FALSE (Keywords vorhanden): überschreibe vorhandenen Wert
* Gibt es mehrere Einträge, werden sie durch `||` getrennt.

**dc.subject.other[en]** -> Edit cells -> Transform...

```
if(isBlank(forEach(cells.SPRINGER.value.parseJson().facets,v,
    if(v.name == 'keyword',
      forEach(v.values,w,w.value).join('||'),
      null
    )
  ).join('')),
  value,
  forEach(cells.SPRINGER.value.parseJson().facets,v,
    if(v.name == 'keyword',
      forEach(v.values,w,w.value.strip()).join('||'),
      null
    )
  ).join('')
)
```

#### Abstract

* Sofern in den Springer Nature-Daten ein Abstract vorhanden ist, wird dieses in der Spalte **dc.description.abstract[en]** gespeichert und der ggf. bereits vorhandene Wert (via Crossref, PubMed, arXiv oder CORE) überschrieben.
* Den Abstracts ggf. vorangestellte Zusätze ("Abstract" bzw. "Zusammenfassung") werden entfernt.
* Gibt es mehrere Einträge, werden sie durch zwei Zeilenumbrüche getrennt (`\n\n`).

**dc.description.abstract[en]** -> Edit cells -> Transform...

```python
import json

j = json.loads(cells.SPRINGER.value)
records = j.get('records', [])

# Set mit allen Abstracts -> Set = Array, in dem jedes Element nur einmal vorkommen kann -> keine Dubletten
abstracts = set([i.get('abstract', '') for i in records])

# Viele (alle?) englischsprachige Abstracts beginnen mit 'Abstract', das sollen sie nicht.
abstracts = [i[8:] if i.startswith('Abstract') else i for i in abstracts]
# Viele (alle?) deutschsprachige Abstracts beginnen mit 'Zusammenfassung', das sollen sie nicht.
abstracts = [i[15:] if i.startswith('Zusammenfassung') else i for i in abstracts]

if abstracts:
    return '\n\n'.join(abstracts)
```


# JSON

**Achtung** API-Keys eintragen: "CORE_API_KEY" suchen und 2x ersetzen, "SPRINGER_API_KEY" suchen und 2x ersetzen.

```
[
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.description.abstract[en] using expression grel:cells.CR.value.parseJson().message.abstract",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.description.abstract[en]",
    "expression": "grel:cells.CR.value.parseJson().message.abstract",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/column-addition-by-fetching-urls",
    "description": "Create column PM_CONVERTER_API at index 26 by fetching URLs based on column dcterms.bibliographicCitation.doi using expression grel:'https://www.ncbi.nlm.nih.gov/pmc/utils/idconv/v1.0/?tool=oagreenserviceTUberlin&email=openaccess@ub.tu-berlin.de&format=json&ids=' + value",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "PM_CONVERTER_API",
    "columnInsertIndex": 26,
    "baseColumnName": "dcterms.bibliographicCitation.doi",
    "urlExpression": "grel:'https://www.ncbi.nlm.nih.gov/pmc/utils/idconv/v1.0/?tool=oagreenserviceTUberlin&email=openaccess@ub.tu-berlin.de&format=json&ids=' + value",
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
        "value": "*/*"
      }
    ]
  },
  {
    "op": "core/column-addition",
    "description": "Create column dc.identifier.pmid[en] at index 27 based on column PM_CONVERTER_API using expression grel:forEach(value.parseJson().records,v,\n  v.pmid\n).uniques().join('||')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dc.identifier.pmid[en]",
    "columnInsertIndex": 27,
    "baseColumnName": "PM_CONVERTER_API",
    "expression": "grel:forEach(value.parseJson().records,v,\n  v.pmid\n).uniques().join('||')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition-by-fetching-urls",
    "description": "Create column PM_MD_API at index 28 by fetching URLs based on column dc.identifier.pmid[en] using expression grel:if(isNonBlank(value),\n  'https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&retmode=xml&rettype=abstract&id=' + value,\n  null\n)",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "PM_MD_API",
    "columnInsertIndex": 28,
    "baseColumnName": "dc.identifier.pmid[en]",
    "urlExpression": "grel:if(isNonBlank(value),\n  'https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&retmode=xml&rettype=abstract&id=' + value,\n  null\n)",
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
        "value": "*/*"
      }
    ]
  },
  {
    "op": "core/column-addition",
    "description": "Create column PM_MD_API at index 29 based on column PM_MD_API using expression grel:forEach(value.parseHtml().select('abstracttext'),v,\n  v.htmlText()\n).uniques().join('||')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "PM_MD_API",
    "columnInsertIndex": 29,
    "baseColumnName": "PM_MD_API",
    "expression": "grel:forEach(value.parseHtml().select('abstracttext'),v,\n  v.htmlText()\n).uniques().join('||')",
    "onError": "set-to-blank"
  },
    {
    "op": "core/column-addition",
    "description": "Create column PM_abstract at index 29 based on column PM_MD_API using expression grel:forEach(value.parseHtml().select('abstracttext'),v,\n  v.htmlText()\n).uniques().join('||')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "PM_abstract",
    "columnInsertIndex": 29,
    "baseColumnName": "PM_MD_API",
    "expression": "grel:forEach(value.parseHtml().select('abstracttext'),v,\n  v.htmlText()\n).uniques().join('||')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition-by-fetching-urls",
    "description": "Create column BASE at index 26 by fetching URLs based on column dcterms.bibliographicCitation.doi using expression grel:'https://api.base-search.net/cgi-bin/BaseHttpSearchInterface.fcgi?func=PerformSearch&query=dcdoi:'\n+ value +\n'&fields=dcautoclasscode,dcclasscode,dcdeweyfull&format=json'",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "BASE",
    "columnInsertIndex": 26,
    "baseColumnName": "dcterms.bibliographicCitation.doi",
    "urlExpression": "grel:'https://api.base-search.net/cgi-bin/BaseHttpSearchInterface.fcgi?func=PerformSearch&query=dcdoi:'\n+ value +\n'&fields=dcautoclasscode,dcclasscode,dcdeweyfull&format=json'",
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
        "value": "*/*"
      }
    ]
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.subject.ddc[de] using expression jython:import json\n\nj = json.loads(cells.BASE.value)\nresponse = j.get('response', {})\ndocs = response.get('docs', {})\n\nreturn '||'.join(set(['BASE-Classifier: {}\\nBASE-Repo-Daten: {}'.format(\n                        '||'.join(i.get('dcautoclasscode', ['--'])),\n                        '||'.join(i.get('dcclasscode', ['--']))\n                    )\n                    for i in docs]))",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.subject.ddc[de]",
    "expression": "jython:import json\n\nj = json.loads(cells.BASE.value)\nresponse = j.get('response', {})\ndocs = response.get('docs', {})\n\nreturn '||'.join(set(['BASE-Classifier: {}\\nBASE-Repo-Daten: {}'.format(\n                        '||'.join(i.get('dcautoclasscode', ['--'])),\n                        '||'.join(i.get('dcclasscode', ['--']))\n                    )\n                    for i in docs]))",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/column-addition-by-fetching-urls",
    "description": "Create column CORE at index 26 by fetching URLs based on column dcterms.bibliographicCitation.doi using expression grel:'https://core.ac.uk/api-v2/articles/search/doi:\"' + value + '\"?apiKey=CORE_API_KEY'",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "CORE",
    "columnInsertIndex": 26,
    "baseColumnName": "dcterms.bibliographicCitation.doi",
    "urlExpression": "grel:'https://core.ac.uk/api-v2/articles/search/doi:\"' + value + '\"?apiKey=CORE_API_KEY'",
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
        "value": "*/*"
      }
    ]
  },
  {
    "op": "core/column-addition",
    "description": "Create column CORE_abstract at index 27 based on column CORE using expression grel:forEach(value.parseJson().data, v,\n  v.description\n).uniques().join('||')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "CORE_abstract",
    "columnInsertIndex": 27,
    "baseColumnName": "CORE",
    "expression": "grel:forEach(value.parseJson().data, v,\n  v.description\n).uniques().join('||')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column CORE_downloadUrl at index 27 based on column CORE using expression grel:forEach(value.parseJson().data, v,\n  v.downloadUrl\n).uniques().join('||')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "CORE_downloadUrl",
    "columnInsertIndex": 27,
    "baseColumnName": "CORE",
    "expression": "grel:forEach(value.parseJson().data, v,\n  v.downloadUrl\n).uniques().join('||')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition-by-fetching-urls",
    "description": "Create column ARXIV at index 26 by fetching URLs based on column dcterms.bibliographicCitation.doi using expression grel:'http://export.arxiv.org/api/query?search_query=doi:' + value",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "ARXIV",
    "columnInsertIndex": 26,
    "baseColumnName": "dcterms.bibliographicCitation.doi",
    "urlExpression": "grel:'http://export.arxiv.org/api/query?search_query=doi:' + value",
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
        "value": "*/*"
      }
    ]
  },
  {
    "op": "core/column-addition",
    "description": "Create column ARXIV_abstract at index 27 based on column ARXIV using expression grel:forEach(value.parseHtml().select('summary'),v,\n    v.htmlText()\n).uniques().join('||')",
    "engineConfig": {
      "mode": "row-based",
      "facets": [
        {
          "omitError": false,
          "expression": "value",
          "selectBlank": false,
          "selection": [
            {
              "v": {
                "v": "1",
                "l": "1"
              }
            }
          ],
          "selectError": false,
          "invert": false,
          "name": "arxiv_totalResults",
          "omitBlank": false,
          "type": "list",
          "columnName": "arxiv_totalResults"
        }
      ]
    },
    "newColumnName": "ARXIV_abstract",
    "columnInsertIndex": 27,
    "baseColumnName": "ARXIV",
    "expression": "grel:forEach(value.parseHtml().select('summary'),v,\n    v.htmlText()\n).uniques().join('||')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition-by-fetching-urls",
    "description": "Create column SPRINGER at index 26 by fetching URLs based on column dcterms.bibliographicCitation.doi using expression grel:if('10.26778 10.26777 10.7603 10.33283 10.1617 10.1245 10.1251 10.3858 10.1208\n    10.1114 10.3758 10.1186 10.1140 10.1361 10.1379 10.1381 10.1385 10.2165 \n    10.19150 10.1007 10.1013 10.1065 10.1023 10.1038 10.1057 10.4333 10.5822\n    10.5819 10.5052 10.4056 10.17269'.contains(value.split('/')[0]),\n  'http://api.springernature.com/metadata/json?q=doi:' + value + '&api_key=SPRINGER_API_KEY',\n  null\n)",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "SPRINGER",
    "columnInsertIndex": 26,
    "baseColumnName": "dcterms.bibliographicCitation.doi",
    "urlExpression": "grel:if('10.26778 10.26777 10.7603 10.33283 10.1617 10.1245 10.1251 10.3858 10.1208\n    10.1114 10.3758 10.1186 10.1140 10.1361 10.1379 10.1381 10.1385 10.2165 \n    10.19150 10.1007 10.1013 10.1065 10.1023 10.1038 10.1057 10.4333 10.5822\n    10.5819 10.5052 10.4056 10.17269'.contains(value.split('/')[0]),\n  'http://api.springernature.com/metadata/json?q=doi:' + value + '&api_key=SPRINGER_API_KEY',\n  null\n)",
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
        "value": "*/*"
      }
    ]
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.subject.other[en] using expression grel:if(isBlank(forEach(cells.SPRINGER.value.parseJson().facets,v,\n    if(v.name == 'keyword',\n      forEach(v.values,w,w.value).join('||'),\n      null\n    )\n  ).join('')),\n  value,\n  forEach(cells.SPRINGER.value.parseJson().facets,v,\n    if(v.name == 'keyword',\n      forEach(v.values,w,w.value.strip()).join('||'),\n      null\n    )\n  ).join('')\n)",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.subject.other[en]",
    "expression": "grel:if(isBlank(forEach(cells.SPRINGER.value.parseJson().facets,v,\n    if(v.name == 'keyword',\n      forEach(v.values,w,w.value).join('||'),\n      null\n    )\n  ).join('')),\n  value,\n  forEach(cells.SPRINGER.value.parseJson().facets,v,\n    if(v.name == 'keyword',\n      forEach(v.values,w,w.value.strip()).join('||'),\n      null\n    )\n  ).join('')\n)",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.description.abstract[en] using expression jython:import json\n\nj = json.loads(cells.SPRINGER.value)\nrecords = j.get('records', [])\n\n# Set mit allen Abstracts -> Set = Array, in dem jedes Element nur einmal vorkommen kann -> keine Dubletten\nabstracts = set([i.get('abstract', '') for i in records])\n\n# Viele (alle?) englisch-sprachige Abstracts beginnen mit 'Abstract', das sollen sie nicht.\nabstracts = [i[8:] if i.startswith('Abstract') else i for i in abstracts]\n# Viele (alle?) deutsch-sprachige Abstracts beginnen mit 'Zusammenfassung', das sollen sie nicht.\nabstracts = [i[15:] if i.startswith('Zusammenfassung') else i for i in abstracts]\n\nif abstracts:\n    return '\\n\\n'.join(abstracts)",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.description.abstract[en]",
    "expression": "jython:import json\n\nj = json.loads(cells.SPRINGER.value)\nrecords = j.get('records', [])\n\n# Set mit allen Abstracts -> Set = Array, in dem jedes Element nur einmal vorkommen kann -> keine Dubletten\nabstracts = set([i.get('abstract', '') for i in records])\n\n# Viele (alle?) englisch-sprachige Abstracts beginnen mit 'Abstract', das sollen sie nicht.\nabstracts = [i[8:] if i.startswith('Abstract') else i for i in abstracts]\n# Viele (alle?) deutsch-sprachige Abstracts beginnen mit 'Zusammenfassung', das sollen sie nicht.\nabstracts = [i[15:] if i.startswith('Zusammenfassung') else i for i in abstracts]\n\nif abstracts:\n    return '\\n\\n'.join(abstracts)",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  }
]
```
