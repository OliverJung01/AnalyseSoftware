# AnalyseSoftware

Analyse-Software zur Untersuchung von auffaelligen Handelsmustern auf Prediction Markets wie Polymarket und Kalshi.

Ziel ist nicht, Insiderhandel direkt zu beweisen. Die Software soll Daten sammeln, strukturieren und Auffaelligkeiten sichtbar machen, die spaeter genauer analysiert werden koennen.

## Zielbild

Die Anwendung soll eine Datenpipeline bereitstellen, die Marktdaten, Handelsdaten und externe Ereignisse sammelt, vereinheitlicht und fuer Analysen vorbereitet.

Im Anschluss sollen darauf Analysefunktionen entstehen, zum Beispiel:

- Erkennung ungewoehnlicher Preis- und Volumenbewegungen
- Identifikation von Wallets oder Accounts mit auffaellig gutem Timing
- Vergleich von Trades mit oeffentlichen Ereignissen und News-Zeitpunkten
- Analyse wiederkehrender Muster ueber mehrere Maerkte hinweg
- Aufbau eines erklaerbaren Suspicion Scores fuer auffaellige Aktivitaet

## Erste Projektphase: Datenpipeline

Der erste konkrete Schritt ist der Aufbau einer Pipeline fuer Polymarket und Kalshi.

Die Pipeline soll:

1. Marktdaten laden
2. Handelsdaten laden
3. Orderbuch- oder Preisverlaufsdaten laden, soweit verfuegbar
4. Daten normalisieren
5. Rohdaten und bereinigte Daten getrennt speichern
6. Datenqualitaet pruefen
7. Eine stabile Grundlage fuer spaetere Analysen bereitstellen

## Datenquellen

### Polymarket

Relevante Daten:

- Maerkte und Outcomes
- Marktstatus, Startzeit, Endzeit und Ergebnis
- Preise und historische Preisbewegungen
- Trades mit Zeitstempel, Preis, Groesse und Seite
- Wallet- oder Nutzerinformationen, soweit oeffentlich verfuegbar
- On-chain-Daten wie Transfers, Funding-Quellen und Wallet-Beziehungen

### Kalshi

Relevante Daten:

- Events und Maerkte
- Kontrakte und Outcomes
- Marktstatus, Laufzeit und Ergebnis
- Trades, Volumen und Preise
- Orderbuchdaten, soweit verfuegbar
- Zeitreihen fuer Preis- und Liquiditaetsentwicklung

Hinweis: Falls mit "Kaushi" eine andere Plattform gemeint ist, sollte diese Stelle angepasst werden.

## Vorgeschlagene Architektur

```text
data sources
    |
    v
ingestion layer
    |
    v
raw storage
    |
    v
normalization / cleaning
    |
    v
processed storage
    |
    v
analysis layer
    |
    v
dashboard / reports / alerts
```

## Pipeline-Komponenten

### 1. Ingestion

Verantwortlich fuer das Laden der Daten aus APIs, Subgraphs, On-chain-Quellen oder exportierten Dateien.

Aufgaben:

- API-Clients fuer Polymarket und Kalshi
- Pagination und Rate Limits behandeln
- Wiederholbare Imports ermoeglichen
- Zeitfenster-basierte Aktualisierung
- Fehler und fehlende Daten protokollieren

### 2. Raw Storage

Rohdaten sollen moeglichst unveraendert gespeichert werden.

Zweck:

- Nachvollziehbarkeit
- Reproduzierbarkeit
- Spaetere Neuverarbeitung bei verbessertem Datenmodell

Moegliche Formate:

- JSONL fuer API-Antworten
- Parquet fuer groessere Datenmengen
- PostgreSQL fuer strukturierte Speicherung

### 3. Normalisierung

Die Plattformen haben unterschiedliche Begriffe und Datenstrukturen. Deshalb braucht es ein einheitliches internes Datenmodell.

Beispiele:

- `market_id`
- `platform`
- `title`
- `category`
- `status`
- `start_time`
- `end_time`
- `resolved_at`
- `outcome`
- `trade_id`
- `trader_id`
- `price`
- `size`
- `side`
- `timestamp`

### 4. Data Quality Checks

Vor der Analyse sollten automatische Checks laufen.

Beispiele:

