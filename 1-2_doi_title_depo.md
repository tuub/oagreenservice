# Dublettenkontrolle 1

Inhalt: [Erläuterungen](#dokumentation-zum-code) || [JSON](#json)


## Intro

Um einerseits effizient bei der Rechteprüfung vorzugehen und andererseits Dubletten im Repositorium zu verhindern, wird  geprüft, ob die gelisteten Publikationen bereits auf [DepositOnce (DepO)](https://depositonce.tu-berlin.de), dem institutionellen Repositorium der TU Berlin, verfügbar sind. Dieser Test erfolgt im Workflow zweimal, bei der [Metadatenakquise 1 (MD1)](/README.md/#metadatenakquise-teil-1-md1) und direkt vor dem [Import in DepositOnce](/README.md/#import-csv-vorbereiten).

Über die [REST-API](https://wiki.duraspace.org/display/DSDOC6x/REST+API) des Repositoriums wird geprüft, ob der Titel und/oder die vom Verlag vergebene DOI bereits im Repositorium vorhanden sind, damit keine Dubletten hochgeladen werden.


## Dokumentation zum Code

### Trim Whitespace

* Sollten DOI oder Titel [Whitespace](https://de.wikipedia.org/wiki/Leerraum) enthalten, wird dieser entfernt, da sonst die in den folgenden Schritten erzeugten URLs nicht funktionieren können.

**DOI** -> Edit cells -> Common transforms -> Trim leading and trailing whitespace

**Title** -> Edit cells -> Common transforms -> Trim leading and trailing whitespace


### Suche nach Titel in DepositOnce

* Es wird auf Basis des vorhandenen Titels (aus Crossref-Abfrage oder ursprünglicher Excel-Datei) eine Anfrage an die intere API gestellt und die Antwort ausgewertet. Dabei sind folgende Rückgabewerte möglich:
  - "Search produced no results" -> Titel ist nicht bekannt
  - "https://depositonce.tu-berlin.de/handle/11303/9317 || http://dx.doi.org/10.14279/depositonce-8390" -> Titel stimmt exakt mit einem vorhandenen Eintrag überein; für diesen werden die DepositOnce-Identifier zurückgegeben, was eine manuelle Verifikation der Übereinstimmung (sofern gewünscht) erleichert kann.
  - "$Anzahl items found" -> Titel stimmt mit mehr als einem vorhandenen Eintrag überein. Es ist in RP manuell zu prüfen, ob eine richtige Publikation zugeordnet wurde.
  - "ERROR" -> Anfrage konnte nicht korrekt verarbeitet werden.
* Häufige **Probleme**:
  - Beim URL-codieren des Titels kann es zu Problemen kommen, welche diesen Schritt scheitern lassen und zu der Rückgabe "ERROR" führen.
  - Es erfolgt ein einfacher Zeichenabgleich. Es kann daher zu Problemen kommen, insbesondere wenn ein Beitrag Haupttitel und Zusatztitel hat und diese unterschiedlich angesetzt sind in den Vergleichswerten.
  - *Fazit*: Dem Dubletten-Check auf Basis des Titels ist nur bedingt zu trauen.

**Title** -> Edit column -> Add column based on this column -> **DEPO_TITLE**

```{python}
import urllib
import urllib2
import json

title = value

try:
    # Title must be encoded.
    query_title = urllib.quote(title)
except KeyError:
    return 'ERROR'

url =  ''.join(['https://depositonce.tu-berlin.de/',
                'rest/filtered-items',
                '?query_field%5B%5D=dc.title.*',
                '&query_op%5B%5D=contains',
                '&query_val%5B%5D=',
                query_title,
                '&expand=metadata'])

try:
    response = urllib2.urlopen(url)
    response_json = json.loads(response.read().decode('utf-8'))
    anzahl_treffer = response_json.get('item-count')
except:
    return 'ERROR'

if anzahl_treffer == 0:
    return 'Search produced no results'
elif anzahl_treffer == 1:
    md_fields = [ i for i in response_json['items'][0]['metadata'] ]
    fields_return = []
    for field in md_fields:
        if field['key'] == 'dc.identifier.uri':
            fields_return.append(field['value'])
    return ' || '.join(fields_return)
elif anzahl_treffer >= 1:
    return '{} items found'.format(anzahl_treffer)
```

### Suche nach DOI in DepositOnce

* Es wird auf Basis der DOI eine Anfrage an die intere API gestellt. Der API-Aufruf erfolgt mehrfach, da die DSpace-API case sensitive ist: Die DOI "10.123456/beispiel" wird nicht als gleich mit "10.123456/BEISPIEL" erkannt. Daher werden maximal drei Fälle getestet, wobei der DOI-Wert nur für die Anfrage transformiert wird: (1) mit Original-Input-DOI, (2) DOI in Kleinbuchstaben (wenn sich diese von der Original-Input-DOI unterscheidet), (3) DOI in Großbuchstaben (wenn sich diese von der Original-Input-DOI unterscheidet).
* Die Antworten auf die drei Abfragen werden ausgewertet, dabei sind folgende Rückgabewerte möglich:
  - - "Search produced no results" -> DOI ist nicht bekannt
  - "https://depositonce.tu-berlin.de/handle/11303/9317 || http://dx.doi.org/10.14279/depositonce-8390" -> DOI stimmt exakt mit einem vorhandenen Eintrag überein; für diesen werden die DepositOnce-Identifier zurückgegeben, was eine manuelle Verifikation der Übereinstimmung (sofern gewünscht) erleichert kann.
  - "Search produced no results || https://depositonce.tu-berlin.de/handle/11303/9317 || http://dx.doi.org/10.14279/depositonce-8390" -> Für eine der Anfragen mit transformierter DOI wurde keine Übereinstimmung ermittelt, aber für eine andere Schreibweise gibt es eine Übereinstimmung &ndash; Fazit: DOI stimmt exakt mit einem vorhandenen Eintrag überein.
  - "$Anzahl items found" -> DOI stimmt mit mehr als einem vorhandenen Eintrag überein. Es ist in RP manuell zu prüfen, ob eine richtige Publikation zugeordnet wurde. (Das sollte eigentlich nicht passieren und kam bei Tests bisher nicht vor, aber theoretisch wäre es möglich.)
  - "ERROR" -> Anfrage konnte nicht korrekt verarbeitet werden.


**DOI** -> Edit column -> Add column based on this column -> **DEPO_DOI**

```{python}
import urllib2
import json

doi = value

def depoCall_jsonParsing(doi):
    url =  ''.join(['https://depositonce.tu-berlin.de/',
                'rest/filtered-items',
                '?query_field%5B%5D=dcterms.bibliographicCitation.doi',
                '&query_op%5B%5D=contains',
                '&query_val%5B%5D=',
                doi,
                '&expand=metadata'])
    
    try:
        req = urllib2.Request(url=url)
        r = urllib2.urlopen(req)
        j = json.loads(r.read().decode('utf-8'))
        item_count = j.get('item-count')
    except:
        return 'ERROR'
        
    if item_count == 0:
        return 'Search produced no results'
    elif item_count == 1:
        return ' || '.join([i.get('value', '') 
                            for i in j['items'][0]['metadata']
                            if i.get('key') == 'dc.identifier.uri'])
    elif item_count >= 1:
        return '{} items found'.format(anzahl_treffer)


result = depoCall_jsonParsing(doi)
if doi != doi.lower():
    result_lower = depoCall_jsonParsing(doi.lower())
if doi != doi.upper():
    result_upper = depoCall_jsonParsing(doi.upper()) 

results = set([result, result_lower, result_upper])
return ' || '.join(results)
```

## JSON

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
    "op": "core/text-transform",
    "description": "Text transform on cells in column Title using expression value.trim()",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "Title",
    "expression": "value.trim()",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/column-addition",
    "description": "Create column DEPO_TITLE at index 8 based on column Title using expression jython:import urllib\nimport urllib2\nimport json\n\ntitle = value\n\ntry:\n    query_title = urllib.quote(title)\nexcept KeyError:\n    return 'ERROR'\n\nurl =  ''.join(['https://depositonce.tu-berlin.de/',\n                'rest/filtered-items',\n                '?query_field%5B%5D=dc.title.*',\n                '&query_op%5B%5D=contains',\n                '&query_val%5B%5D=',\n                query_title,\n                '&expand=metadata'])\n\ntry:\n    request = urllib2.Request(url=url)\n    response = urllib2.urlopen(request)\n    response_json = json.loads(response.read().decode('utf-8'))\n    anzahl_treffer = response_json.get('item-count')\nexcept:\n    anzahl_treffer = None\n\nif anzahl_treffer == 0:\n    return 'Search produced no results'\nelif anzahl_treffer == 1:\n    md_fields = [ i for i in response_json['items'][0]['metadata'] ]\n    fields_return = []\n    for field in md_fields:\n        if field['key'] == 'dc.identifier.uri':\n            fields_return.append(field['value'])\n    return ' || '.join(fields_return)\nelif anzahl_treffer >= 1:\n    return '{} items found'.format(anzahl_treffer)\n\nif anzahl_treffer == None:\n    return 'ERROR'",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "DEPO_TITLE",
    "columnInsertIndex": 1,
    "baseColumnName": "Title",
    "expression": "jython:import urllib\nimport urllib2\nimport json\n\ntitle = value\n\ntry:\n    query_title = urllib.quote(title)\nexcept KeyError:\n    return 'ERROR'\n\nurl =  ''.join(['https://depositonce.tu-berlin.de/',\n                'rest/filtered-items',\n                '?query_field%5B%5D=dc.title.*',\n                '&query_op%5B%5D=contains',\n                '&query_val%5B%5D=',\n                query_title,\n                '&expand=metadata'])\n\ntry:\n    request = urllib2.Request(url=url)\n    response = urllib2.urlopen(request)\n    response_json = json.loads(response.read().decode('utf-8'))\n    anzahl_treffer = response_json.get('item-count')\nexcept:\n    anzahl_treffer = None\n\nif anzahl_treffer == 0:\n    return 'Search produced no results'\nelif anzahl_treffer == 1:\n    md_fields = [ i for i in response_json['items'][0]['metadata'] ]\n    fields_return = []\n    for field in md_fields:\n        if field['key'] == 'dc.identifier.uri':\n            fields_return.append(field['value'])\n    return ' || '.join(fields_return)\nelif anzahl_treffer >= 1:\n    return '{} items found'.format(anzahl_treffer)\n\nif anzahl_treffer == None:\n    return 'ERROR'",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column DEPO_DOI at index 1 based on column DOI using expression jython:import urllib2\nimport json\n\ndoi = value\n\ndef depoCall_jsonParsing(doi):\n    url =  ''.join(['https://depositonce.tu-berlin.de/',\n                'rest/filtered-items',\n                '?query_field%5B%5D=dcterms.bibliographicCitation.doi',\n                '&query_op%5B%5D=contains',\n                '&query_val%5B%5D=',\n                doi,\n                '&expand=metadata'])\n    \n    try:\n        req = urllib2.Request(url=url)\n        r = urllib2.urlopen(req)\n        j = json.loads(r.read().decode('utf-8'))\n        item_count = j.get('item-count')\n    except:\n        return 'ERROR'\n        \n    if item_count == 0:\n        return 'Search produced no results'\n    elif item_count == 1:\n        return ' || '.join([i.get('value', '') \n                            for i in j['items'][0]['metadata']\n                            if i.get('key') == 'dc.identifier.uri'])\n    elif item_count >= 1:\n        return '{} items found'.format(anzahl_treffer)\n\n    \n\nresult = depoCall_jsonParsing(doi)\nresult_lower = depoCall_jsonParsing(doi.lower())\nresult_upper = depoCall_jsonParsing(doi.upper()) \n# result = depo_call(doi)\n# if result:\n#     itemCount = json.loads(result)\n# else:\n#     itemCount = None\n\n# if doi != doi.lower():\n#     result_lower = depo_call(doi.lower())\n# else:\n#     result_lower = None\n    \n# if doi != doi.upper():\n#     result_upper = depo_call(doi.upper())\n# else:\n#     result_upper = None\n    \n    \n# print result\n# print result_lower\n# print result_upper\n# print\nresults = set([result, result_lower, result_upper])\n# results = [i for i in results if i != 'Search produced no results']\nreturn ' || '.join(results)",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "DEPO_DOI",
    "columnInsertIndex": 1,
    "baseColumnName": "DOI",
    "expression": "jython:import urllib2\nimport json\n\ndoi = value\n\ndef depoCall_jsonParsing(doi):\n    url =  ''.join(['https://depositonce.tu-berlin.de/',\n                'rest/filtered-items',\n                '?query_field%5B%5D=dcterms.bibliographicCitation.doi',\n                '&query_op%5B%5D=contains',\n                '&query_val%5B%5D=',\n                doi,\n                '&expand=metadata'])\n    \n    try:\n        req = urllib2.Request(url=url)\n        r = urllib2.urlopen(req)\n        j = json.loads(r.read().decode('utf-8'))\n        item_count = j.get('item-count')\n    except:\n        return 'ERROR'\n        \n    if item_count == 0:\n        return 'Search produced no results'\n    elif item_count == 1:\n        return ' || '.join([i.get('value', '') \n                            for i in j['items'][0]['metadata']\n                            if i.get('key') == 'dc.identifier.uri'])\n    elif item_count >= 1:\n        return '{} items found'.format(anzahl_treffer)\n\n    \n\nresult = depoCall_jsonParsing(doi)\nresult_lower = depoCall_jsonParsing(doi.lower())\nresult_upper = depoCall_jsonParsing(doi.upper()) \n# result = depo_call(doi)\n# if result:\n#     itemCount = json.loads(result)\n# else:\n#     itemCount = None\n\n# if doi != doi.lower():\n#     result_lower = depo_call(doi.lower())\n# else:\n#     result_lower = None\n    \n# if doi != doi.upper():\n#     result_upper = depo_call(doi.upper())\n# else:\n#     result_upper = None\n    \n    \n# print result\n# print result_lower\n# print result_upper\n# print\nresults = set([result, result_lower, result_upper])\n# results = [i for i in results if i != 'Search produced no results']\nreturn ' || '.join(results)",
    "onError": "set-to-blank"
  }
]
```
