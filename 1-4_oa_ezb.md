# OA-EZB

Inhalt: 
[Erläuterungen](#intro) || 
[JSON](#json)


## Intro

Im Rahmen von [Allianz- und Nationallizenzen](https://www.nationallizenzen.de) wurden Open-Access-Rechte für die lizenznehmenden Institutionen verhandelt, das heißt Verlage räumen mitunter den Institutionen direkt Rechte zur Zweitveröffentlichung ein. Diese Rechte gehen in der Regel über die vom Verlag im Rahmen der allgemeinen Policy eingeräumten Rechte für Autor\*innen hinaus. Bislang gestaltete sich die Recherche danach, ob für einen bestimmten Artikel Open-Access-Rechte vorliegen, als
aufwändig (vgl. [Thomas & Stadler (2016)](https://doi.org/10.1515/bd-2016-0006), [Voigt (2016)](https://doi.org/10.5281/zenodo.322574)).

Das DFG-geförderte Projekt [OA-EZB](https://ezb.ur.de/services/oa-ezb) hatte zum Ziel, Angaben zu diesen besonderen Open-Access-Rechten in der Elektronischen Zeitschriftenbibliothek (EZB) zu verankern und maschinell abfragbar zu machen. Resultat ist die OA-EZB-Schnittstelle, welche die in der EZB hinterlegten Open-Access-Rechte zur Veröffentlichung von Volltexten gemäß Allianz-, National- oder Konsortiallizenzen zur Verfügung stellt und für die keine Registrierung erforderlich ist. Um die OA-Berechtigung für die eigene Institution zu ermitteln, ist die EZB-Kennung anzugeben.
Die Abfrage kann auf Artikel- oder Zeitschriftenebene erfolgen; Daten werden in den Formaten JSON oder XML zurückgegeben.

## Dokumentation zum Code

### Aufruf OA-EZB API

* vgl. [Informationen zu OA-EZB &amp; Dokumentation der Schnittstelle](https://ezb.ur.de/services/oa-ezb)
* Die Schnittstelle gibt Daten prinzipiell in zwei Formaten aus. Der Standard ist XML, durch Angabe des Parameters `&format=application/json` erfolgt die Rückgabe im JSON-Format.
* Unabhängig davon, ob die Abfrage über GREL oder Jython gestellt wird, wird die Antwort der REST API in OpenRefine zunächst so dargestellt: `f\u-1fcr Verlags-PDFs w\u00fcnschen k\u00f6nnten`. Nach dem Parsen mit GREL oder Python werden Umlaute und Sonderzeichen in den meisten Fällen korrekt encoded.
* Im Folgenden werden zwei Ansätze für die Abfrage (Option a) mit GREL und b) mit Jython) beschrieben. Ausgangspunkt für beide ist die Anfrage auf Basis der Spalte **DOI** und unter Angabe der EZB-Kennung. Es wird eine neue Spalte **OAEZB** angelegt, in der die JSON-Daten von OA-EZB gespeichert werden.

#### Option a) Abfrage mit GREL:

**DOI** -> Edit column -> Add column by fetching URLs ... -> **OAEZB**

* **Wichtig**: OA-EZB-Kennung der eigenen Bibliothek eintragen (siehe `bibid=`)!
* **Wichtig**: `Throttle delay 1000 ms`
* **Wichtig**: `On Error: store error`

```
'https://ezb.ur.de/api/oa_rights?bibid=TUBB&doi=' + trim(cells.DOI.value) + '&format=application/json'
```

#### Option b) Abfrage mit Jython:

**DOI** -> Edit column -> Add column by fetching URLs ... -> **OAEZB**

* **Wichtig**: OA-EZB-Kennung der eigenen Bibliothek eintragen (siehe `bibid=`)!

```python
import urllib2

doi = cells.DOI.value.strip()

try:
    url = 'https://ezb.ur.de/api/oa_rights?bibid=TUBB&doi={}&format=application/json'.format(doi.strip())
    return urllib2.urlopen(url=url).read()
except:
    return 'Error'
```


### Angaben zum Status auslesen

* Es wird eine neue Spalte **OAEZB_STATUS** angelegt, in der die Werte der OA-EZB-Felder `state` und `message` gespeichert werden. Die Subfelder werden in Reihenfolge "$Statuscode : $Beschreibung" erfasst.
* Mit `.uniques()` werden etwaige dublette Angaben entfernt.
* Gibt es mehrere Einträge, werden sie durch `||` getrennt.

**OAEZB** -> Edit column -> Add column based on this column -> **OAEZB_STATUS**

```
forEach(value.parseJson(), v,
  v.state + ' : ' + v.message
).uniques().join(' || ')
```

* Beispiel 1 für Ausgabe: `O : OA-Verwertungsrechte vorhanden`
* Beispiel 2 für Ausgabe: `2 : Für Ihre Anfrage sind keine OA-Rechte in der EZB hinterlegt`


### Angaben zu OA-Rechten

* Es wird eine neue Spalte **OAEZB_RECHTSGRUNDLAGE** angelegt, in der Werte aus verschiedenen OA-EZB-Feldern gespeichert werden.
* Die Subfelder werden in der folgenden Reihenfolge und Form ausgegeben: "`oa_agreement` (EZB-Paket-ID: `oa_package_name` OA-Jg. `oa_first_date` - `oa_last_date` -> `oa_pub_authority`: `oa_archivable_version` (`oa_source`) in: `oa_repositories` (Embargo: `oa_embargo_months`)"
* Gibt es mehrere Einträge, werden sie durch `-----` getrennt.
* In einem zweiten Schritt werden nicht benötigte Angaben gelöscht.

#### Angaben zu möglichen OA-Rechten auslesen

**OAEZB** -> Edit column -> Add column based on this column -> **OAEZB_RECHTSGRUNDLAGE**

```python
import json
oaezb = json.loads(value)

out = set([])

for i in oaezb:
    i_string = '{} (EZB-Paket-ID: {}) OA-Jg. {}-{} -> {}: {} ({}) in: {} (Embargo: {} Monate)'.format(
        i.get('oa_agreement', ''),
        i.get('oa_package_name', ''),
        i.get('oa_first_date', ''),
        i.get('oa_last_date', ''),
        i.get('oa_pub_authority', '').upper(),
        ' oder '.join([v for v in i.get('oa_archivable_version', {}).itervalues()]),
        ' oder '.join([v for v in i.get('oa_source', {}).itervalues()]),
        ' oder '.join([v for v in i.get('oa_repositories', {}).itervalues()]),
        i.get('oa_embargo_months', '')
    )
    out.add(i_string)

return ' ----- '.join(out)
```

* Beispiel für Ausgabe: `Deutsche Nationallizenzen: Walter de Gruyter Online-Zeitschriften Archiv (EZB-Paket-ID: NALIG) OA-Jg. 1861-2017 -> AUTOR UND INSTITUTION: Verlags-PDF (Automatische Artikellieferung durch Verlag oder Direktdownload aus der Datenbank) in: Repositorium nach Wahl (Embargo: 0 Monate)`


#### Werte zu möglichen OA-Rechten transformieren

* Sind keine OA-Rechte hinterlegt, findet sich nun in der Spalte **OAEZB_RECHTSGRUNDLAGE** die folgende Angabe: `' (EZB-Paket-ID: ) OA-Jg. - -> :  () in:  (Embargo:  Monate)'` 
* Diese Angabe ist wenig hilfreich für die Rechteprüfung und wird daher gelöscht:

**OAEZB_RECHTSGRUNDLAGE** -> Edit cells -> Transform...

```
if(value == ' (EZB-Paket-ID: ) OA-Jg. - -> :  () in:  (Embargo:  Monate)',
  null,
  value
)
```


### OAEZB_ANMERKUNGEN

* Ggf. sind weitere Anmerkungen zu den OA-Rechten in der EZB hinterlegt; diese werden in einer neuen Spalte **OAEZB_ANMERKUNGEN** erfasst.
* Dabei werden die Felder `remarks_de` und `remarks_en` ausgelesen; mehrere Einträge werden durch ` ----- ` getrennt.

**OAEZB** -> Edit column -> Add column based on this column -> **OAEZB_ANMERKUNGEN**

```python
import json

rem_out = set([])
for i in json.loads(value):
  rem_out.add(i.get('remarks_de'))
  rem_out.add(i.get('remarks_en'))
return ' ----- '.join([i.strip() for i in rem_out if i])
```

* Beispiel für Ausgabe: `Auf Wunsch und Anfrage zusätzlich Bereitstellung der Artikel auf dem Dokumentenserver der HU Berlin durch den Verlag möglich. Kontakt beim Verlag: Ben Ashcroft, Director Global Sales (ben.ashcroft@degruyter.com). Die an der Nationallizenz teilnehmenden Institutionen bekommen nach vereinbarter Frequenz die Dokumente mittels des SWORD-Protokolls automatisch übermittelt (Kontakt: Katharina.Rach@degruyter.com). Voraussetzung für diese Leistung ist die Identifizierbarkeit der Artikel über eine Zugehörigkeitskennung zu der teilnehmenden Institution in den Metadaten der Artikel.`


## JSON


```
[
  {
    "op": "core/column-addition-by-fetching-urls",
    "description": "Create column OAEZB at index 13 by fetching URLs based on column DOI using expression grel:'https://ezb.ur.de/api/oa_rights?bibid=TUBB&doi=' + trim(cells.DOI.value) + '&format=application/json'",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "OAEZB",
    "columnInsertIndex": 13,
    "baseColumnName": "DOI",
    "urlExpression": "grel:'https://ezb.ur.de/api/oa_rights?bibid=TUBB&doi=' + trim(cells.DOI.value) + '&format=application/json'",
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
    "description": "Create column OAEZB_STATUS at index 14 based on column OAEZB using expression grel:forEach(value.parseJson(), v,\n  v.state + ' : ' + v.message\n).uniques().join(' || ')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "OAEZB_STATUS",
    "columnInsertIndex": 14,
    "baseColumnName": "OAEZB",
    "expression": "grel:forEach(value.parseJson(), v,\n  v.state + ' : ' + v.message\n).uniques().join(' || ')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column OAEZB_RECHTSGRUNDLAGE at index 14 based on column OAEZB using expression jython:import json\n\nreturn ' ----- '.join(set(\n    ['{} (EZB-Paket-ID: {}) OA-Jg. {}-{} -> {}: {} ({}) in: {} (Embargo: {} Monate)'.format(\n        i.get('oa_agreement', ''),\n        i.get('oa_package_name', ''),\n        i.get('oa_first_date', ''),\n        i.get('oa_last_date', ''),\n        i.get('oa_pub_authority', '').upper(),\n        ' oder '.join([v for v in i.get('oa_archivable_version', {}).itervalues()]),\n        ' oder '.join([v for v in i.get('oa_source', {}).itervalues()]),\n        ' oder '.join([v for v in i.get('oa_repositories', {}).itervalues()]),\n        i.get('oa_embargo_months', '')\n        )\n     for i in json.loads(value)]))",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "OAEZB_RECHTSGRUNDLAGE",
    "columnInsertIndex": 14,
    "baseColumnName": "OAEZB",
    "expression": "jython:import json\n\nreturn ' ----- '.join(set(\n    ['{} (EZB-Paket-ID: {}) OA-Jg. {}-{} -> {}: {} ({}) in: {} (Embargo: {} Monate)'.format(\n        i.get('oa_agreement', ''),\n        i.get('oa_package_name', ''),\n        i.get('oa_first_date', ''),\n        i.get('oa_last_date', ''),\n        i.get('oa_pub_authority', '').upper(),\n        ' oder '.join([v for v in i.get('oa_archivable_version', {}).itervalues()]),\n        ' oder '.join([v for v in i.get('oa_source', {}).itervalues()]),\n        ' oder '.join([v for v in i.get('oa_repositories', {}).itervalues()]),\n        i.get('oa_embargo_months', '')\n        )\n     for i in json.loads(value)]))",
    "onError": "set-to-blank"
  },
  {
    "op": "core/text-transform",
    "description": "Text transform on cells in column OAEZB_RECHTSGRUNDLAGE using expression grel:if(value == ' (EZB-Paket-ID: ) OA-Jg. - -> :  () in:  (Embargo:  Monate)',\n  null,\n  value\n)",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "columnName": "OAEZB_RECHTSGRUNDLAGE",
    "expression": "grel:if(value == ' (EZB-Paket-ID: ) OA-Jg. - -> :  () in:  (Embargo:  Monate)',\n  null,\n  value\n)",
    "onError": "keep-original",
    "repeat": false,
    "repeatCount": 10
  },
  {
    "op": "core/column-addition",
    "description": "Create column OAEZB_ANMERKUNGEN at index 14 based on column OAEZB using expression jython:import json\n\nrem_out = set([])\nfor i in json.loads(value):\n  rem_out.add(i.get('remarks_de'))\n  rem_out.add(i.get('remarks_en'))\nreturn ' ----- '.join([i.strip() for i in rem_out if i])",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "OAEZB_ANMERKUNGEN",
    "columnInsertIndex": 14,
    "baseColumnName": "OAEZB",
    "expression": "jython:import json\n\nrem_out = set([])\nfor i in json.loads(value):\n  rem_out.add(i.get('remarks_de'))\n  rem_out.add(i.get('remarks_en'))\nreturn ' ----- '.join([i.strip() for i in rem_out if i])",
    "onError": "set-to-blank"
  }
]
```
