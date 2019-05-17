# Felder Pflicht/optional

Inhalt: 
[Erläuterungen](#dokumentation-zum-code) || 
[JSON](#json)

## Intro

* Zahlreiche Metadatenfelder auf DepositOnce sind verpflichtend anzugeben; einige sind optional. Welches Feld belegt werden muss bzw. soll, ist vom Dokumententyp abhängig (Angabe in Spalte **dc.type[en]**).
* Um die manuelle Aufbereitung der Metadaten (vgl. [MDK2](/README.md#metadatenkontrolle-2-mdk2)) für den Import zu unterstützen, werden leere Felder gekennzeichnet:
  * **leere Pflichtfelder**: "PFLICHTFELD, bitte von Hand nachtragen"
  * **leere optionale Felder**: "OPTIONALES Feld, bitte prüfen ob Angabe möglich"


## Dokumentation zum Code

### Feld Dokumententyp

* Das Feld **dc.type[en]** kann nicht in Abhängigkeit vom Feld **dc.type[en]** gefüllt werden. Wenn kein Typ angegeben ist, soll immer "PFLICHTFELD, bitte von Hand nachtragen" zurückgegeben werden.  

**dc.type[en]** -> Edit cells -> Transform
```
if(isBlank(value),
  'PFLICHTFELD, bitte von Hand nachtragen',
  value
)
```

### Andere Metadatenfelder

* Um ein eigenes Skript für diesen Schritt zu erstellen, ist zunächst eine Analyse der Metadatenfelder des eigenen Repositoriums erforderlich &ndash; pro Dokumententyp: Kann das Feld vorkommen und wenn ja, ist es ein Pflicht- oder optionales Feld?
* Es gibt viele verschiedene Ansätze, um diese Kennzeichnung vorzunehmen. Wir haben uns für den folgenden Ansatz entschieden, da er relativ gut lesbar und editierbar ist.
* Jedes Metadatenfeld (jede Spalte) wird einzeln transformiert, wobei immer der gleiche Code genutzt wird. In diesem Code gibt es Einträge für die drei Dokumententypen, die berücksichtigt werden sollen ("Article" = Zeitschriftenartikel, "Book Part" = Buchkapitel, "Conference Object" = Beitrag in Konferenzband). Es kann dadurch jeweils die Zeile, welcher Wert für welchen Typen *nicht* zurückgegeben werden soll, einfach auskommentiert (oder gelöscht) werden (siehe [Beispiel](#beispiel)).
* `try: dc_type = cells['dc.type[en]'].value` -> Versuche, den Wert aus der Spalte **dc.type[en]** in der Variable `dc_type` zu speichern. Wenn ein Fehler auftritt (weil kein Typ angegeben ist), wird die Anweisung unter `except` ausgeführt.
* `except: return value` -> Wenn keine Angabe zum Typ vorhanden ist, kann eine Prüfung der einzelnen Felder nicht stattfinden. Daher werden lediglich vorhandene Werte zurückgegeben.
* `if value: return value` -> Wenn ein Wert in dem zu prüfenden Metadatenfeld vorhanden ist, wird der vorhandene Wert zurückgegeben.
* `else:` -> Wenn *kein* Wert vorhanden ist, findet die Prüfung statt.
* `if dc_type == 'Article'`, `elif dc_type == 'Book Part'`, `elif dc_type == 'Conference Object'` -> Nun wird getestet, ob der Typ "Article", "Book Part" oder "Conference Object" entspricht und es wird der entsprechende definierte Wert ("Pflichtfeld" oder "Optionales Feld") zurückgegeben (`return`).
* Für die Ausführung in OpenRefine ist pro Metadatenfeld (d.h. pro Spalte) folgende Aktion durchzuführen:

**$Spalte** -> Edit cells -> Transform

```python
try: dc_type = cells['dc.type[en]'].value
except: return value

if value:
    return value
else:
    if dc_type == 'Article':
        return('PFLICHTFELD, bitte von Hand nachtragen')
        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')
    elif dc_type == 'Book Part':
        return('PFLICHTFELD, bitte von Hand nachtragen')
        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')
    elif dc_type == 'Conference Object':
        return('PFLICHTFELD, bitte von Hand nachtragen')
        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')
```


### Beispiel

* In diesem Beispiel soll für den Typ Article der Wert "Pflichtfeld" und für die Typen Book Part und Conference Object der Wert "Optionales Feld" zurückgegeben werden; die jeweils anderen Rückgaben wurden deswegen auskommentiert. 
* Um eine Zeile auszukommentieren, ist eine Raute (`#`) vor `return` einzufügen.

```python
try: dc_type = cells['dc.type[en]'].value
except: return value

if value:
    return value
else:
    if dc_type == 'Article':
        return('PFLICHTFELD, bitte von Hand nachtragen')
        #return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')
    elif dc_type == 'Book Part':
#        return('PFLICHTFELD, bitte von Hand nachtragen')
        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')
    elif dc_type == 'Conference Object':
    #    return('PFLICHTFELD, bitte von Hand nachtragen')
        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')
```

## JSON


```
[
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dcterms.bibliographicCitation.doi using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dcterms.bibliographicCitation.doi",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.contributor.author using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.contributor.author",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.title[en] using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.title[en]",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.type[en] using expression grel:if(isBlank(value),'PFLICHTFELD, bitte von Hand nachtragen', value)",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.type[en]",
    "expression": "grel:if(isBlank(value),'PFLICHTFELD, bitte von Hand nachtragen', value)",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.date.issued using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.date.issued",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dcterms.bibliographicCitation.originalpublishername[en] using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dcterms.bibliographicCitation.originalpublishername[en]",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dcterms.bibliographicCitation.journaltitle[en] using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n#    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n#    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dcterms.bibliographicCitation.journaltitle[en]",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n#    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n#    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dcterms.bibliographicCitation.booktitle[en] using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dcterms.bibliographicCitation.booktitle[en]",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dcterms.bibliographicCitation.proceedingstitle[en] using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dcterms.bibliographicCitation.proceedingstitle[en]",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.identifier.issn using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.identifier.issn",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.identifier.eissn using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.identifier.eissn",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.identifier.isbn using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n#    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    if dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.identifier.isbn",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n#    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    if dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.type.version[en] using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.type.version[en]",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.rights.uri using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.rights.uri",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.description.abstract[en] using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.description.abstract[en]",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.description.abstract[de] using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.description.abstract[de]",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.subject.other[en] using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.subject.other[en]",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.subject.other[de] using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.subject.other[de]",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dcterms.bibliographicCitation.originalpublisherplace[en] using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dcterms.bibliographicCitation.originalpublisherplace[en]",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.description.sponsorship[en] using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.description.sponsorship[en]",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.subject.ddc[de] using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.subject.ddc[de]",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column tub.accessrights.openaire using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "tub.accessrights.openaire",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column tub.accessrights.dnb using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "tub.accessrights.dnb",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dc.language.iso using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dc.language.iso",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('PFLICHTFELD, bitte von Hand nachtragen')\n#        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dcterms.bibliographicCitation.pagestart using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dcterms.bibliographicCitation.pagestart",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dcterms.bibliographicCitation.pageend using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dcterms.bibliographicCitation.pageend",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dcterms.bibliographicCitation.articlenumber using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dcterms.bibliographicCitation.articlenumber",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dcterms.bibliographicCitation.volume using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dcterms.bibliographicCitation.volume",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Book Part':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n#        return('PFLICHTFELD, bitte von Hand nachtragen')\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dcterms.bibliographicCitation.issue using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dcterms.bibliographicCitation.issue",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Article':\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column dcterms.bibliographicCitation.editor using expression jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Book Part':\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "dcterms.bibliographicCitation.editor",
    "expression": "jython:try: dc_type = cells['dc.type[en]'].value\nexcept: return value\n\nif value:\n    return value\nelse:\n    if dc_type == 'Book Part':\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')\n    elif dc_type == 'Conference Object':\n        return('OPTIONALES Feld, bitte prüfen ob Angabe möglich')",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  }
]
```
