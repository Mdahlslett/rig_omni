# Puppy Alive V1

Alternativ uttrykkspakke og lokal PC-simulator. Ingen firmware, originalmanifest,
original EAF, V2, motor, lyd, OTA eller LLM-integrasjon aktiveres eller endres.

## Åpne på PC

Pakk ut **hele** `puppy-alive-v1-viewer.zip`. Åpne `START-HER.html` i Chrome,
Edge eller en annen moderne nettleser. Ingen installasjon, nettverk, server eller
API-nøkkel kreves. Behold alle JS-filer og mapper sammen. `gallery.html` viser
native GIF-er også uten JavaScript. Den tidligere vieweren finnes uendret under
«Åpne den tidligere vieweren» og i `legacy/START-HER.html`.

Velg følelse, low/medium/high og pakkene på venstre/høyre side. Ikke-eksisterende
utvidelser vises som manglende, aldri under et falskt originalnavn. Begge sider
bruker samme forløpte tid, men sin egen FPS/loop/varighet. Rammetrinn er 1/30 s.
PNG-knappen lagrer venstre bilde; GIF-lenkene laster ned eksisterende GIF-er.

Playground er en lokal demonstrasjon: velg varighet og overgang, send en
JSON-melding, prøv systemprioritet, eller start en rolig personlighetssekvens.
«Mikroanimasjoner» styrer ekstra mikroreaksjoner; følelsens egne bevegelser,
f.eks. winking, beholdes. Geometrivisning brukes når mikroreaksjoner slås av
og under dynamiske overganger; ellers vises faktiske dekodede EAF-rammer.
PC-canvas kan ha andre antialiasingspiksler enn eksportens Pillow-raster.

## Bygg fra kilde

Kjør fra prosjektroten med Python 3.12+, Pillow og NumPy. Node 18+ brukes til
uavhengig EAF-validering; Playwright/Chromium kreves bare for nettlesertester.

```sh
python tools/expression_sources/puppy/puppy-alive-v1/export.py --seed 20261025 --workers 2
python tools/expression_sources/puppy/puppy-alive-v1/validate.py
node tools/expression_viewer/alive/validate_eaf.cjs
python tools/expression_sources/puppy/puppy-alive-v1/validate_c_decoder.py
python tools/expression_sources/puppy/puppy-alive-v1/package.py
```

`package.py` trenger eksisterende `build/puppy-viewer-ready`, font og
WakeNet-staging fra tidligere originalbygg. Manglende forutsetninger er feil,
ikke grunn til å endre aktiv konfigurasjon. Se `tools/build_expressions.py` for
samlet byggeinngang. Eksport havner under `build/puppy-alive-v1/`.

Full kilde-ZIP inneholder nummererte 240×240 PNG-rammer, alle GIF/EAF-klipp,
rapporter, kildekode, manifest og viewer. Tunge byggemellomfiler er utelatt.

## Omfang

29 følelser × tre nivåer = **87 følelsesanimasjoner**, pluss **20**
mikrodemonstrasjoner og **10** ikke-loopende overgangsklipp: **117 EAF/GIF-par**.
Mikroene er kodeprimitiver; demoklippene er bare test/forhåndsvisning.
Nøytral er en forhåndsgenerert 18 sekunders medium-loop med seedstyrte, ujevne
pauser. Dette er en endelig loop, ikke uendelig tilfeldig aktivitet på roboten.

Se `architecture.md`, `luwu-compatibility.md`, `emotion-mapping.md`,
`hardware-validation.md` og rapportene for presist skille mellom implementert,
PC-testet, validert mot C-kode og uprøvd på maskinvare.
