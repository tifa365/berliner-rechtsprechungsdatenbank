# Gesetze Berlin API

Unofficial API documentation for the Berlin legal database (Berliner Vorschriften- und Rechtsprechungsdatenbank).

**Base URL:** `https://gesetze.berlin.de/jportal/wsrest/recherche3`

## Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/search` | POST | Search for legal documents |
| `/suggest` | POST | Get autocomplete suggestions |
| `/document` | POST | Get full document content |
| `/toc` | POST | Get table of contents (for laws) |

## Search Field Coverage

| UI Field | API `id` | Suggest | Search | Notes |
|----------|----------|---------|--------|-------|
| Text | `Text` | ✅ | ✅ | Full-text search |
| Titel | `Titel` | ✅ | ✅ | Law/decision title |
| Norm | `Norm` | ✅ | ✅ | Legal norm reference |
| Fundstelle | `Fundstelle` | ✅ | ❌ 500 | Publikationsnachweis (z.B. "GVBl. 2025, 163") |
| Autor/Gericht | `AutorGericht` | ✅ | ✅ | Author/Court name |
| AZ/ECLI | `AZ` | ✅ | ✅ | Case number/ECLI |
| Datum | `Datum` | - | ✅ | Date range: `DD.MM.YYYY bis DD.MM.YYYY` |
| Category | `filters.CATEGORY` | - | ✅ | Alles, Gesetze, Rechtsprechung |
| Pagination | `searchTasks.RESULT_LIST` | - | ✅ | start, size |
| Sorting | `searchTasks.RESULT_LIST.sort` | - | ✅ | juris, date, relevance |

**Legend:** ✅ Documented | ❌ Error | ? Not yet tested | - Not applicable

## Search Request

```json
{
  "searchTasks": {
    "CATEGORY_HITS": {},
    "RESULT_LIST": {
      "start": 1,
      "size": 26,
      "sort": "juris",
      "addToHistory": true,
      "addCategory": true
    },
    "RESULT_LIST_CACHE": { "start": 25, "size": 27 },
    "SEARCH_WORD_HITS": {}
  },
  "filters": {
    "CATEGORY": ["Alles"]
  },
  "searches": [
    {"id": "Text", "value": "Mietrecht"},
    {"id": "Titel", "value": "\"SchulG BE\""},
    {"id": "FastSearch", "value": "*"}
  ],
  "clientID": "bsbe",
  "clientVersion": "bsbe - V08_26_01 - 17.12.2025 16:06",
  "r3ID": "2025-12-28T10:44:51.022Z"
}
```

## Suggest Request

```json
{
  "field": "Titel",
  "input": "Schul",
  "withHits": true,
  "category": "Alles",
  "startTime": 1766919299629,
  "clientID": "bsbe",
  "clientVersion": "bsbe - V08_26_01 - 17.12.2025 16:06",
  "r3ID": "2025-12-28T10:44:51.022Z"
}
```

## Response

| Field | Description |
|-------|-------------|
| `resultList` | Array of search results |
| `hits` | Total number of matching documents |
| `categoryHits` | Hit counts per category |
| `searchWords` | Parsed search terms with individual hit counts |

### Categories

| Category | Description | Count |
|----------|-------------|-------|
| `Alles` | All documents | 78,475 |
| `Gesetze` | Laws | 49,982 |
| `Rechtsprechung` | Court decisions | 28,493 |

## Document Request

```json
{
  "docId": "NJRE001628332",
  "format": "xsl",
  "docPart": "L",
  "keyword": null,
  "sourceParams": {
    "position": 0,
    "sort": "juris",
    "source": "TL",
    "category": "Alles"
  },
  "searches": [],
  "clientID": "bsbe",
  "clientVersion": "bsbe - V08_26_01 - 17.12.2025 16:06",
  "r3ID": "2025-12-28T10:44:51.022Z"
}
```

### Document Response

| Field | Description |
|-------|-------------|
| `text` | Full HTML content of document |
| `head` | Header HTML with metadata |
| `otherRepresentations.pdfUrl` | PDF download path |
| `permalink` | Permanent URL |
| `tabs` | Available parts (K=Kurztext, L=Langtext) |
| `documentTitle` | Formatted title |

### docPart Values

| Value | Description |
|-------|-------------|
| `L` | Langtext (full text) |
| `K` | Kurztext (summary) |

## Query Syntax

### Grundregeln

