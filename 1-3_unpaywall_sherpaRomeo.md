# Unpaywall und SHERPA/RoMEO

Inhalt: 
[Erläuterungen Unpaywall](#unpaywall) || 
[Erläuterungen SHERPA/RoMEO](#sherparomeo) || 
[Erläuterungen Spalten anlegen](#spalten-neu-anlegen) || 
[JSON](#json)

## Unpaywall

### Intro

* Die Abfrage an Unpaywall erfolgt, um den OA-Status einer Publikation zu ermitteln und ggf. andere auf Repositorien verfügbare Versionen zu identifizieren, die für eine Zweitveröffentlichung auf DepositOnce nachgenutzt werden könnten.
* [Unpaywall](https://unpaywall.org) "harvest[s] Open Access content from over 50,000 publishers and repositories, and make[s] it easy to find, track, and use."
* Unpaywall stellt eine REST API zur Verfügung, siehe
  * [API-Dokumentation](https://unpaywall.org/products/api) 
  * [Übersicht über vorhandene Felder](https://unpaywall.org/data-format).
* Daten werden im JSON-Format ausgegeben.
* 100.000 Abfragen am Tag an die REST-Schnittstelle sind frei. Eine Registrierung ist nicht erforderlich, jedoch soll bei jeder Abfrage die E-Mail-Adresse übermittelt werden.
* Beispielanfragen:
  * OA-Artikel in OA-Zeitschrift <https://api.unpaywall.org/v2/10.1371/journal.pone.0198249?email=INSERT@YOUR.EMAIL>
  * OA-Version via Repository <https://api.unpaywall.org/v2/10.1080/00987913.2008.10765150?email=INSERT@YOUR.EMAIL>
  * Verschiedene OA-Versionen via Repositories: <https://api.unpaywall.org/v2/10.1038/nature12373?email=INSERT@YOUR.EMAIL>


### Dokumentation zum Code

#### Aufruf Unpaywall API

* Sofern eine DOI vorhanden ist (vgl. `if(isNotNull(value)`), erfolgt die Abfrage auf Basis dieser DOI.
* Unpaywall verlangt, dass bei Anfragen eine E-Mail-Adresse übermittelt wird.

**DOI** -> Edit column -> Add column by fetching URLs -> **UPW**

* **Wichtig**: `Throttle delay 1000 ms`
* **Wichtig**: `On Error: store error`
* **Wichtig**: E-Mail-Adresse anpassen

```
if(isNotNull(value),
  'https://api.unpaywall.org/v2/' + value + '?email=INSERT@YOUR.EMAIL',
  null
)
```


#### OA-Status

* Es wird das Unpaywall-Feld `is_oa` ausgelesen und der darin enthaltene Boolesche Wert in einer neuen Spalte **UPW: IS OA?** gespeichert.
* Es sind also zwei Rückgabewerte möglich:
  - "true" -> Unpaywall konnte eine OA verfügbare Variante zuordnen
  - "false" -> Unpaywall konnte *keine* OA verfügbare Variante zuordnen

**UPW** -> Edit column -> Add column based on this column -> **UPW: IS OA?**

```
value.parseJson().is_oa
```

#### Status Zeitschrift

* Es wird das Unpaywall-Feld `journal_is_in_doaj` ausgelesen &ndash; also die Angabe dazu, ob es sich um eine reine OA-Zeitschrift handelt, die im [Directory of Open Access Journals (DOAJ)](https://doaj.org/) gelistet ist &ndash; und der darin enthaltene Boolesche Wert in einer neuen Spalte **UPW: J DOAJ?** gespeichert.

**UPW** -> Edit column -> Add column based on this column -> **UPW: J DOAJ?**

```
value.parseJson().journal_is_in_doaj
```

#### Beste OA-Variante

* Unpaywall identifiziert mitunter verschiedene OA-Varianten eines Artikels; nach eigenen Vorgaben wird die "beste" OA-Variante identifizert und in dem (verschachtelten) Unpaywall-Feld `best_oa_location` angegeben. 
* Einzelne Subfelder des Unpaywall-Feldes `best_oa_location` werden ausgelesen und in einer neuen Spalte **UPW: bestOA** gespeichert.
* Die ausgewählten Subfelder werden in der folgenden Reihenfolge und Form ausgegeben: "Evidence: `evidence` -- URL: `url` -- Host Type: `host_type` -- License: `license`"

**UPW** -> Edit column -> Add column based on this column -> **UPW: bestOA**

```
'Evidence: ' + value.parseJson()['best_oa_location'].evidence
+ ' -- URL: ' + value.parseJson()['best_oa_location'].url
+ ' -- Host Type: ' + value.parseJson()['best_oa_location']['host_type']
+ ' -- License: ' + value.parseJson()['best_oa_location']['license']
```

* Beispiel 1 für Ausgabe: `Evidence: oa repository (via OAI-PMH title and last author match) -- URL: https://depositonce.tu-berlin.de//bitstream/11303/7235/1/rose.2010.015.pdf -- Host Type: repository -- License: null`
* Beispiel 2 für Ausgabe: `Evidence: open (via page says license) -- URL: http://www.degruyter.com/downloadpdf/j/znb.2010.65.issue-3/znb-2010-0304/znb-2010-0304.xml -- Host Type: publisher -- License: cc-by-nc-nd`


#### OA-Version(en) über Repository

* Unpaywall identifiziert mitunter verschiedene OA-Varianten eines Artikels und listet relevante Angaben zu allen Varianten in dem (verschachtelten) Unpaywall-Feld `oa_locations`.
* An dieser Stelle sollen die OA-Varianten, die über OA-Repositorien verfügbar sind, identifiziert werden.
* Sofern als "Host Type" die Angabe "repository" enthalten ist (vgl. `if(v['host_type'] == 'repository'`), werden dazu pro Variante einzelne Subfelder des (verschachtelten) Unpaywall-Feldes `oa_locations` ausgelesen und in einer neuen Spalte **UPW: OA-Repos** gespeichert.
* Gibt es mehrere OA-Varianten, werden die verschiedenen Einträge durch `||` getrennt.
* Mit `.uniques()` werden etwaige dublette Angaben entfernt.
* **Achtung** Die bisherige Beobachtung ist, dass für Einträge zu OA-Varianten auf Repositorien die Unpaywall-Angabe zur Version (etwa "submittedVersion") nicht zuverlässig ist. Die Angabe zur Version sollte daher manuell verifiziert werden, sofern der Wunsch besteht, vorhandene Dateien nachzunutzen.

**UPW** -> Edit column -> Add column based on this column -> **UPW: OA-Repos**

```
forEach(value.parseJson()['oa_locations'], v,
  if(v['host_type'] == 'repository',
    'Landing page: ' + v['url_for_landing_page']
    + ' -- PDF-Link: ' + v['url_for_pdf']
    + ' -- License: ' + v['license']
    + ' -- version: ' + v['version'],
    
    null
    )
).uniques().join(' || ')
```

* Beispiel für Ausgabe: `Landing page: https://depositonce.tu-berlin.de//handle/11303/7216 -- PDF-Link: https://depositonce.tu-berlin.de//bitstream/11303/7216/1/bmt.2010.055.pdf -- License: null -- version: submittedVersion || Landing page: http://edoc.mpg.de/564579 -- PDF-Link: http://edoc.mpg.de/get.epl?fid=97003&did=564579&ver=0 -- License: null -- version: submittedVersion`


## SHERPA/RoMEO

### Intro

* Die Abfrage an SHERPA/RoMEO (SR) erfolgt, um Angaben zu Zweitveröffentlichungspolicies von Verlagen zu beziehen. SR stellt eine REST API zur Verfügung.
  * [About RoMEO](http://www.sherpa.ac.uk/romeo/about.php)
  * [FAQ](http://www.sherpa.ac.uk/romeo/faq.php)
  * API [Short documentation](http://sherpa.ac.uk/romeo/apimanual.php)
  * API [Full technical description (PDF)](http://sherpa.ac.uk/romeo/SHERPA%20RoMEO%20API%20V-2-9%202013-11-25.pdf)
* Ohne Registrierung können pro Tag maximal 500 Anfragen gestellt werden. Für unbegrenzte Anzahl an Anfragen wird ein API-Key benötigt, vgl. [API User Registry](http://sherpa.ac.uk/romeo/apiregistry.php). 
* Daten werden im XML-Format ausgegeben.
* Prinzipiell sind verschiedene Arten von Anfragen möglich; wir nutzen zwei Arten &ndash; auf Basis einer ISSN oder des Verlagsnamens.
  * ISSN: `http://www.sherpa.ac.uk/romeo/api29.php?issn={ISSN}`
  * Verlag: `http://www.sherpa.ac.uk/romeo/api29.php?pub={Publisher}&qtype=exact` (Leerzeichen im Verlagsnamen werden mit `%20`maskiert; statt einer exakten Suche sind auch ähnliche Suchen möglich (`&qtype=all` oder `&qtype=any`).)


### Dokumentation zum Code

#### Aufruf SHERPA/RoMEO API

* Die Abfrage soll wenn möglich auf Basis einer ISSN erfolgen, da dies die zuverlässigsten Ergebnisse bringt. Dafür werden die Spalten **CR_ISSN_ALL**, **ISSN** und **eISSN** auf Werte geprüft:
  * Die Spalte **CR_ISSN_ALL** kann mehr als eine ISSN enthalten -> Wenn die Spalte nicht leer ist, werden die ersten neun Zeichen genutzt.
  * Die Spalten **ISSN** und **eISSN** können jeweils maximal eine ISSN enthalten.
* Ist in keiner Spalte eine ISSN enthalten, erfolgt die Abfrage auf Basis des Verlagsnamens in der Spalte **Publisher**.
* Auf Basis des Verlagsnamens sind weniger zuverlässige Ergebnisse zu erwarten. Für Publikationen ohne DOI empfiehlt es sich daher, ISSN-Angaben bei der [Projektaufnahme](/README.md/#projektaufnahme) manuell zu recherchieren und zu ergänzen.

**CR_ISSN_ALL** -> Edit column -> Add column by fetching URLs... -> **SR**

* **Wichtig**: API-Key für SHERPA/RoMEO eintragen (`SR_API_KEY` mit [eigenem API-Key](http://sherpa.ac.uk/romeo/apiregistry.php) ersetzen)
* **Wichtig**: `Language: Python / Jython`

```python
import urllib2
from urllib import quote

base_url = 'http://www.sherpa.ac.uk/romeo/api29.php'

# Versuche, die ISSN aus dem Feld CR_ISSN_ALL zu nehmen
# Nimm nur die ersten neun Zeichen, falls mehr als eine ISSN dort steht
try:
    issn = cells.CR_ISSN_ALL.value[0:9]
except:
    issn = None

# Wenn keine ISSN in CR_ISN_ALL gefunden wurde, versuche die aus dem Feld ISSN zu nehmen
if not issn:
    try:
        issn = cells.ISSN.value
    except:
        issn = None

# Wenn noch keine ISSN gefunden wurde, versuche die aus dem Feld eISSN zu nehmen
if not issn:
    try:
        issn = cells.eISSN.value
    except:
        issn = None

# Wenn eine ISSN gefunden wurde, mache mit dieser den SR-API-Call
if issn:
    url = base_url + '?issn={}&ak=SR_API_KEY'.format(issn)
    return urllib2.urlopen(url).read()

# Wenn nirgendwo eine ISSN gefunden wurde, versuche einen API-Call mit dem Verlagsnamen durchzuführen
else:
    try:
        publisher = cells.Publisher.value
        url = base_url + '?pub={}&qtype=exact&ak=SR_API_KEY'.format(quote(publisher))
        return urllib2.urlopen(url).read()
    except:
        return None
```

#### SR AOM

* Es wird eine neue Spalte **SR AOM** angelegt, in der der Wert des SR-Feldes `prearchiving` gespeichert wird &ndash; d.h. die Angabe in SR dazu, ob eine Nutzung des Preprints ("Author Original Manuscript" = AOM) erlaubt ist.
* Mögliche Rückgabewerte (vgl. [Appendix B in der API-Dokumentation](http://sherpa.ac.uk/romeo/SHERPA%20RoMEO%20API%20V-2-9%202013-11-25.pdf)) sind:
  * "can" -> Preprint darf genutzt werden
  * "cannot" -> Preprint darf *nicht* genutzt werden
  * "restricted" -> Preprint darf *unter bestimmten Bedingungen* genutzt werden
  * "unclear" -> Angaben in Policy unklar
  * "unknown" -> es liegen keine Angaben vor

**SR** -> Edit column -> Add column based on this column -> **SR AOM**

```
forEach(value.parseHtml().select('prearchiving'),v,
  v.htmlText()
).uniques().join(' --- ')
```

#### SR AOM restriction

* Es wird eine neue Spalte **SR AOM restriction** angelegt, in der der Wert des SR-Feldes `prerestriction` gespeichert wird &ndash; d.h. die Angabe in SR dazu, unter welchen Bedingungen/Einschränkungen der Preprint ("Author Original Manuscript" = AOM) genutzt werden kann.

**SR** -> Edit column -> Add column based on this column -> **SR AOM restriction**

```
forEach(value.unescape('html').parseHtml().select('prerestriction'),v,
  v.htmlText()
).uniques().join(' --- ')
```

* Beispiel für Ausgabe: `Must obtain written permission from Editor --- Must not violate ACS ethical Guidelines`

#### SR AAM

* Es wird eine neue Spalte **SR AAM** angelegt, in der der Wert des SR-Feldes `postarchiving` gespeichert wird &ndash; d.h. die Angabe in SR dazu, ob eine Nutzung des Postprints ("Author Accepted Manuscript" = AAM) möglich ist.
* Mögliche Rückgabewerte [s.o.](#sr-aom)

**SR** -> Edit column -> Add column based on this column -> **SR AAM**

```
forEach(value.parseHtml().select('postarchiving'),v,
  v.htmlText()
).uniques().join(' --- ')
```

#### SR AAM restriction

* Es wird eine neue Spalte **SR AAM restriction** angelegt, in der der Wert des SR-Feldes `postrestriction` gespeichert wird &ndash; d.h. die Angabe in SR dazu, unter welchen Bedingungen/Einschränkungen der Postprint ("Author Accepted Manuscript" = AAM) genutzt werden kann.

**SR** -> Edit column -> Add column based on this column -> **SR AAM restriction**

```
forEach(value.unescape('html').parseHtml().select('postrestriction'),v,
  v.htmlText()
).join(' --- ')
```
* Beispiel 1 für Ausgabe: `12 months embargo`
* Beispiel 2 für Ausgabe: `If mandated by funding agency or employer/ institution --- If mandated to deposit before 12 months, must obtain waiver from Institution/Funding agency or use AuthorChoice --- 12 months embargo`


#### SR VOR

* Es wird eine neue Spalte **SR VOR** angelegt, in der der Wert des SR-Feldes `pdfarchiving` gespeichert wird &ndash; d.h. die Angabe in SR dazu, ob eine Nutzung der Verlagsversion ("Version of Record" = VOR) möglich ist.
* Mögliche Rückgabewerte [s.o.](#sr-aom)

**SR** -> Edit column -> Add column based on this column -> **SR VOR**

```
forEach(value.parseHtml().select('pdfarchiving'),v,
  v.htmlText()
).uniques().join(' --- ')
```

#### SR VOR restriction

* Es wird eine neue Spalte **SR VOR restriction** angelegt, in der der Wert des SR-Feldes `pdfrestriction` gespeichert wird &ndash; d.h. die Angabe in SR dazu, unter welchen Bedingungen/Einschränkungen die Verlagsversion ("Version of Record" = VOR) genutzt werden kann.

**SR** -> Edit column -> Add column based on this column -> **SR VOR restriction**

```
forEach(value.unescape('html').parseHtml().select('pdfrestriction'),v,
  v.htmlText()
).uniques().join(' --- ')
```

#### SR Links Policy

**SR** -> Edit column -> Add column based on this column -> **SR Links Policy**

```
forEach(value.parseHtml().select('copyrightlink'),v,
  v.htmlText()
).uniques().join(' --- ')
```

* Beispiel 1 für Ausgabe: `Policy http://www.rsc.org/journals-books-databases/open-access/green-open-access/`
* Beispiel 2 für Ausgabe: `ACS Journal Publishing Agreement http://pubs.acs.org/page/4authors/jpa/index.html --- Copyright form http://pubs.acs.org/page/copyright/index.html --- User Guide http://pubs.acs.org/userimages/ContentEditor/1285231362937/jpa_user_guide.pdf --- NIH Policy http://pubs.acs.org/page/policy/nih/index.html`

## Spalten neu anlegen

* Es sollen in Vorbereitung für die [Rechteprüfung](/README.md/#rechteprüfung-rp) neue, leere Spalten angelegt werden, die exakt wie folgt zu benennen sind:
  * **RP Ergebnis**
  * **RP Version**
  * **RP Embargo**
  * **Datum Ablauf Embargo**
  * **RP Phrase**
* Dabei ist jeweils folgende Aktion durchzuführen:

**DOI** -> Edit column -> Add column based on this column -> **$Spaltenname**


## JSON

**Achtung** API-Key für SHERPA/RoMEO eintragen: "SR_API_KEY" suchen und 4x ersetzen.

```
[
  {
    "op": "core/column-addition-by-fetching-urls",
    "description": "Create column UPW at index 7 by fetching URLs based on column DOI using expression grel:if(isNotNull(value),\n'https://api.unpaywall.org/v2/' + value + '?email=openaccess@ub.tu-berlin.de',\nnull)",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "UPW",
    "columnInsertIndex": 7,
    "baseColumnName": "DOI",
    "urlExpression": "grel:if(isNotNull(value),\n'https://api.unpaywall.org/v2/' + value + '?email=openaccess@ub.tu-berlin.de',\nnull)",
    "onError": "store-error",
    "delay": 50,
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
    "description": "Create column UPW: IS OA? at index 8 based on column UPW using expression grel:value.parseJson().is_oa",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "UPW: IS OA?",
    "columnInsertIndex": 8,
    "baseColumnName": "UPW",
    "expression": "grel:value.parseJson().is_oa",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column UPW: J DOAJ? at index 8 based on column UPW using expression grel:value.parseJson().journal_is_in_doaj",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "UPW: J DOAJ?",
    "columnInsertIndex": 8,
    "baseColumnName": "UPW",
    "expression": "grel:value.parseJson().journal_is_in_doaj",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column UPW: bestOA at index 8 based on column UPW using expression grel:'Evidence: ' + value.parseJson()['best_oa_location'].evidence + ' -- URL: ' + value.parseJson()['best_oa_location'].url + ' -- Host Type: ' + value.parseJson()['best_oa_location']['host_type'] + ' -- License: ' + value.parseJson()['best_oa_location']['license']",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "UPW: bestOA",
    "columnInsertIndex": 8,
    "baseColumnName": "UPW",
    "expression": "grel:'Evidence: ' + value.parseJson()['best_oa_location'].evidence + ' -- URL: ' + value.parseJson()['best_oa_location'].url + ' -- Host Type: ' + value.parseJson()['best_oa_location']['host_type'] + ' -- License: ' + value.parseJson()['best_oa_location']['license']",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column UPW: OA-Repos at index 8 based on column UPW using expression grel:forEach(value.parseJson()['oa_locations'],\n  v,\n  if(v['host_type'] == 'repository',\n    'Landing page: ' + v['url_for_landing_page'] + ' -- PDF-Link: ' + v['url_for_pdf'] + ' -- License: ' + v['license'] + ' -- version: ' + v['version'],\n    '')\n).join(' ///// ')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "UPW: OA-Repos",
    "columnInsertIndex": 8,
    "baseColumnName": "UPW",
    "expression": "grel:forEach(value.parseJson()['oa_locations'],\n  v,\n  if(v['host_type'] == 'repository',\n    'Landing page: ' + v['url_for_landing_page'] + ' -- PDF-Link: ' + v['url_for_pdf'] + ' -- License: ' + v['license'] + ' -- version: ' + v['version'],\n    '')\n).join(' ///// ')",
    "onError": "set-to-blank"
  },

  {
    "op": "core/column-addition",
    "description": "Create column SR at index 31 based on column ISSN using expression jython:import urllib2\nfrom urllib import quote\n\nbase_url = 'http://www.sherpa.ac.uk/romeo/api29.php'\n\ntry:\n    issn = cells.CR_ISSN_ALL.value[0:9]\nexcept:\n    issn = None\n\nif not issn:\n    try:\n        issn = cells.ISSN.value\n    except:\n        issn = None\n\nif not issn:\n    try:\n        issn = cells.eISSN.value\n    except:\n        issn = None\n\nif issn:\n    url = base_url + '?issn={}&ak=SR_API_KEY'.format(issn)\n    return urllib2.urlopen(url).read()\n\n\nelse:\n    try:\n        publisher = cells.Publisher.value\n        url = base_url + '?pub={}&qtype=exact&ak=SR_API_KEY'.format(quote(publisher))\n        return urllib2.urlopen(url).read()\n    except:\n        return None",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "SR",
    "columnInsertIndex": 9,
    "baseColumnName": "ISSN",
    "expression": "jython:import urllib2\nfrom urllib import quote\n\nbase_url = 'http://www.sherpa.ac.uk/romeo/api29.php'\n\ntry:\n    issn = cells.CR_ISSN_ALL.value[0:9]\nexcept:\n    issn = None\n\nif not issn:\n    try:\n        issn = cells.ISSN.value\n    except:\n        issn = None\n\nif not issn:\n    try:\n        issn = cells.eISSN.value\n    except:\n        issn = None\n\nif issn:\n    url = base_url + '?issn={}&ak=SR_API_KEY'.format(issn)\n    return urllib2.urlopen(url).read()\n\n\nelse:\n    try:\n        publisher = cells.Publisher.value\n        url = base_url + '?pub={}&qtype=exact&ak=SR_API_KEY'.format(quote(publisher))\n        return urllib2.urlopen(url).read()\n    except:\n        return None",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column SR AOM at index 32 based on column SR using expression grel:forEach(value.parseHtml().select('prearchiving'),v,\n  v.htmlText()\n).uniques().join(' --- ')",
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
                "v": "can --- can",
                "l": "can --- can"
              }
            }
          ],
          "selectError": false,
          "invert": false,
          "name": "SR_AOM_ohneUniques",
          "omitBlank": false,
          "type": "list",
          "columnName": "SR_AOM_ohneUniques"
        }
      ]
    },
    "newColumnName": "SR AOM",
    "columnInsertIndex": 10,
    "baseColumnName": "SR",
    "expression": "grel:forEach(value.parseHtml().select('prearchiving'),v,\n  v.htmlText()\n).uniques().join(' --- ')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column SR AOM restriction at index 32 based on column SR using expression grel:forEach(value.unescape('html').parseHtml().select('prerestriction'),v,\n  v.htmlText()\n).uniques().join(' --- ')",
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
                "v": "can --- can",
                "l": "can --- can"
              }
            }
          ],
          "selectError": false,
          "invert": false,
          "name": "SR_AOM_ohneUniques",
          "omitBlank": false,
          "type": "list",
          "columnName": "SR_AOM_ohneUniques"
        }
      ]
    },
    "newColumnName": "SR AOM restriction",
    "columnInsertIndex": 10,
    "baseColumnName": "SR",
    "expression": "grel:forEach(value.unescape('html').parseHtml().select('prerestriction'),v,\n  v.htmlText()\n).uniques().join(' --- ')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column SR AAM at index 32 based on column SR using expression grel:forEach(value.parseHtml().select('postarchiving'),v,\n  v.htmlText()\n).uniques().join(' --- ')",
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
                "v": "can --- can",
                "l": "can --- can"
              }
            }
          ],
          "selectError": false,
          "invert": false,
          "name": "SR_AOM_ohneUniques",
          "omitBlank": false,
          "type": "list",
          "columnName": "SR_AOM_ohneUniques"
        }
      ]
    },
    "newColumnName": "SR AAM",
    "columnInsertIndex": 10,
    "baseColumnName": "SR",
    "expression": "grel:forEach(value.parseHtml().select('postarchiving'),v,\n  v.htmlText()\n).uniques().join(' --- ')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column SR AAM restriction at index 32 based on column SR using expression grel:forEach(value.unescape('html').parseHtml().select('postrestriction'),v,\n  v.htmlText()\n).join(' --- ')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "SR AAM restriction",
    "columnInsertIndex": 10,
    "baseColumnName": "SR",
    "expression": "grel:forEach(value.unescape('html').parseHtml().select('postrestriction'),v,\n  v.htmlText()\n).join(' --- ')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column SR VOR at index 32 based on column SR using expression grel:forEach(value.parseHtml().select('pdfarchiving'),v,\n  v.htmlText()\n).uniques().join(' --- ')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "SR VOR",
    "columnInsertIndex": 10,
    "baseColumnName": "SR",
    "expression": "grel:forEach(value.parseHtml().select('pdfarchiving'),v,\n  v.htmlText()\n).uniques().join(' --- ')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column SR VOR restriction at index 32 based on column SR using expression grel:forEach(value.unescape('html').parseHtml().select('pdfrestriction'),v,\n  v.htmlText()\n).uniques().join(' --- ')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "SR VOR restriction",
    "columnInsertIndex": 10,
    "baseColumnName": "SR",
    "expression": "grel:forEach(value.unescape('html').parseHtml().select('pdfrestriction'),v,\n  v.htmlText()\n).uniques().join(' --- ')",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column SR Links Policy at index 32 based on column SR using expression grel:forEach(value.parseHtml().select('copyrightlink'),v,\n  v.htmlText()\n).uniques().join(' --- ')",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "SR Links Policy",
    "columnInsertIndex": 10,
    "baseColumnName": "SR",
    "expression": "grel:forEach(value.parseHtml().select('copyrightlink'),v,\n  v.htmlText()\n).uniques().join(' --- ')",
    "onError": "set-to-blank"
  },

  {
    "op": "core/column-addition",
    "description": "Create column RP Phrase at index 35 based on column SR using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "RP Phrase",
    "columnInsertIndex": 5,
    "baseColumnName": "SR",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column Datum Ablauf Embargo at index 35 based on column SR using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "Datum Ablauf Embargo",
    "columnInsertIndex": 5,
    "baseColumnName": "SR",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column RP Embargo at index 35 based on column SR using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "RP Embargo",
    "columnInsertIndex": 5,
    "baseColumnName": "SR",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column RP Version at index 35 based on column SR using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "RP Version",
    "columnInsertIndex": 5,
    "baseColumnName": "SR",
    "expression": "grel:null",
    "onError": "set-to-blank"
  },
  {
    "op": "core/column-addition",
    "description": "Create column RP Ergebnis at index 35 based on column SR using expression grel:null",
    "engineConfig": {
      "mode": "row-based",
      "facets": []
    },
    "newColumnName": "RP Ergebnis",
    "columnInsertIndex": 5,
    "baseColumnName": "SR",
    "expression": "grel:null",
    "onError": "set-to-blank"
  }
]
```
