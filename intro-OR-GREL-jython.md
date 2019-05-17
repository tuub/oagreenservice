## Einführung OpenRefine, GREL, Jython

### OpenRefine

* "powerful tool for working with messy data": Bereinigung, (Format-)Transformation & Aggregation von Daten
* Aktuelle Version 3.1 ([Download](http://openrefine.org/download.html) für Windows, Mac OS, Linux)
* primär GUI-gestützt; diverse [Client libraries](https://github.com/OpenRefine/OpenRefine/wiki/Documentation-For-Developers#known-client-libraries-for-refine) zur Automatisierung (z.B. Python, R, ruby)
* Desktop-Anwendung (Java-basiert): Bearbeitung im Browser (Aufruf via <http://127.0.0.1:3333/> oder <http://localhost:3333/>); Daten & Bearbeitungshistorie werden lokal gespeichert
* Was kann OpenRefine?
  * Datenexploration: Filter & Facetten (verschiedene Datentypen), Clustering, Dubletten
  * Transformation: auf Ebenen Zellen, Zeilen oder Spalten; Standardfunktionen (Zellen aufteilen oder verketten, Leerzeichen entfernen, Datentyp ändern, Groß-/Kleinschreibung ändern, ...); Mini-Skripte mit Google Refine Expression Language (GREL), Python/Jython oder Clojure
  * History: Undo auch für ausgewählte (frühere) Schritte; kann exportiert werden, um gleiche Operationen auf andere Datensets  anzuwenden
  * Aggregation: mit Daten aus anderen OR-Projekten; Webschnittstellen abfragen + Daten parsen (JSON, XML, HTML); Reconciliation Services (z.B. Wikidata, GND via LOBID, VIAF) bzw. [Extensions](http://openrefine.org/download.html) (z.B. GoKB)
  * Bearbeitung der Daten in tabellarischer Form ([Achtung mit row/record-Modus](https://librarycarpentry.org/lc-open-refine/03-working-with-data/index.html)) -> diverse Import- und Exportoptionen
  * Import: TSV, CSV, \*SV, XLS(X), JSON, XML, RDF as XML, Google Data documents; andere Formate via OpenRefine Extensions
  * Export: Standardformate (CSV, TSV, XLS(X), HTML table, ...); custom templating z.B. für XML
* Was kann OpenRefine nicht?
  * Manuelles Editieren von Zellen möglich aber mühselig
  * Manuelles Hinzufügen neuer Zeilen
  * Statistische Auswertung oder Plotten (Diagramme, Grafiken) -- aber bestens geeignet für Vorbereitung der Daten für weitere Verwendung in Excel (Pivot) oder R, Python o.Ä.
  * Rechteverwaltung für verschiedene Nutzer\*innen, die an gleichem Projekt arbeiten
  * Mitunter Performance Probleme bei sehr großen Datensets (> 10.000 records)
* Mehr lernen:
  * Online Tutorials: u.a. [OR-Webseite](http://openrefine.org/documentation.html), [Recipies](https://github.com/OpenRefine/OpenRefine/wiki/Recipes) bzw. [Liste externer Tutorials](https://github.com/OpenRefine/OpenRefine/wiki/External-Resources) im [OpenRefine GitHub Wiki](https://github.com/OpenRefine/OpenRefine/wiki)
  * Tutorials Bibliotheken/Kultureinrichtungen: [Library Carpentry OpenRefine](https://librarycarpentry.org/lc-open-refine/), Artikelreihe bei [HistHub](https://histhub.ch/cat/net/blog/openrefine/)
  * Einführung: Ruben Verborgh, Max De Wilde (2013) [Using OpenRefine](http://www.packtpub.com/openrefine-guide-for-data-analysis-and-linking-dataset-to-the-web/book), ISBN 9781783289080
  * Zahlreiche allgemeine Einführungen und Lösungsansätze für bestimmte Aufgaben auf [YouTube](https://www.youtube.com/results?search_query=openefine)


### GREL: General Refine Expression Language

* [Einführung in GREL auf GitHub-Seite von OpenRefine](https://github.com/OpenRefine/OpenRefine/wiki/General-Refine-Expression-Language), darin u.a. [Understanding expressions: Introduction](https://github.com/OpenRefine/OpenRefine/wiki/Understanding-Expressions).
* Um die in den Scripten häufig genutzten GREL-Ausdrücke nachzuvollziehen, sind ggf. folgende Abschnitte hilfreich:
  * [GREL Controls](https://github.com/OpenRefine/OpenRefine/wiki/GREL-Controls) -> `if()`, `forEach()`, `isBlank()`
  * [GREL String Functions](https://github.com/OpenRefine/OpenRefine/wiki/GREL-String-Functions) -> `strip()`
  * [GREL Array Functions](https://github.com/OpenRefine/OpenRefine/wiki/GREL-Array-Functions) -> `join()`, `uniques()`
  * [JSON-Parsing in GREL](https://github.com/OpenRefine/OpenRefine/wiki/GREL-Other-Functions#parsejsonstring-s) -> `parseJson()` ([Beispiel für JSON Parsing](https://github.com/OpenRefine/OpenRefine/wiki/Fetching-URLs-From-Web-Services))
  * [XML-Parsing in GREL](https://github.com/OpenRefine/OpenRefine/wiki/GREL-Other-Functions#jsoup-xml-and-html-parsing-functions) -> `parseHtml()`
* In GREL wird &ndash; so wie auch in Python und anderen Skriptsprachen &ndash; beginnend bei "0" gezählt. Um den ersten Wert einer Zeichenkette oder eines Arrays zu adressieren, ist als Index 0 anzugeben, für den zweiten Wert der Index 1 usw. (siehe auch [GREL Beispiele](https://github.com/OpenRefine/OpenRefine/wiki/Recipes#2-string-manipulation)).
* In GREL-Code können Zeilenumbrüche genutzt werden, müssen aber nicht: 

```
if(value == 'closed access', value, 'smash paywall!')
```

ist identisch mit

```
if(value == 'closed access',
  value,
  'smash paywall!'
)
```

#### Vor- und Nachteile

Pro

* Einfache Syntax -> schnell zu lernen
* Viele Funktionen

Contra

* Keine Variablen
* Begrenzte Möglichkeiten zum Skripten
* Begrenzte Lesbarkeit bei komplexeren Operationen


### Jython

* Für Operationen, für die GREL nicht ausreichend ist, gibt es in OpenRefine die Möglichkeit Jython zu nutzen &ndash; eine Java-Implementierung von Python 2.7, die praktisch alles kann, was Python 2.7 kann. Damit ist es u.a. möglich, Variabeln zu nutzen, Funktionen zu schreiben oder Ausnahmen abzufangen.
* Mehr zu Jython
  * [Jython in OpenRefine](https://github.com/OpenRefine/OpenRefine/wiki/Jython)
  * [Jython Webseite](https://www.jython.org/)
  * [Jython - Quick Guide](https://www.tutorialspoint.com/jython/jython_quick_guide.htm)
  * [W3C School Python](https://www.w3schools.com/python/)


#### REST API Call

```python
# DOI -> Edit column -> Add column based on this column -> CR

import urllib2

if cells.doiRA.value == 'Crossref':
    try:
        doi = cells.DOI.value.strip()
        url = 'https://api.crossref.org/works/{}?mailto=INSERT@YOUR.EMAIL'.format(doi)
        response = urllib2.urlopen(url)
        return response.read().decode('unicode_escape')
    except:
        return None
else:
    return None
```

  * `import` -> Mittels `import` lädt Python/Jython Packete (packages) und Module (modules) nach (packes sind modules, die modules enthalten). `urllib2` ist ein Packet, mittels dessen Module Verbindungen zum Internet hergestellt werden können. ([Dokumentation](https://docs.python.org/2/library/urllib2.html).)
  * `if` -> Syntax durch Einzug -> Ausführung von Befehlen nach Test einer Bedingung: Wenn die Bedingung (`if`) erfüllt wird, wird das darunter Eingerückte ausgeführt; wenn die Bedingung nicht erfüllt wird, wird die Anweisung unter `else` ausgeführt. Mit `elif` (lies: else if) könnten weitere Bedingung integriert werden.
  * `try ... except ...` -> Ausführen von Befehlen, auch wenn Fehler auftreten: Wenn in dem unter dem `try:` Eingerückten ein Fehler passiert, bricht das Script nicht ab (wie bei anderen Fehlern), sondern führt `except` aus (hier könnte man dann z.B. eine Fehlermeldung zurückgeben). Könnte um weitere Bedingungen (`else` oder `finally`) ergänzt werden, siehe [W3C School](https://www.w3schools.com/python/python_try_except.asp).
  * `doi = cells.DOI.value.strip()`-> der Variable `doi` wird der um Whitespace bereinigte Wert der Spalte **DOI** zugewiesen von (Achtung: DOI-Wert wird lediglich für die Anfrage transformiert)
  * `url = 'https://api.crossref.org/works/{}?mailto=INSERT@YOUR.EMAIL'.format(doi)` -> String-Formatation, siehe [pyformat.info/](https://pyformat.info/).
  * `response = urllib2.urlopen(url)` -> Die eben erstellte URL wird mittels des [Moduls `urlopen()`](https://docs.python.org/2/library/urllib2.html#urllib2.urlopen) geöffnet, ein *response object*, welches u.a. den Seitenquelltext enthält, wird zurückgegeben und in der Variable `response` gespeichert.
  * `return response.read().decode('unicode_escape')` -> `return` siehe unten. `response.read()` liest die Antwort, die in dem *response object* gespeichert wurde, aus. Hier kann das Decodieren wichtig sein (`.decode('unicode_escape')`) und muss je nach REST API, HTML-Quelltext etc. angepasst werden.
  * `return` -> Rückgabewert definieren: Was OpenRefine in die Zelle schreiben soll, muss mit `return` übergeben werden. Wird im Programmablauf das erste `return` erfolgreich erreicht, werden eventuell darunter folgende Codezeilen nicht mehr ausgeführt.


#### List Comprehension 

Eine schnelle Methode, um Listen (arrays) zu erstellen, sind sog. [List Comprehensions](https://docs.python.org/2/tutorial/datastructures.html#list-comprehensions).

Um aus einer Ausgangs-Liste eine neue Liste zu erstellen, wobei die neue Liste alle Elemente aus der Ausgangs-Liste enthalten soll, die eine bestimmte Bedingung erfüllen (gerade Zahlen, Großbuchstaben, ...), ist folgender Ansatz möglich:

```python
new_list = []
for item in input_list:
    if foo:
        new_list.append(item)
outout_string = '||'.join(new_list)
return output_string
```
  * `new_list = []` -> Eine neue, leere Liste mit dem Namen `new_list` erstellen
  * `for item in input_list:` -> Über alle Elemente der `input_list` iterieren (vgl. `forEach()`)
  * `if foo:` -> Wenn die Bedingung `foo` erfüllt ist, den unter ihr eingerückten Schritt ausführen
  * `new_list.append(item)` -> Der neuen Liste ein neues Element hinzufügen, nämlich das aktuelle Element der Liste
  * `outout_string = '||'.join(new_list)'` -> Da in Openrefine keine Arrays in Zellen abgespeichert werden können, soll die neue Liste als String ausgegeben werden: dafür nutzen wir die Methode `join()`, die alle Elemente aus `new_list` mit dem selbst-definierten Trennzeichen (im Beispiel `||`) zwischen ihnen verbindet.
  * `return output_string'` -> Die gebildete Zeichenkette wird (als string) zurückgegeben, Openrefine speichert ihn in den Zellen.
  
Dasselbe Resultat wird erreicht mit: `return '||'.join([i for i in input_list if foo])`.


#### Data Wrangling

Hier ein Beispiel, wie die Antwort von der BASE-REST-API (diese liegt in der Spalte **BASE** vor), die im JSON-Format gegeben wird, verarbeitet werden kann.

```python
# dc.subject.ddc[de] -> Edit cells -> Transform

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
  * `import json` -> Importiert das Packet `json`, mit dem JSON-Daten verarbeitet werden können. Basis ist, dass diese in ein [Dictonary](https://docs.python.org/2/tutorial/datastructures.html#dictionaries) überführt werden.
  * `j = json.loads(cells.BASE.value)` -> Von dem Packet `json` wird das Modul `.loads()` aufgerufen, das einen String als Dictornary lädt; der String ist der Wert aus der Spalte **BASE** (`cells.BASE.value` ist GREL-Syntax und besagt 'Nimm für jede Zeile den Wert aus der Spalte **BASE**.'). Dieses Dictonary wird der Variable `j` zugewiesen.
  * `response = j.get('response', {})` -> Als Dictonary verfügt die Variable `j` über die Methode `.get()`: Der Methode wird ein Schlüssel übergeben (in dem Beispiel den String *response*), sie gibt aus dem Dictonary den Wert für den Schlüssel zurück. Ist der Schlüssel nicht vorhanden, wird der Default-Wert `None` zurückgeliefert. Dieser kann geändert werden; in dem Beispiel in ein leeres Dictonary (`{}`), da `None` im kommenden Schritt einen Fehler verursachen würde.
  * `docs = response.get('docs', {})` -> Siehe Erläuterung `response = j.get('response', {})`.
  * `return '||'.join(set(...))`
    - `return` -> Siehe Erläuterung `return` [oben](#rest-api-call-using-jython).
    - `'||'.join(...)` -> Jeder String verfügt über die join-Methode: In der Klammer muss ein iterierbares Object übergeben werden, dessen Elemente mit dem angegebenen Trennzeichen (hier `||`) zu einem String verbunden werden.
    - `set(...)` ist ein Python-Datentyp (siehe [Set](https://docs.python.org/2/tutorial/datastructures.html#sets)) &ndash; ein Array, in dem jedes Element nur einmal vorkommen kann.
  * `['BASE-Classifier: {}\nBASE-Repo-Daten: {}'.format('||'.join(i.get('dcautoclasscode', ['--'])), '||'.join(i.get('dcclasscode', ['--']))) for i in docs]` -> Dieser längere Codeblock macht zwei Dinge: 
    - `'BASE-CLASSIFIER: {}'.format(i.get('dcautoclasscode', ['--']))` -> [formatiert einen String](https://pyformat.info/), indem der Wert einer Variable hinzugefügt wird (hier wird der Value von einem Dictonary eingefügt).
    - `[i for i in docs]` -> [List Comprehension](https://docs.python.org/2/tutorial/datastructures.html#list-comprehensions), eine schnelle Methode, um aus einem iterierbaren Object (hier einem Dictonary) eine neue Liste zu erstellen.