| Regel | Beschreibung |
|-------|--------------|
| Case-insensitive | `VwGO` = `vwgo` = `VWGO` |
| AND (Standard) | Leerzeichen = Schnittmenge: `vg berlin` → beide Begriffe müssen vorkommen |
| OR (explizit) | `miete ODER pacht` → einer der Begriffe reicht |
| Exact Match | `"§ 4 BauO Bln"` → exakte Phrase |

### Mischfeld-Suche

Die Schnellsuche akzeptiert Textbegriffe **und** bibliografische Angaben gemischt:

```
9 u 19/20              # Aktenzeichen
vg berlin              # Gericht
29.09.2005             # Datum
§ 4 bauo bln           # Norm
gvbl                   # Fundstelle (GVBl.)
```

### Linguistische Erweiterungen

Das System findet automatisch:
- **Flexionsformen**: `Baum` → findet auch `Bäume`
- **Wortstämme**: `Baum` → findet auch `Baumkrone`, `Apfelbaum`
- **Synonyme**: `Handy` → findet auch `Mobiltelefon`

**Empfehlung:** Begriffe in Grundform eingeben.

### Suchstrategien

| Ziel | Empfehlung |
|------|------------|
| Gerichtsentscheidung | Aktenzeichen allein **oder** Gericht + Datum |
| Norm | Abkürzung mit/ohne § (z.B. `§ 4 bauo bln`) |
| Fundstelle | Token wie `gvbl` über Text-Feld |
| Große Treffermenge | Kategorie einschränken (Rechtsprechung/Gesetze) |
| Relevante Treffer oben | `sort=relevance` statt `date` |

### Query Prefixes (FastSearch)

Interne Prefixes für kombinierte Suchen:

| Prefix | Feld | Beispiel |
|--------|------|----------|
| `TXT:` | Text | `TXT:Kita` |
| `NORM:` | Norm | `NORM:"BImSchV 16"` |
| `FUND:` | Fundstelle | `FUND:Berlin` |
| `AUTOR:` | AutorGericht | `AUTOR:Mueller` |
| `TITEL:` | Titel | `TITEL:"SchulG BE"` |
| `AZ:` | AZ/ECLI | `AZ:ECLI:DE:BGH:2017:...` |
| `DATUM:` | Datum | `DATUM:"01.12.2025 bis 12.12.2025"` |

### Hinweise

- **Whitespace:** Geschützte Leerzeichen (NBSP) können Probleme verursachen → normalisieren
- **Sonderzeichen:** `§`, `/`, `-` werden korrekt verarbeitet

## Field Notes

### Fundstelle (Citation/Source)

"Fundstelle" ist die Zitier-/Nachweisstelle - **wo** ein Dokument veröffentlicht bzw. nachgewiesen ist.

Typische Werte:
- `GVBl.` - Gesetz- und Verordnungsblatt Berlin
- `GVBl. 2025, 163` - Mit Jahr und Seite

**Bug:** Search auf `Fundstelle` liefert 500-Fehler (Suggest funktioniert).
**Workaround:** Fundstellen-Token über `Text`-Feld suchen.

### Getestete Suchbegriffe (Text-Feld)

| Begriff | Hits | Begriff | Hits |
|---------|------|---------|------|
| berlin | 78,475 | vergabe | 2,923 |
| gesetz | 54,407 | schulgesetz | 2,610 |
| gvbl | 53,039 | vwvfg | 2,492 |
| verordnung | 36,173 | datenschutz | 2,163 |
| vwgo | 12,676 | schulg | 1,779 |
| gewerbe | 5,020 | asog | 1,668 |
| umwelt | 4,980 | bauordnung | 1,388 |
| polizei | 4,830 | steuergesetz | 1,382 |
| verwaltungsverfahren | 4,168 | bauo | 771 |
| kündigung | 3,307 | mietrecht | 693 |

## Authentication

**Erforderlich für alle Requests:**

- Session cookie: `JSESSIONID` (aus Browser-Session)
- CSRF token header: `x-csrf-token`
- Portal ID header: `juris-portalid: bsbe`

Ohne gültige Session: `{"msgId": "security_notAuthenticated", "httpStatus": 500}`

**Session erhalten:**
1. https://gesetze.berlin.de/bsbe/search im Browser öffnen
2. DevTools → Network → beliebige API-Request kopieren
3. Cookies und x-csrf-token extrahieren

## Files

- `openapi.yaml` - OpenAPI 3.1 specification
