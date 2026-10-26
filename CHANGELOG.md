# Changelog

Alle nennenswerten Änderungen an diesem Modul werden hier dokumentiert.
Format angelehnt an [Keep a Changelog](https://keepachangelog.com/de/1.0.0/).

## [Unreleased]

### Added
- **Archiv-Faktor bei der Übernahme (`archiveFactor`):** Liefert das Zielmodul im Rückgabewert
  von `<PREFIX>_GetIdentMapping` zusätzlich das optionale Feld `'archiveFactor' => <Zahl>` (z. B.
  `1000.0` für kW→W), rechnet Gerätetausch beim Übernehmen alle archivierten Alt-Werte dieser
  Variable damit um (`neu = alt × Faktor`). Hintergrund: bei der Übernahme vom alten
  go-e-Charger-Modul auf ChargerHub landete die Leistungshistorie in kW im W-Profil, alte Ladevorgänge
  erschienen um Faktor 1000 zu klein. Neue Spalte „Archiv-Faktor" in der Migrationsliste
  (vom Nutzer änderbar, 1 = keine Umrechnung, fehlendes Feld = 1, also abwärtskompatibel). Sicherungen:
  vor jeder Umrechnung wird eine Sicherungsdatei aller Originalwerte angelegt
  (`Geraetetausch-Sicherung/` im IP-Symcon-Verzeichnis, bei Fehlschlag Abbruch ohne Änderung);
  je Zeitfenster wird nach dem Schreiben nachgezählt und bei Abweichung aus den Originalwerten
  wiederhergestellt; dieselbe Variable wird nie ein zweites Mal umgerechnet (Attribut
  `ArchiveRescaled`); umgerechnet werden nur Werte bis zum Beginn der Übernahme (spätere schreibt
  das Zielmodul bereits in der neuen Einheit); danach `AC_ReAggregateVariable`. Gilt für den
  Übernahme-Weg (inkl. dessen Rückfall auf `AC_ChangeVariableID`), nicht für das reine
  „Verknüpfen" im Erweitert-Panel. Der Probelauf meldet vorab die Anzahl der betroffenen Werte.
  Live gegen das Archiv geprüft (Wegwerf-Variable: Fenster-Teilung, Umrechnung, Nachzählung,
  Doppel-Schutz).

### Changed
- **Übernahme lehnt Ziele mit vorhandener Archivhistorie ab:** Die Übernahme löscht die frische
  Zielvariable samt Archiv. Hat sie bereits mehr als 10 archivierte Werte (z. B. von Hand
  übernommene/korrigierte Historie), wird die Übernahme jetzt mit klarer Meldung abgelehnt,
  statt diese Historie stillschweigend zu verlieren. Einzelne Werte bleiben unkritisch (schon das
  Einschalten der Archivierung schreibt einen Wert).
- **Modul umbenannt: MigrationsHub/„Vormund" → Gerätetausch** (13.10.2026, DG65-Toolkit-weite
  Umstellung auf Funktionsnamen statt Personen-/Behördenbilder). PHP-Klasse, Modulordner,
  Präfix und Repository wurden mit umbenannt — `MigrationsHub` → `Geraetetausch`, `MIGHUB_` →
  `GTAUSCH_`, `github.com/DG65/NRGMigrationsHub` → `github.com/DG65/Geraetetausch`. Modul- und
  Bibliotheks-GUID bleiben unverändert, damit bestehende/künftige Instanzen über dieselbe GUID
  weiterhin funktionieren. Alle bisherigen `MIGHUB_*`-Funktionen (inkl. der unten dokumentierten
  `*Ex`-Varianten) heißen jetzt `GTAUSCH_*`, Parameterzahl/-reihenfolge unverändert. Zum
  Zeitpunkt der Umbenennung lief bei Dietmar keine Instanz dieses Moduls mehr — kein
  Live-Umstieg nötig, nur die bekannten Discovery-Aufrufer (MeterHub/InverterHub/ChargerHub)
  mussten auf den neuen Präfix (mit `MIGHUB_`-Rückfall) umgestellt werden.
- **Verhaltensänderung `MIGHUB_RunAdoptions($confirmed, $migrations)`:** Diese Funktion lehnt
  jeden Lauf jetzt grundsätzlich ab und verweist auf `MIGHUB_RunAdoptionsEx($confirmed,
  $riskAcknowledged, $migrations)`. Grund: Symcons Kernel-Wrapper haben eine feste Arität —
  der neu eingeführte Risiko-Schalter (`$riskAcknowledged`, Einverständnis zur Übernahme bei
  Fremdmodulen) durfte deshalb nicht einfach als weiterer Parameter an die bestehende Funktion
  angehängt werden, ohne bestehende Aufrufer zu brechen. Die alte Funktion bleibt aus
  Kompatibilitätsgründen aufrufbar, führt aber bewusst keine Übernahme mehr blind ohne
  Risiko-Bestätigung aus — bitte auf `RunAdoptionsEx` umstellen.
- `MIGHUB_AddSourceVariablesToMigrations($sourceVariables, $migrations, $targetInstanceID)`
  bleibt aus demselben Grund in der alten 3-Parameter-Form bestehen; die neue, um
  `$sourceInstanceID` erweiterte Fassung heißt `AddSourceVariablesToMigrationsEx`. Hier ist der
  alte Aufruf weiterhin voll funktionsfähig (fällt nur auf reinen Ident-Abgleich zurück, ohne
  Fremdmodul-Ident-Übersetzung) — keine sicherheitsrelevante Einschränkung wie bei RunAdoptions.

## [0.1.0] - 2026-07-21

### Added
- Repo-Gerüst angelegt: Modul-Skelett (`MigrationsHub/module.php`, `module.json`),
  `library.json`, README, LICENSE, CLAUDE.md, `.tools/check-standalone.php`
- Noch keine Formular-/Massenverarbeitungslogik implementiert
