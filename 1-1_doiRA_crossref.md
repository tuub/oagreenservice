# DOI-Resolver und Crossref 1

Inhalt: 
[Erläuterungen DOI-Resolver](#doi-resolver) || 
[Erläuterungen Crossref](#crossref) || 
[JSON für DOI-Resolver und Crossref](#json)

## DOI-Resolver

### Intro

* Über `https://doi.org/doiRA/{DOI}` lässt sich prüfen, ob eine DOI und wenn ja, von welcher DOI-Registry-Agency sie registriert wurde. 
* Crossref liefert nur Metadaten für von Crossref registrierte DOIs. Bleibt die spätere Crossref-Anfrage erfolglos, liefert die Anfrage an den DOI-Resolver Hinweise dazu, warum für eine DOI keine Metadaten von Crossref geliefert werden.
* vgl. [Dokumentation zentraler DOI-Resolver](https://www.doi.org/doi_handbook/5_Applications.html#5.2).

* Beispiel für Antwort **nicht valide DOI**:

```
[
  {
    "DOI": "10.666/1234567890",
    "status": "DOI does not exist"
  }
]
```
* Beispiel für Antwort **valide DOI**:

```
[
  {
   "DOI": "10.1145/2637002.2637038",
    "RA": "Crossref"
  }
]
```


### Dokumentation zum Code


#### Trim Whitespace

* Sollte die DOI [Whitespace](https://de.wikipedia.org/wiki/Leerraum) enthalten, wird dieser entfernt, da sonst die in den folgenden Schritten erzeugten URLs nicht funktionieren können. Diese Änderung wird in der Spalte **DOI** gespeichert.

**DOI** -> Edit cells -> Common transforms -> Trim leading and trailing whitespace


#### API Call

* Von der API wird entweder ein Status zur DOI oder der Name der DOI-Registrierungsagentur ("DOI-RA", z.B. Crossref, DataCite) zurückgegeben. Sofern der API-Aufruf nicht korrekt verarbeitet werden konnte, wird eine Fehlermeldung ausgegeben.
* Mögliche Einträge in den OpenRefine-Daten nach dieser Abfrage sind:
  - "ERROR" -> API-Aufruf konnte nicht korrekt verarbeitet werden
  - "DOI does not exist" -> angefragte DOI nicht registriert (mögliche Gründe: DOI syntaktisch inkorrekt ist Ausgangsdaten; Fehler bei Registrierung)
  - "Crossref" (oder "DataCite" etc.) -> Name der Registrierungsagentur

**DOI** -> Edit column -> Add column based on this column -> **doiRA**

```python
import urllib2
import json

try:
    url = 'https://doi.org/doiRA/' + value.strip()
    r = urllib2.urlopen(url).read()
    j = json.loads(r)
except:
    return 'ERROR'

status = '||'.join(set([i.get('status', '') for i in j]))
if status:
    return status

ra = '||'.join(set([i.get('RA', '') for i in j]))
if ra:
    return ra
```

## Crossref

### Intro

* Crossref, die für wissenschaftliche Publikationen namhafter Verlage wichtigste DOI-Registrierungsagentur, stellt eine REST API zur Verfügung, über die u.a. Metadaten zu einzelnen Publikationen abgerufen werden können.
* Beispielanfrage an Crossref-API: <https://api.crossref.org/works/10.1177/0022002711420971> bzw. <https://api.crossref.org/works/10.1177/0022002711420971?mailto=openaccess@ub.tu-berlin.de> 
* vgl. [Dokumentation Crossref API](https://github.com/CrossRef/rest-api-doc)
* vgl. [Dokumentation Crossref API-Felder](https://github.com/CrossRef/rest-api-doc/blob/master/api_format.md)
* Im ersten Schritt wird nur ein Grundset an Metadaten abgefragt, die JSON-Daten werden in der Spalte **CR** geschrieben, einzelne benötigte Angaben werden in den unten dokumentierten Spalten gespeichert. 
* Warum werden an dieser Stelle nicht alle Crossref-Daten vollständig ausgewertet?
  * Für Rechteprüfung ist nur geringe Anzahl von Feldern erforderlich (weniger Felder = bessere Übersichtlichkeit).
  * Für Autor\*innennamen sind manuelle Korrekturen erforderlich, die bei einer späteren Abfrage bzw. Datentransformation nachgenutzt werden (vgl. [MD2](/2-1_crosscite_crossref2.md#crosscite)); es ist also in jedem Fall ein zweiter Schritt für das Auslesen von Crossref-Daten erforderlich.
* Die Python-Pakete [habanero](https://github.com/sckott/habanero) und [crossrefapi](https://github.com/fabiobatalha/crossrefapi) lassen sich leider nicht mit Jython in OpenRefine nutzen.


### Dokumentation zum Code

#### Trim Whitespace

* Sollte die DOI [Whitespace](https://de.wikipedia.org/wiki/Leerraum) enthalten, wird dieser entfernt, da sonst die in den folgenden Schritten erzeugten URLs nicht funktionieren können.

**DOI** -> Edit cells -> Common transforms -> Trim leading and trailing whitespace


#### Aufruf Crossref API

* Crossref bittet darum, bei Anfragen eine E-Mail-Adresse zu übermitteln; dies wird mit schnelleren Servern [belohnt](https://github.com/CrossRef/rest-api-doc#good-manners--more-reliable-service).
* Angabe DOI wird bereinigt (Entfernen von Leerzeichen bzw. nicht-sichtbaren Zeichen)
* JSON-Daten von Crossref werden in Spalte **CR** gespeichert &ndash; sie können damit später (in MD2) nachgenutzt werden, ohne erneut die API abzufragen.

**DOI** -> Edit column -> Add column based on this column -> **CR**

```python
import urllib2

if cells.doiRA.value == 'Crossref':
    try:
        doi = cells.DOI.value.strip()
        url = 'https://api.crossref.org/works/' + doi + '?mailto=openaccess@ub.tu-berlin.de'
        response = urllib2.urlopen(url)
        return response.read().decode('unicode_escape')
    except:
        return None
else:
    return None
```

#### TU-Affiliation

* Mit der folgenden Abfrage werden Affiliationsangaben aus Crossref darauf analysiert, ob eine TU-Affiliation vorhanden ist. Dafür wird folgender Suchterm umgesetzt: `'Berlin' UND ('TU' ODER 'Techn')`.  Mit diesem einfachen Ansatz wird mitunter auch eine TU-Affiliation fälschlich zugewiesen (z.B. auch für `'Technikmuseum Berlin'`).
* Eine komplexere Struktur zum Abgleich von Affiliationsangaben ist denkbar und technisch umsetzbar; wird in Anbetracht der unzähligen möglichen Namensformen und Abkürzungen (inkl. potentieller Schreibfehler) aber immer fehleranfällig bleiben.
* Im Rahmen der Rechteprüfung ist die Affiliationsangabe zu verifizieren (dabei sind falsche Zuordnungen zu korrigieren) bzw. manuell zu ermitteln.

**TU-Affiliation** -> Edit cells -> Transform...

```python
import json

# Affiliations, die mitgegeben wurden aus der Spalte TU-Affiliation
aff_tu = value

# Affiliations aus Crossref
j = json.loads(cells.CR.value)
msg = j.get('message', {})
authors = msg.get('author', {})
aff_cr = []
for author in authors:
    for affiliation in author.get('affiliation', []):
        affili_name = affiliation.get('name', [])
        if 'Berlin' in affili_name:
            if 'TU' in affili_name or 'Techn' in affili_name:
            # alternativer Ansatz, um weniger false positives zu bekommen (noch nicht getestet):
            # if 'TU' in affili_name or ('Techn' in affili_name and ('Univ' in affili_name or 'Inst' in affili_name)):
                aff_cr.append('{f}, {g}: {a}'.format(
                                                    f=author.get('family', ''),
                                                    g=author.get('given', ''),
                                                    a=affili_name))

# Rückgabe von beiden
return 'Input: {tu} ----- CR: {cr}'.format(
                                        tu=aff_tu,
                                        cr='||'.join(aff_cr)
                                        )
```

#### Date Published Online

* In Crossref gibt es verschiedene Datumsangaben, häufig wird unterschieden zwischen Daten für Online- und Printausgabe. Es werden für beide Werte neue Spalten angelegt.
* Ein Abweichen von Jahresangaben ist an dieser Stelle nicht unüblich. Im Schritt Rechteprüfung sollte festgelegt werden, welches das maßgebliche Datum ist, um die ggf. erforderliche Embargofrist zu bestimmen.

**CR** -> Edit column -> Add column based on this column -> **DATE PUBLISHED ONLINE**
```
forEach(value.parseJson().message['published-online']['date-parts'][0],v,v).join('-')
```

#### Date Published Print

**CR** -> Edit column -> Add column based on this column -> **DATE PUBLISHED PRINT**
```
forEach(value.parseJson().message['published-print']['date-parts'][0],v,v).join('-')
```

#### Author

##### Author: Trennzeichen

* **Annahme**: Namen der Autor\*innen liegen vor (etwa aus Bibtex- oder RIS-Datei), und zwar im Format `Nachname1, Vorname1; Nachname2, Vorname2 Mittelname2; etc.`.
* Ziel: Trennzeichen zwischen mehreren Autor\*innen vereinheitlichen (`||` ist das von DSpace erwartete Trennzeichen für mehrere Einträge im gleichen Metadatenfeld)

**Author** -> Edit cells -> Transform... 

```
value.split('; ').join('||')
```

Alternativ:
```
value.replace('; ', '||')
```

##### Author: Crossref auslesen

* Gibt es Angaben zu Autor\*innen in Crossref, wird der vorhandene Wert überschrieben &ndash; andernfalls bleibt der ursprüngliche Wert erhalten.

**Author** -> Edit cells -> Transform...

* **Wichtig**: `On Error: keep original`

```
forEach(cells['CR'].value.parseJson().message.author,v,
    v.family + ', ' + v.given
).join('||')
```

#### Title

* Gibt es Angaben zum Titel in Crossref, wird der vorhandene Wert überschrieben &ndash; andernfalls bleibt der ursprüngliche Wert erhalten.

**Title** -> Edit cells -> Transform...

* **Wichtig**: `On Error: keep original`

```
forEach(cells.CR.value.parseJson().message.title,v,v).join('||')
```


#### Type

* Sofern es in den Crossref-Daten eine Angabe zum Typ gibt, wird diese übernommen, wobei drei Typen unserem Vokabular angepasst werden (z.B. "journal-article" -> "Article").
* Sofern in den Crossref-Daten die Typ-Angabe fehlt, bleibt der ggf. vorhandene Wert der Spalte **Type** erhalten.


**Type** -> Edit cells -> Transform...

```
if(isBlank(cells.CR.value.parseJson().message.type),
  value,
  if(cells.CR.value.parseJson().message.type == 'journal-article',
    'Article',
    if(cells.CR.value.parseJson().message.type == 'book-chapter',
      'Book Part',
      if(cells.CR.value.parseJson().message.type == 'proceedings-article',
        'Conference Object',
        cells.CR.value.parseJson().message.type
      )
    )
  )
)
```

#### dcterms.bibliographicCitation.journaltitle[en]

* Sofern als Typ ein Zeitschriftenartikel erkannt wurde, wird der Wert des Feldes **container-title** in eine neue Spalte **dcterms.bibliographicCitation.journaltitle[en]** geschrieben.

**CR** ->  Edit column ->Add column based on this column -> **dcterms.bibliographicCitation.journaltitle[en]**

```
if(value.parseJson().message.type == 'journal-article',
  value.parseJson().message['container-title'].join('||'),
  null
)
```

#### CR_ContainerTitle

* Sofern der Typ NICHT einem Zeitschriftenartikel entspricht, wird der Wert des Crossref-Feldes **container-title** in eine neue Spalte **CR_ContainerTitle** geschrieben.
* Hintergrund: In Crossref werden sowohl Angaben zu Titeln von Büchern/Konferenzbänden als auch zu Schriftenreihen in einem Feld erfasst, wobei die Reihenfolge unterschiedlich sein kann (d.h. manchmal, aber nicht immer wird zuerst Buchtitel und als zweites die Schriftenreihe angegeben). Die korrekte Zuordnung von Titel des Buches bzw. Konferenzbandes muss daher in MDK2 manuell erfolgen.

**CR** -> Edit column -> Add column based on this column -> **CR_ContainerTitle**

```
if(value.parseJson().message.type == 'journal-article',
  null,
  forEach(value.parseJson().message['container-title'],
    v,
    v).join('||')
)
```

#### Publisher

* Wenn in den Crossref-Daten kein Verlag enthalten ist, wird der Wert in der Spalte **Publisher** bestehen gelassen, sonst wird er mit den Crossref-Daten überschrieben.

**Publisher** -> Edit cells -> Transform...

```
if(isBlank(cells.CR.value.parseJson().message.publisher),
  value,
  cells.CR.value.parseJson().message.publisher
)
```


#### CR_LICENSE_URLs

* Neue Spalte für Lizenz-URLs angelegen; *alle* vorhandenen Werte von Crossref übernehmen und ergänzen um Angabe, auf welche Art von Inhalt ([vgl. Vokabular in Crossref-Doku](https://github.com/CrossRef/rest-api-doc/blob/master/api_format.md#license)) sich die Lizenzangabe bezieht.
* Gibt es mehrere Einträge, werden sie durch `||` getrennt.

**CR** -> Edit column -> Add column based on this column -> **CR_LICENSE_URLs**

```
forEach(value.parseJson().message.license,v,
'content-version: ' + v['content-version'] + ' -- ' + v.URL
).join('||')
```

* Beispiel 1 für Ausgabe: `content-version: vor -- http://pubs.acs.org/page/policy/authorchoice_termsofuse.html`
* Beispiel 2 für Ausgabe: `content-version: tdm -- https://www.elsevier.com/tdm/userlicense/1.0/||content-version: vor -- http://creativecommons.org/licenses/by-nc-nd/4.0/`


#### CR_ISSN_ALL

* Es wird eine neue Spalte angelegt, in der alle in Crossref vorhandenen ISSN-Angaben gespeichert werden.

**CR** -> Edit column -> Add column based on this column -> **CR_ISSN_ALL**

```python
import json

d = json.loads(cells.CR.value)
msg = d.get('message', [])

issn_all = set([])

for issn in msg.get('ISSN', []):
    issn_all.add(issn)

for issn in msg.get('issn-type', []):
    issn_all.add(issn.get('value'))

return '||'.join(issn_all)
```

#### ISSN

* Crossref-Angaben zur ISSN der Printausgabe werden in Spalte **ISSN** übernommen.
* Gibt es mehrere Einträge, werden sie durch `||` getrennt.
* In der Spalte ggf. vorhandene Werte bleiben dann erhalten, wenn beim Auslesen der Crossref-Daten kein Fehler auftritt.

**ISSN** -> Edit cells -> Transform...

* **Wichtig**: `On Error: keep original` 

```
forEach(cells.CR.value.parseJson().message['issn-type'],
  v,
  if(v.type == 'print',
    v.value,
    null)
).join('||')
```

#### eISSN

* Crossref-Angaben zur ISSN der Onlineausgabe werden in Spalte **eISSN** übernommen.
* Gibt es mehrere Einträge, werden sie durch `||` getrennt.
* In der Spalte ggf. vorhandene Werte bleiben dann erhalten, wenn beim Auslesen der Crossref-Daten kein Fehler auftritt.

**eISSN** -> Edit cells -> Transform...

* **Wichtig**: `On Error: keep original` 

```
forEach(cells.CR.value.parseJson().message['issn-type'],
  v,
  if(v.type == 'electronic',
    v.value,
    null)
).join('||')
```

#### ISBN

* Alle Crossref-Angaben zur ISBN werden werden in Spalte **ISBN** übernommen.
* Gibt es mehrere Einträge, werden sie durch `||` getrennt.


**ISBN**  -> Edit cells -> Transform...

* **Wichtig**: `On Error: keep original` 

```python
import json

d = json.loads(cells.CR.value)
msg = d.get('message', [])

isbn_all = set([])

for isbn in msg.get('ISBN', []):
    isbn_all.add(isbn)

for isbn in msg.get('isbn-type', []):
    isbn_all.add(isbn.get('value'))

return '||'.join(isbn_all)
```

#### CR_FUNDER

* Es wird eine neue Spalte angelegt, in der alle in Crossref vorhandenen Funding-Angaben (d.h. Angaben zu Projektförderung) gespeichert werden. Subfelder werden in Reihenfolge "$Fördereinrichtung : $Fördernummer" erfasst.
* Gibt es mehrere Einträge, werden sie durch `||` getrennt.

**CR** -> Edit column -> Add column based on this column -> **CR_FUNDER**

```
forEach(value.parseJson().message.funder, v,
    v.name + ' : ' + forEach(v.award, aw, aw).join(', ')
).join('||')
```

* Beispiel 1 für Ausgabe: `Deutscher Akademischer Austauschdienst: 91541142`
* Beispiel 2 für Ausgabe: `Einstein Foundation Berlin, Germany: ||National Centre for Research and Development, Poland: ERA-NET-TRANSPORT-III/2/2014`


#### DOI-Link

* Es wird eine neue Spalte angelegt, in dem die DOI als HTTPS-Link erfasst wird.

**DOI** -> Edit column -> Add column based on this column -> **DOI-Link**

```
'https://doi.org/' + value
```

#### dc.title.subtitle[en]

* Neue Spalte für englischen Zusatztitel anlegen, Werte aus den Crossref-Daten übernehmen.

**CR** -> Edit column -> Add column based on this column -> **dc.title.subtitle[en]**

```
forEach(value.parseJson().message.subtitle,v,v).join('||')
```

#### dc.title[de]

* Neue leere Spalte für den deutschen Haupttitel angelegen.

**DOI** -> Edit column -> Add column based on this column -> dc.title[de]

```
null
```

#### dc.title.subtitle[de]

* Neue, leere Spalte für den deutschen Zusatztitel angelegen.

**DOI** -> Edit column -> Add column based on this column -> dc.title.subtitle[de]

```
null
```

#### dcterms.bibliographicCitation.booktitle[en]

* Neue, leere Spalte für den englischen Titel des Buches angelegen.

**DOI** -> Edit column -> Add column based on this column -> dcterms.bibliographicCitation.booktitle[en]

```
null
```

#### dcterms.bibliographicCitation.proceedingstitle[en]

* Neue, leere Spalte für den englischsprachigen Titel des Konferenzbandes angelegen.

**DOI** -> Edit column -> Add column based on this column -> dcterms.bibliographicCitation.proceedingstitle[en]

```
null
```

#### dc.rights.uri

* Neue, leere Spalte für Lizenzangabe anlegen.

**DOI** -> Edit column -> Add column based on this column -> dc.rights.uri

```
null
```


# JSON

```
[
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column DOI using expression value.trim()",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "DOI",
    "expression": "value.trim()",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/column-addition",
    "description": "Create column doiRA at index 8 based on column DOI using expression jython:import urllib2\nimport json\n\nif cells.DOI.value:\n    try:\n        url = 'https://doi.org/doiRA/' + cells.DOI.value.strip()\n        response = urllib2.urlopen(url)\n        doira = response.read()\n        doira_json = json.loads(doira[2:-2])\n    except:\n        return 'ERROR'\nelse:\n    return 'NO DOI'\n\nstatus = doira_json.get('status')\nif status:\n    return status\n\nra = doira_json.get('RA')\nif ra:\n    return ra",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "doiRA",
    "columnInsertIndex": 8,
    "baseColumnName": "DOI",
    "expression": "jython:import urllib2\nimport json\n\nif cells.DOI.value:\n    try:\n        url = 'https://doi.org/doiRA/' + cells.DOI.value.strip()\n        response = urllib2.urlopen(url)\n        doira = response.read()\n        doira_json = json.loads(doira[2:-2])\n    except:\n        return 'ERROR'\nelse:\n    return 'NO DOI'\n\nstatus = doira_json.get('status')\nif status:\n    return status\n\nra = doira_json.get('RA')\nif ra:\n    return ra",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column CR at index 8 based on column DOI using expression jython:import urllib2\n\nif cells.doiRA.value == 'Crossref':\n    try:\n        doi = cells.DOI.value.strip()\n        url = 'https://api.crossref.org/works/' + doi + '?mailto=openaccess@ub.tu-berlin.de'\n        response = urllib2.urlopen(url)\n        return response.read().decode('unicode_escape')\n    except:\n        return None\nelse:\n    return None",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "CR",
    "columnInsertIndex": 8,
    "baseColumnName": "DOI",
    "expression": "jython:import urllib2\n\nif cells.doiRA.value == 'Crossref':\n    try:\n        doi = cells.DOI.value.strip()\n        url = 'https://api.crossref.org/works/' + doi + '?mailto=openaccess@ub.tu-berlin.de'\n        response = urllib2.urlopen(url)\n        return response.read().decode('unicode_escape')\n    except:\n        return None\nelse:\n    return None",
    "onError": "set-to-blank"
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column TU-Affiliation using expression jython:import json\n\n# Affiliations, die mitgegeben wurden aus dem Feld TU-Affiliation\naff_tu = value\n\n# Affiliations aus Crossref\nd = json.loads(cells.CR.value)\nmsg = d.get('message', {})\nauthors = msg.get('author', {})\naff_cr = []\nfor author in authors:\n    for affiliation in author.get('affiliation', []):\n        affili_name = affiliation.get('name', [])\n        if 'Berlin' in affili_name:\n            if 'TU' in affili_name or 'Techn' in affili_name:\n                aff_cr.append('%s, %s : %s' % (author.get('family'),\n                                                    author.get('given'),\n                                                    affili_name))\n\n# Rückgabe von beiden\nreturn 'Input: %s ----- CR: %s' % (aff_tu, ' --- '.join(aff_cr))",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "TU-Affiliation",
    "expression": "jython:import json\n\n# Affiliations, die mitgegeben wurden aus dem Feld TU-Affiliation\naff_tu = value\n\n# Affiliations aus Crossref\nd = json.loads(cells.CR.value)\nmsg = d.get('message', {})\nauthors = msg.get('author', {})\naff_cr = []\nfor author in authors:\n    for affiliation in author.get('affiliation', []):\n        affili_name = affiliation.get('name', [])\n        if 'Berlin' in affili_name:\n            if 'TU' in affili_name or 'Techn' in affili_name:\n                aff_cr.append('%s, %s : %s' % (author.get('family'),\n                                                    author.get('given'),\n                                                    affili_name))\n\n# Rückgabe von beiden\nreturn 'Input: %s ----- CR: %s' % (aff_tu, ' --- '.join(aff_cr))",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/column-addition",
    "description": "Create column DATE PUBLISHED ONLINE at index 9 based on column CR using expression grel:forEach(value.parseJson().message['published-online']['date-parts'][0],v,v).join('-')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "DATE PUBLISHED ONLINE",
    "columnInsertIndex": 9,
    "baseColumnName": "CR",
    "expression": "grel:forEach(value.parseJson().message['published-online']['date-parts'][0],v,v).join('-')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column DATE PUBLISHED PRINT at index 9 based on column CR using expression grel:forEach(value.parseJson().message['published-print']['date-parts'][0],v,v).join('-')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "DATE PUBLISHED PRINT",
    "columnInsertIndex": 9,
    "baseColumnName": "CR",
    "expression": "grel:forEach(value.parseJson().message['published-print']['date-parts'][0],v,v).join('-')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column Author using expression grel:value.split('; ').join('||')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "Author",
    "expression": "grel:value.split('; ').join('||')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column Author using expression grel:forEach(cells['CR'].value.parseJson().message.author,v,\n(v.family + ', ' + v.given)\n).join('||')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "Author",
    "expression": "grel:forEach(cells['CR'].value.parseJson().message.author,v,\n(v.family + ', ' + v.given)\n).join('||')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column Title using expression grel:forEach(cells.CR.value.parseJson().message.title,v,v).join('||')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "Title",
    "expression": "grel:forEach(cells.CR.value.parseJson().message.title,v,v).join('||')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column Type using expression grel:if(isBlank(cells.CR.value.parseJson().message.type),\n  value,\n  if(cells.CR.value.parseJson().message.type == 'journal-article',\n    'Article',\n    if(cells.CR.value.parseJson().message.type == 'book-chapter',\n      'Book Part',\n      if(cells.CR.value.parseJson().message.type == 'proceedings-article',\n        'Conference Object',\n        cells.CR.value.parseJson().message.type\n      )\n    )\n  )\n)",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "Type",
    "expression": "grel:if(isBlank(cells.CR.value.parseJson().message.type),\n  value,\n  if(cells.CR.value.parseJson().message.type == 'journal-article',\n    'Article',\n    if(cells.CR.value.parseJson().message.type == 'book-chapter',\n      'Book Part',\n      if(cells.CR.value.parseJson().message.type == 'proceedings-article',\n        'Conference Object',\n        cells.CR.value.parseJson().message.type\n      )\n    )\n  )\n)",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/column-addition",
    "description": "Create column dcterms.bibliographicCitation.journaltitle[en] at index 9 based on column CR using expression grel:if(value.parseJson().message.type == 'journal-article',\n  value.parseJson().message['container-title'].join('||'),\n  null\n)",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dcterms.bibliographicCitation.journaltitle[en]",
    "columnInsertIndex": 9,
    "baseColumnName": "CR",
    "expression": "grel:if(value.parseJson().message.type == 'journal-article',\n  value.parseJson().message['container-title'].join('||'),\n  null\n)",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column CR_ContainerTitle at index 9 based on column CR using expression grel:if(value.parseJson().message.type == 'journal-article',\n  null,\n  forEach(value.parseJson().message['container-title'],\n    v,\n    v).join('||')\n)",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "CR_ContainerTitle",
    "columnInsertIndex": 9,
    "baseColumnName": "CR",
    "expression": "grel:if(value.parseJson().message.type == 'journal-article',\n  null,\n  forEach(value.parseJson().message['container-title'],\n    v,\n    v).join('||')\n)",
    "onError": "set-to-blank"
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column Publisher using expression grel:if(isEmptyString(cells.CR.value),\n  value,\n  cells.CR.value.parseJson().message.publisher\n)",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "Publisher",
    "expression": "grel:if(isEmptyString(cells.CR.value),\n  value,\n  cells.CR.value.parseJson().message.publisher\n)",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/column-addition",
    "description": "Create column CR_LICENSE_URLs at index 9 based on column CR using expression grel:forEach(value.parseJson().message.license,v,\n'content-version: ' + v['content-version'] + ' -- ' + v.URL\n).join('||')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "CR_LICENSE_URLs",
    "columnInsertIndex": 9,
    "baseColumnName": "CR",
    "expression": "grel:forEach(value.parseJson().message.license,v,\n'content-version: ' + v['content-version'] + ' -- ' + v.URL\n).join('||')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column CR_ISSN_ALL at index 9 based on column CR using expression jython:import json\n\nd = json.loads(cells.CR.value)\nmsg = d.get('message', [])\n\nissn_all = set([])\n\nfor issn in msg.get('ISSN', []):\n    issn_all.add(issn)\n\nfor issn in msg.get('issn-type', []):\n    issn_all.add(issn.get('value'))\n\nreturn '||'.join(issn_all)",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "CR_ISSN_ALL",
    "columnInsertIndex": 9,
    "baseColumnName": "CR",
    "expression": "jython:import json\n\nd = json.loads(cells.CR.value)\nmsg = d.get('message', [])\n\nissn_all = set([])\n\nfor issn in msg.get('ISSN', []):\n    issn_all.add(issn)\n\nfor issn in msg.get('issn-type', []):\n    issn_all.add(issn.get('value'))\n\nreturn '||'.join(issn_all)",
    "onError": "set-to-blank"
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column ISSN using expression grel:forEach(cells.CR.value.parseJson().message['issn-type'],\n  v,\n  if(v.type == 'print',\n    v.value,\n    null)\n).join('||')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "ISSN",
    "expression": "grel:forEach(cells.CR.value.parseJson().message['issn-type'],\n  v,\n  if(v.type == 'print',\n    v.value,\n    null)\n).join('||')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column eISSN using expression grel:forEach(cells.CR.value.parseJson().message['issn-type'],\n  v,\n  if(v.type == 'electronic',\n    v.value,\n    null)\n).join('||')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "eISSN",
    "expression": "grel:forEach(cells.CR.value.parseJson().message['issn-type'],\n  v,\n  if(v.type == 'electronic',\n    v.value,\n    null)\n).join('||')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column ISBN using expression jython:import json\n\nd = json.loads(cells.CR.value)\nmsg = d.get('message', [])\n\nisbn_all = set([])\n\nfor isbn in msg.get('ISBN', []):\n    isbn_all.add(isbn)\n\nfor isbn in msg.get('isbn-type', []):\n    isbn_all.add(isbn.get('value'))\n\nreturn '||'.join(isbn_all)",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "ISBN",
    "expression": "jython:import json\n\nd = json.loads(cells.CR.value)\nmsg = d.get('message', [])\n\nisbn_all = set([])\n\nfor isbn in msg.get('ISBN', []):\n    isbn_all.add(isbn)\n\nfor isbn in msg.get('isbn-type', []):\n    isbn_all.add(isbn.get('value'))\n\nreturn '||'.join(isbn_all)",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/column-addition",
    "description": "Create column CR_FUNDER at index 9 based on column CR using expression grel:forEach(value.parseJson().message.funder, v, v.name + ' : ' + forEach(v.award, aw, aw).join(', ')).join(' --- ')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "CR_FUNDER",
    "columnInsertIndex": 9,
    "baseColumnName": "CR",
    "expression": "grel:forEach(value.parseJson().message.funder, v, v.name + ' : ' + forEach(v.award, aw, aw).join(', ')).join(' --- ')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column DOI-Link at index 8 based on column DOI using expression grel:'https://doi.org/' + value",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "DOI-Link",
    "columnInsertIndex": 8,
    "baseColumnName": "DOI",
    "expression": "grel:'https://doi.org/' + value",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dc.title.subtitle[en] at index 10 based on column CR using expression grel:forEach(value.parseJson().message.subtitle,v,v).join('||')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dc.title.subtitle[en]",
    "columnInsertIndex": 10,
    "baseColumnName": "CR",
    "expression": "grel:forEach(value.parseJson().message.subtitle,v,v).join('||')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dc.title[de] at index 8 based on column DOI using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dc.title[de]",
    "columnInsertIndex": 8,
    "baseColumnName": "DOI",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dc.title.subtitle[de] at index 8 based on column DOI using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dc.title.subtitle[de]",
    "columnInsertIndex": 8,
    "baseColumnName": "DOI",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dcterms.bibliographicCitation.booktitle[en] at index 8 based on column DOI using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dcterms.bibliographicCitation.booktitle[en]",
    "columnInsertIndex": 8,
    "baseColumnName": "DOI",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dcterms.bibliographicCitation.proceedingstitle[en] at index 8 based on column DOI using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dcterms.bibliographicCitation.proceedingstitle[en]",
    "columnInsertIndex": 8,
    "baseColumnName": "DOI",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column dc.rights.uri at index 8 based on column DOI using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "dc.rights.uri",
    "columnInsertIndex": 8,
    "baseColumnName": "DOI",
    "expression": "grel:null",
    "onError": "set-to-blank"
  }
]
```
