# Plukbos Kardinge, kaartviewer

Open-source kaartviewer voor de bomen in het plukbos bij Kardinge in Groningen. De kaart laat zien waar elke boom staat, wanneer hij vrucht draagt en hoe gezond hij is. De bomen zijn geïnspecteerd met [Boomwacht](https://www.geominds.nl/nl/products/boomwacht/), de boominspectie-app van [Geominds](https://www.geominds.nl/nl/).

De viewer is een statische website: HTML, CSS, JavaScript en twee JSON-bestanden. Er is geen database, geen build-stap en geen servercode. Elke webserver die bestanden kan serveren, kan hem hosten.

## Structuur

```
index.html              de pagina
css/viewer.css          opmaak
js/viewer.js            kaart en bediening; bovenin staat het blok INSTELLINGEN
data/bomen.geojson      de bomen uit de inspectie, één punt per boom
data/soorten.json       oogstperiode en Latijnse naam per soort en ras
data/foto/              foto's per boom
lib/leaflet/            Leaflet 1.9.4, lokaal meegeleverd
fonts/                  Atkinson Hyperlegible en Newsreader, lokaal meegeleverd
img/                    Geominds-logo en merkteken
LICENSE                 EUPL-1.2, voor de code
LICENSE-DATA            CC BY 4.0, voor alles in data/
NOTICE                  naamsvermelding en onderdelen van derden
```

Alles draait vanaf de eigen server. Alleen de ondergrond (luchtfoto en kaart) komt live van PDOK, de publieke kaartdienst van de Nederlandse overheid.

## Lokaal bekijken

Browsers laden de gegevens niet als je `index.html` dubbelklikt. Start daarom een eenvoudige webserver in deze map:

```
python3 -m http.server
```

en open `http://localhost:8000`. Op Windows heet Python meestal `py`:

```
py -m http.server --bind 127.0.0.1
```

## Hosten

Kopieer de hele map naar een webserver, bijvoorbeeld als submap van een bestaande website. Codeberg Pages of GitHub Pages werken ook: zet de repository daar neer en zet Pages aan.

Zet in `js/viewer.js` bij `broncodeUrl` het adres van de repository. Dan verschijnt in het venster "Over deze kaart" een link naar de broncode.

## Gegevens bijwerken

### Bomen: `data/bomen.geojson`

Een standaard GeoJSON FeatureCollection met punten in WGS84 (lengtegraad, breedtegraad). Opent direct in QGIS en op geojson.io. Na een nieuwe inspectie vervang je dit bestand door de export.

| Veld | Verplicht | Inhoud |
|---|---|---|
| `id` | ja | uniek boomnummer, bijvoorbeeld `b017`; wordt ook de link naar de boom (`#b017`) |
| `soort` | ja | Nederlandse soortnaam, gelijk aan een sleutel in `soorten.json` |
| `ras` | nee | rasnaam, bijvoorbeeld `Elstar` |
| `gezondheid` | ja | `goed`, `matig` of `slecht`; iets anders wordt `onbekend` |
| `inspectiedatum` | ja | datum als `JJJJ-MM-DD` |
| `foto` | nee | bestandsnaam in `data/foto/`, of een volledige URL |
| `opmerking` | nee | vrije tekst, verschijnt op het kaartje van de boom |
| `oogst_van`, `oogst_tot` | nee | maandnummers 1 tot en met 12; overschrijven de periode uit `soorten.json` voor deze ene boom |
| `wetenschappelijk` | nee | Latijnse naam; overschrijft die uit `soorten.json` |

Staat bovenin het bestand `"metadata": { "voorbeeld": true }`, dan toont de viewer de melding "Voorbeeldgegevens". Haal die regel weg zodra de echte inspectie erin staat.

### Oogstperiodes: `data/soorten.json`

De inspectie legt de oogstperiode niet vast; die hoort bij de soort en het ras. Daarom staat hij in een eigen bestand, dat bij een nieuwe inspectie gewoon blijft staan.

```json
"Appel": {
  "wetenschappelijk": "Malus domestica",
  "oogst": { "van": 8, "tot": 10 },
  "rassen": {
    "Elstar": { "oogst": { "van": 9, "tot": 10 } }
  }
}
```

De viewer zoekt eerst de periode van het ras, dan die van de soort. Een periode mag over de jaarwisseling lopen, bijvoorbeeld `{ "van": 11, "tot": 1 }`. Nieuwe soort in de inspectie? Voeg hier een blok toe met dezelfde naam als in `bomen.geojson`.

### Foto's: `data/foto/`

Maak foto's kleiner en haal de metadata (tijdstip, toestel, soms GPS) eruit voordat je ze publiceert. Met ImageMagick in één keer voor de hele map:

```
mogrify -resize '1200x1200>' -strip -quality 80 data/foto/*.jpg
```

Een foto van 1200 pixels is ongeveer 150 kB en scherp genoeg voor het kaartje.

### Posities controleren

Telefoon-GPS zit onder bladerdek al snel een paar meter naast de boom. Controleer de posities na een inspectie in QGIS: laad `bomen.geojson` samen met de PDOK-luchtfoto (Actueel_orthoHR) en schuif punten die bij de buurboom zijn beland naar de juiste kruin. Sla op als GeoJSON in WGS84.

## Instellingen

Bovenin `js/viewer.js` staat het blok `INSTELLINGEN` met:

- de paden naar de gegevens en foto's
- de links naar Geominds en naar de productpagina, en de productnaam
- de link naar de broncode
- het startpunt van de kaart en het zoomniveau bij het openen van een boom

Meer hoeft er normaal niet aangepast te worden.

## Wijzigingen in de code doorvoeren

Browsers bewaren `css/viewer.css` en `js/viewer.js` in hun cache. In `index.html` staat daarom een versie achter beide bestanden, bijvoorbeeld `viewer.js?v=2026-09-20`. Zet daar na elke wijziging in `css/` of `js/` een nieuwe datum neer; dan halen bezoekers de nieuwe versie op.

De gegevens in `data/` worden altijd vers geladen. Daarvoor hoeft niets aangepast te worden.

Zie je zelf na een wijziging nog de oude versie, druk dan op Ctrl+F5 (Cmd+Shift+R op een Mac).

## Licenties en naamsvermelding

De code valt onder de [EUPL-1.2](LICENSE). De gegevens en foto's in `data/` vallen onder [CC BY 4.0](LICENSE-DATA). Wie de gegevens hergebruikt, vermeldt daarbij: **Inventarisatie: Geominds (https://www.geominds.nl)**.

De viewer toont Geominds op vier plekken: als badge linksonder op de kaart, in de voet van het paneel, onderaan elk boomkaartje en in het venster "Over deze kaart".

Het Geominds-logo en het merkteken in `img/` vallen niet onder de EUPL. Ze mogen alleen gebruikt worden om naar Geominds te verwijzen.

Meegeleverde onderdelen van derden staan in [NOTICE](NOTICE): Leaflet (BSD-2-Clause) en de lettertypen Atkinson Hyperlegible en Newsreader (SIL Open Font License 1.1).
