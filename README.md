# werk-vorlagen

Gemeinsame Bausteine für alle Seiten. Die Logik liegt einmal hier statt
verstreut in jedem Seitenrepo — eine Verbesserung hier erreicht alle Seiten.

Öffentlich, weil es keine Geheimnisse enthält und weil wiederverwendbare
Workflows aus öffentlichen Repositorien ohne Sonderregeln funktionieren.

## Eine Seite anschließen

**1. `auslieferung.toml` ins Repo legen:**

```toml
name     = "koenig"      # Kurzname, taucht im Paketnamen auf
art      = "sftp"        # rsync | sftp | ftp | dienst
host     = "wp123.webhosting.example"
pfad     = "/html"
bauen    = ""            # leer = keine Bauschritte, Quelle ist das Repo
ergebnis = "."           # bei Astro: "dist"
rohlogs  = ""            # Pfad, wenn der Anbieter Rohlogs herausgibt
beacon   = "cname"       # cname | fremd | keins
```

`host`, `pfad` und `rohlogs` liest nur der `ausroller` auf der Werkstatt.
Die **Zugangsdaten stehen nicht hier** — sie liegen in
`/etc/werk/ziele/<name>.env` auf der Werkstatt. Der Kunde ist Vertragspartner
seines Hosters; das sind fremde Zugangsdaten, die nicht auf GitHub gehören.

**2. `.github/workflows/deploy.yml` anlegen:**

```yaml
name: Bauen und melden
on:
  push: { branches: [main] }
  workflow_dispatch:
jobs:
  bauen:
    uses: felixrath/werk-vorlagen/.github/workflows/bauen-und-melden.yml@main
    secrets: inherit
```

Das ist alles, was im Seitenrepo steht.

## Was die Vorlage tut

1. `auslieferung.toml` lesen und prüfen
2. bauen, sofern `bauen` gesetzt ist — davor `npm test` und `astro check`,
   sofern vorhanden
3. die Fassung als `<meta name="fassung" content="a3f2c19">` in jede
   HTML-Datei stempeln
4. das Ergebnis als `tar.gz` an ein Release hängen
5. der Werkstatt Bescheid geben

**Ausgeliefert wird hier nichts.** Das tut der `ausroller` auf der Werkstatt,
mit seinen eigenen Zugangsdaten. Der Grund steht oben.

## Der Fassungsstempel

Er ist der einzige verlässliche Beleg, dass beim Kunden auch wirklich das
Neue steht — auf FTP-Zielen gibt es keine Atomarität und sonst keinen Beweis.
Der `ausroller` ruft nach dem Hochladen die Startseite ab und vergleicht.

Dieselbe Zeile dient später der Wache: weicht die Fassung auf einer
Kundenseite von der zuletzt ausgelieferten ab, hat jemand von Hand etwas
geändert — oder etwas Schlimmeres.

## Geheimnisse

| Name | Wofür | Pflicht |
|---|---|---|
| `MELDEN_URL` | Adresse des `ausrollers` | nein — ohne meldet die Vorlage nicht, der `ausroller` fragt selbst nach |
| `MELDEN_GEHEIMNIS` | HMAC-Signatur der Meldung | nur zusammen mit `MELDEN_URL` |

Beide gehören als Organisations- bzw. Kontogeheimnis hinterlegt, nicht je
Repo. Sonst sind es wieder zwanzig Stellen.