- Fehlende Zeitstempel
- Doppelte Trades
- Negative oder unplausible Preise
- Fehlende Marktzuordnung
- Inkonsistente Zeitzonen
- Spruenge oder Luecken in Zeitreihen

### 5. Processed Storage

Bereinigte Daten sollen analysefreundlich gespeichert werden.

Empfohlener Start:

- PostgreSQL als Hauptdatenbank
- Optional spaeter TimescaleDB fuer Zeitreihen
- Optional Neo4j oder NetworkX fuer Wallet- und Account-Beziehungen

## Erstes internes Datenmodell

### Markets

```text
market_id
platform
external_id
title
description
category
status
start_time
end_time
resolved_at
outcome
volume
liquidity
created_at
updated_at
```

### Trades

```text
trade_id
platform
external_id
market_id
trader_id
side
price
size
notional
timestamp
raw_payload_id
created_at
```

### Price Snapshots

```text
snapshot_id
platform
market_id
outcome
bid
ask
mid
last_price
volume
liquidity
timestamp
raw_payload_id
```

### Traders

```text
trader_id
platform
external_id
wallet_address
first_seen_at
last_seen_at
metadata
```

### Events

```text
event_id
source
title
description
published_at
event_time
url
related_market_id
created_at
```

## Analyseideen fuer spaetere Phasen

### Timing-Anomalien

Untersucht, ob grosse oder profitable Trades kurz vor relevanten Ereignissen stattfinden.

Beispiele:

- Grosse Position kurz vor News
- Wiederholtes Kaufen kurz vor Ergebnisveroeffentlichung
- Preisbewegung vor oeffentlicher Information

### Profitabilitaets-Anomalien

Untersucht, ob einzelne Trader dauerhaft ungewoehnlich erfolgreich sind.

Beispiele:

- Hohe Trefferquote ueber viele Maerkte
- Hoher Gewinn bei geringem Risiko
- Wiederholte Gewinne in eng begrenzten Informationsfenstern

### Koordinierte Aktivitaet

Untersucht, ob mehrere Wallets oder Accounts auffaellig aehnlich handeln.

Beispiele:

- Gleiche Handelsrichtung im selben Zeitfenster
- Gemeinsame Funding-Quellen
- Wiederkehrende Marktueberschneidungen

## MVP

Der erste MVP sollte bewusst klein bleiben.

Geplanter Umfang:

1. Einen Polymarket-Importer bauen
2. Einen Kalshi-Importer bauen
3. Rohdaten lokal speichern
4. Daten in ein gemeinsames Schema transformieren
5. Eine einfache Datenbankstruktur anlegen
6. Erste Auswertungen fuer Volumen, Preisbewegung und Trader-Aktivitaet erzeugen

Erste einfache Fragen:

- Welche Maerkte hatten die groessten Volumen-Spikes?
- Welche Trader waren kurz vor starken Preisbewegungen aktiv?
- Welche Maerkte zeigen Preisbewegungen vor bekannten Ereignissen?
- Welche Accounts oder Wallets tauchen wiederholt in auffaelligen Zeitfenstern auf?

## Technologievorschlag

Start:

- Python fuer Datenpipeline und Analyse
- Pandas oder Polars fuer Datenverarbeitung
- PostgreSQL fuer strukturierte Daten
- SQLAlchemy oder DuckDB fuer lokale Experimente
- pytest fuer Tests

Spaeter:

- TimescaleDB fuer Zeitreihen
- NetworkX oder Neo4j fuer Graphanalysen
- Streamlit, Dash oder Next.js fuer ein Dashboard
- Airflow, Dagster oder Prefect fuer orchestrierte Pipelines

## Naechste Schritte

1. Projektstruktur anlegen
2. API- und Datenquellen fuer Polymarket und Kalshi konkret verifizieren
3. Datenmodell als SQL-Migration oder ORM-Modelle definieren
4. Ersten Polymarket-Importer implementieren
5. Rohdaten speichern und Normalisierung testen
6. Danach Kalshi-Importer nach gleichem Muster ergaenzen

## Rechtlicher und analytischer Hinweis

Die Software soll auffaellige Muster erkennen und Hinweise fuer weitere Pruefung liefern. Auffaelliges Trading ist kein Beweis fuer illegales Verhalten. Ergebnisse sollten deshalb immer als Indikatoren, Scores oder Verdachtsmomente formuliert werden.
