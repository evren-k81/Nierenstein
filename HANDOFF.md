# Nierenstein-Tagebuch – Übergabe an Claude Code

## Worum es geht
Eine kleine Web-App für Menschen mit wiederkehrenden Nierenstein-Koliken.
Ziel: Trinkmenge im Alltag festhalten, Kolik-Anfälle dokumentieren und vor dem
Arzttermin eine Zusammenfassung erstellen. Die App ersetzt keinen Arzt.

Zielgruppe: Erwachsene (ca. 45–55), nutzt es am Handy → große Tippflächen, einfache Sprache,
alles auf Deutsch (Österreich).

Design: schlicht und hell, immer im hellen Modus (kein Dark Mode). Warmes Off-White (`--bg`),
weiße Karten mit weichem Schatten statt Rahmen, eingefärbte Knöpfe statt Umrandungen,
eine Schrift (Figtree). Akzente: Blau `--water` fürs Trinken, Terrakotta `--pain` für die Kolik,
Grün `--ok` für Erledigtes. Weiße Schrift auf den Akzentfarben erfüllt den Kontrast für große Schrift.

## Aktueller Stand
- Datei: `nierenstein-tagebuch.html` (eine einzige Datei, HTML + CSS + JS, keine Libraries)
- Ursprünglich als claude.ai-Artifact veröffentlicht. Speichern läuft dort über die
  Artifact-Datenbank (`claude.use("db")`, privater Bereich pro Person).
- **Außerhalb von claude.ai** fällt die App automatisch auf `localStorage` zurück,
  d. h. sie funktioniert auch als normale Datei / Website, speichert dann aber nur
  in diesem einen Browser.

## Funktionen
1. **Heute**
   - Gläser Wasser zählen (+1 / −1, je 250 ml), Tagesziel wählbar (2,0–3,5 l)
   - Balkendiagramm der letzten 7 Tage (unter Ziel = hell)
   - Kasten „Sofort ärztliche Hilfe holen bei …“ (Fieber/Schüttelfrost, Schmerzen
     trotz Schmerzmittel, anhaltendes Erbrechen, kaum Urin) + Notruf 144 / 1450
2. **Anfälle**
   - Formular: Beginn, Dauer (h), Schmerz 1–10 (Slider mit Wortbeschreibung),
     Seite (links/rechts/beidseits/Unterbauch-Leiste), Begleitbeschwerden als Chips
     (Übelkeit, Erbrechen, Blut im Urin, Brennen, Harndrang, Fieber, Schüttelfrost),
     „Was hat geholfen?“, Notizen
   - Warnhinweis erscheint, sobald Fieber oder Schüttelfrost angekreuzt ist
   - Liste aller Anfälle, bearbeiten und löschen (Löschen mit Bestätigung durch zweiten Tipp)
3. **Für den Arzt**
   - Zeitraum 30 Tage / 3 Monate / 1 Jahr
   - Kennzahlen: Anzahl Anfälle, Ø Schmerz, Ø Dauer, Ø Trinkmenge (14 Tage)
   - Textbericht inkl. Häufigkeit der Begleitbeschwerden, Einzelliste und eigene Fragen
   - Button „Bericht kopieren“

## Datenmodell (alle Dokumente in einer Sammlung)
| ID | Inhalt |
|---|---|
| `a-<timestamp>` | `{kind:"attack", start:"YYYY-MM-DDTHH:MM", hours:number\|null, pain:1-10, side:string, symptoms:string[], helped:string, note:string}` |
| `w-YYYY-MM-DD` | `{kind:"water", date:"YYYY-MM-DD", glasses:number}` |
| `settings` | `{kind:"settings", goalMl:number}` |
| `plan` | `{kind:"plan", steps:[{id,text}], meds:[{id,name,note}]}` – eigener Kolik-Plan; fehlt er, gilt `DEFAULT_PLAN` |
| `episode` | `{kind:"episode", start:ms, side, symptoms:[], events:[…]}` – nur solange eine Kolik läuft |

Attack-Einträge, die aus dem Kolik-Modus kommen, haben zusätzlich `log:[{t:ms, type:"pain"\|"med"\|"step"\|"sym"\|"note", v?, text?, step?}]`.

localStorage-Key: `nierenstein-v1` (Objekt ID → Daten); Fragen an den Arzt in `nierenstein-v1-q`.
Die Speicher-Schicht ist in `initStore()` gekapselt (`backend.set(id, data)` / `backend.del(id)`),
dort lässt sich ein anderes Backend einhängen.

## Erledigt (Oktober 2026)
- **Sichern / Wiederherstellen** (Tab „Für den Arzt“ → „Daten sichern“):
  - „Sicherung speichern“ lädt `nierenstein-sicherung-YYYY-MM-DD.json` herunter:
    `{app:"nierenstein-tagebuch", version:1, exportedAt, docs:{<ID>:<Daten>}, questions}`
  - „Sicherung laden“ zeigt erst eine Zusammenfassung, übernimmt nach Bestätigung.
    Zusammenführen statt Ersetzen: nichts wird gelöscht, bei gleichem Tag gewinnt die
    höhere Glaszahl. Jedes Dokument wird geprüft und normalisiert (`cleanDoc`),
    Unbekanntes wird verworfen. Das rohe localStorage-Objekt wird ebenfalls akzeptiert.
  - „Anfälle als Tabelle (CSV)“: Semikolon, Komma-Dezimalen, UTF-8-BOM → öffnet in
    Excel (de-AT) direkt richtig.
