# Korrigert Puppy-viewer

Pakk ut HELE ZIP-filen og åpne START-HER.html i en vanlig nettleser.
Behold app.js, codec.js, viewer-data.js og previews-mappen ved siden av HTML-filen.
Ikke åpne HTML direkte inne i ZIP-filen eller source/index.html.

Først vises vanlige GIF-animasjoner. Åpne pakken og navnet på følelsen.
Disse vises uten JavaScript og uten canvas. Original er standard.
Åpne Avansert EAF-spiller for pakkevalg, sammenligning, pause, omstart,
rammesteg, tempo, looping, maske og eksport av PNG.

Alle 51 GIF-forhåndsvisninger er laget fra EAF-filene. Alle 2393 bildeframes
og GIF-tidsforløp er pikselkontrollert mot EAF-dekoderen. GIF-tid er avrundet
med 30/40 ms-intervaller der FPS er 30; EAF-spilleren bruker manifestets FPS.

Testet på den faktiske ZIP-filen etter utpakking i Chromium på Linux:
- 51 GIF-forhåndsvisninger åpnes med JavaScript AV.
- Begge pakkevalg og alle 60 listevalg gir synlig EAF-grafikk offline.
- Avanserte kontroller, pause ved lukking og mobilbredde er kontrollert.
- 11 dekoder-/pakketester består; originalressurser/konfigurasjon er uendret.
- Repeterte pausebilder deler dekodet minne: v2 neutral bruker 23 bildebuffere
  for 135 tidslinjebilder, fremfor 135 buffere.

Den tidligere store enkeltfil-HTML-en erstattes i denne leveransen av en
liten HTML-inngang og relative filer. EAF-spilleren lastes ved behov, og den
vanlige GIF-visningen kan brukes selv om scripts eller canvas ikke virker.
Vi har ikke kunnet bekrefte den konkrete årsaken på brukerens PC.
Firefox og WebKit kunne ikke kjøres i testmiljøet på grunn av manglende
plattformstøtte/biblioteker; det er ikke påstått at disse er nettlesertestet.
Ingen fysisk Puppy er testet.

Redigerbare kilder, EAF-filer, kandidatkonfigurasjon og originalgrunnlag finnes
i den komplette pakken. Dette er ikke en automatisk aktivering eller et fullt
firmwareprosjekt. Les tools/expression_sources/puppy/puppy-eyes-v2/README.md.