- **Bericht als PDF:** „Drucken / PDF“ druckt ein eigenes Layout (`#printout`, nur per
  `@media print` sichtbar): Kennzahlen, Tabelle der Anfälle, Trinkmenge 14 Tage, Fragen,
  Leerzeilen für Notizen beim Arzt. PDF über „Als PDF speichern“ im Druckdialog.
- Vollständiger HTML-Kopf ergänzt (`<meta charset="utf-8">`, viewport, `lang="de-AT"`),
  sonst werden Umlaute beim direkten Öffnen der Datei falsch angezeigt.
- Hinweis: Im claude.ai-Artifact können Downloads je nach Browser blockiert sein; als
  normale Datei/Website funktionieren sie.

- **Als Website mit Homescreen-Icon (PWA-light):** Projekt liegt jetzt in
  `C:\Dokumente\Htl Maturaklasse\App`, Hauptdatei heißt `index.html`.
  - `manifest.webmanifest` (Name, Vollbild ohne Adressleiste, Farben, Icons)
  - `icons/` – weißer Tropfen auf Petrol (#1f7a8c): 192, 512 (auch maskable), 180 (Apple), 32 (Favicon)
  - `sw.js` – Offline: Seite network-first (Updates kommen sofort an), Icons/Manifest und
    Google Fonts cache-first. Bei Änderung an Icons/Manifest `VERSION` hochzählen.
    Registriert sich nur über https/localhost.
  - `navigator.storage.persist()` beim ersten Eintrag, damit der Browser die Daten nicht wegräumt.
  - Speicher bleibt bewusst localStorage (reicht für die Datenmenge, kein Umbau nötig).
- Wichtig: Speicher hängt an der genauen Adresse (Domain + Pfad). Adresse nach dem Launch
  nicht mehr ändern, sonst vorher Sicherung speichern und am neuen Ort laden.

- **Zwei Bereiche (einfache Bedienung):**
  - *Tagebuch* für den Alltag – Leiste unten mit nur zwei Knöpfen: Trinken (`#heute`) und Kolik (`#kolik`).
  - *Verwaltung* – Knopf oben rechts (`#verwaltung`), Menü mit Kolik-Plan & Medikamente (`#plan`),
    Anfälle (`#anfaelle`, alt `#verlauf`), Bericht für den Arzt (`#arzt`), Einstellungen & Sicherung
    (`#daten`, inkl. Trinkziel). Innerhalb der Verwaltung wird der Knopf zu „✓ Fertig“ → zurück ins Tagebuch.
  - Zurück-Taste am Handy funktioniert über die Hash-Routen. Bugfix: `.panel{display:grid}`
  hatte das `hidden`-Attribut überschrieben, alle Bereiche waren gleichzeitig sichtbar
  (jetzt globale Regel `[hidden]{display:none!important}`).
- **Kolik-Modus:** eigener Plan (Checkliste) + eigene Medikamente mit Notiz (keine Dosierungen).
  Während der Kolik: Timer, Schmerz 1–10, „Medikament genommen“ mit „zuletzt vor …“, Plan abhaken,
  Seite/Beschwerden, Notizen, Verlauf mit Entfernen. Doppeltipp innerhalb 3 s wird ignoriert.
  Läuft eine Kolik, öffnet die App direkt dort; auf anderen Tabs erscheint ein Balken.
  „Kolik ist vorbei“ → Anfall mit Dauer, stärkstem Schmerz und `log`, Formular öffnet sich zum Ergänzen.
  Bericht, Druck und CSV zeigen Medikamente mit Uhrzeit, Schmerzverlauf und erledigte Schritte.
- **Teilen** über das Teilen-Menü des Handys (`navigator.share`, sonst Kopieren):
  „Stand teilen“ während der Kolik (Dauer, letzter Schmerz, Medikamente mit Uhrzeit, Beschwerden)
  und „Bericht teilen“ beim Arztbericht.
- Anfälle: Liste oben, Formular „+ Nachtragen“ eingeklappt.

## Nächste Schritte (Wünsche)
1. ~~Alles sichern / wiederherstellen~~ erledigt
2. ~~Bericht als PDF~~ erledigt
3. ~~Als App installierbar machen~~ erledigt (als Website + Homescreen-Icon, siehe oben)
4. Optional: Erinnerung zum Trinken (Push-Benachrichtigung), Sync zwischen Geräten.
5. Optional: Felder für Steinanalyse-Ergebnis und Medikamente.

## Wichtig beim Weiterbauen
- Keine medizinischen Dosierungen oder Diagnosen in die App einbauen.
- Gesundheitsdaten nur lokal bzw. privat speichern, nichts an fremde Server schicken.
- Bestehende Daten (Schlüssel oben) beim Umbau weiter lesen können, damit nichts verloren geht.
