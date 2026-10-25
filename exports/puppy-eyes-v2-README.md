# puppy-eyes-v2

En separat alternativ følelsespakke med 21 geometrisk tegnede ansiktsanimasjoner.
Originalpakken er fortsatt standard. Ingen firmware, layout, aktiv emote.json,
original EAF-fil eller systemressurs er endret av denne pakken.

## Innhold

- `face.py`: felles parametrisk ansikt, masker for pupiller/lysrefleks og øyelokk.
- `animations.json`: redigerbare beskrivelser, nøkkelbilder, pauser og varigheter.
- `generate.py`: regenererer kandidater direkte fra tidslinjen, uten å miste GIF-pausebilder.
- `emoji/puppy-eyes-v2-*.eaf`: ferdige EAF-filer. Prefikset unngår å overskrive originalfilene når kandidaten skal prøves.
- `gif/`: praktiske animerte kilde-/forhåndsvisningsfiler. GIF kan slå sammen identiske bilder; EAF-tidslinjen kommer fra animations.json, ikke GIF-forsinkelsene.
- `stills/` og `contact-sheet.png`: representative stillbilder.
- `240_240/emote.json`: kandidatmanifest; bare de 21 følelsene peker til nye filer.
- `report.json`: bildeantall, FPS, varighet, filstørrelse mot originalen og SHA-256.
- `original-hashes.json`: fingeravtrykk for 30 originale EAF-filer og aktiv konfigurasjon/layout fra før arbeidet.
- `asset-check.json`: resultat fra faktisk ressurspakking, ikke en fysisk maskinvaretest.

De ni systemfilene ligger ikke i kandidatens emoji-mappe. De brukes uendret fra
originalpakken når man bygger. Wi-Fi-varianten velges fortsatt av regionen.

## Regenerering

Fra prosjektroten, med Python og Pillow (testet med Pillow 13.0.0):

```sh
python3 tools/expression_sources/puppy/puppy-eyes-v2/generate.py
python3 tools/preview_expressions.py
node --test tools/expression_viewer/test_codec.cjs tools/expression_viewer/test_packs.cjs
```

Endre følelsenes `pose`-egenskaper og nøkkelbildetider i animations.json. Alle
uttrykk bruker samme geometriske tegnekode og palett. Bruk `--only neutral,happy`
for rask utvikling, men kjør hele genereringen før levering/rapportering.

EAF-koderen fra tools/gif_to_eaf.py gjenbrukes. Kandidatene bruker 240×240,
8-bits fast palett, RLE, RGB565-tilpasset visning og 30 FPS fra manifestet. Bildene
har svart bakgrunn og ingen innbakt skjermkant. Avsluttende _C-markør er ikke en
bildeframe. Identiske pausebilder kan referere til samme immutable EAF-data;
alle tidslinjebilder beholdes, så pauser og varighet er uendret. Start/sluttpose
inngår i det avtalte bildeantallet; ingen ekstra sluttsekvenser legges til.

## Forhåndsvisning

Pakk ut leveransen og åpne `puppy-viewer.html`, eller åpne den nygenererte
`build/puppy-viewer.html`. Vieweren starter alltid med Original valgt.

Velg `puppy-eyes-v2` i Pakke og en følelse i listen. Den andre pakken vises ved
siden av. Begge bilder dekodes fra de faktiske EAF-filene, ikke GIF-ene. De
spilles med hver sin opprinnelige varighet. Kontrollene gir pause, omstart,
rammesteg, tidslinje, FPS, looping, rund maske, PNG-eksport og løkkeovergang.

1× viser 240×240 når det er plass. Slå av Sammenlign ved stor zoom på smal
skjerm. Masken er kun visning, ikke del av eksporten. Systemuttrykk merkes som
uendret og viser originalgrafikken uansett pakkevalg.

## Prøv på robot og gå tilbake

Firmware har ingen generell pakkevelger: main/CMakeLists.txt bruker Puppys
aktive mappe/manifest. Vi har derfor ikke innført en ny firmwaremekanisme.
Kandidatmanifestet og prefiksfilene gir en enkel manuell, reversibel prøve.

Disse kommandoene er veiledning til en senere, eksplisitt aktivering. De er
IKKE kjørt under leveransen. Kontroller og bevar egne endringer først.

```sh
# Ta vare på den aktive konfigurasjonen før endring.
mkdir -p outputs/puppy-eyes-v2-backup
cp main/boards/puppy/240_240/emote.json outputs/puppy-eyes-v2-backup/emote.json

# Legg til kandidatfiler med egne navn; ingen originale EAF-filer overskrives.
cp tools/expression_sources/puppy/puppy-eyes-v2/emoji/*.eaf main/boards/puppy/emoji/
cp tools/expression_sources/puppy/puppy-eyes-v2/240_240/emote.json main/boards/puppy/240_240/emote.json

# I et aktivert ESP-IDF 5.5.3-miljø, med Puppy/esp32s3 valgt:
idf.py -B build/puppy -DIDF_TARGET=esp32s3 reconfigure
ninja -C build/puppy -j4
```

Flash med riktig fysisk serieport og prosjektets IDF-arbeidsflyt. Inkluder
assets-partisjonen; application-only-flashing oppdaterer ikke animasjonene.
Ikke bruk erase-flash som et normalt byttetrinn. GPIO-er, layout, system- og
QR-filer skal ikke endres.

Tilbake til den opprinnelige konfigurasjonen:

```sh
cp outputs/puppy-eyes-v2-backup/emote.json main/boards/puppy/240_240/emote.json
idf.py -B build/puppy -DIDF_TARGET=esp32s3 reconfigure
ninja -C build/puppy -j4
```

Flash den gjenoppbygde ressurspakken. Ubrukte prefiksfiler kan bli liggende;
de velges ikke når manifestet er tilbake. Originalene er aldri overskrevet.
Kjør ikke baseline-hashtesten mens du bevisst har aktivert kandidatmanifestet.

## Ressurs- og kvalitetskontroll

`report.json` og `STORRELSER.md` sammenligner hver følelse. Med et tidligere
bygd originalmiljø kan kandidatpakken kontrolleres uten aktivering:

```sh
python3 tools/expression_sources/puppy/puppy-eyes-v2/check_asset_pack.py
```

Verktøyet bruker prosjektets eksisterende packer i et isolert område under
build/. Det kopierer nødvendige systemfiler dit, med identisk innhold, og lar
aktiv konfigurasjon og originalens byggartefakter være urørt. Testen gjelder
domestic-regionen, eksisterende font og WakeNet-modeller fra originalbygget.

Validering omfatter dekoding/kontrollsum, dimensjoner, opacity, tid, pauser,
løkke, sikker sirkelmargin og uendret originalgrafikk. Det er ikke bekreftet
avspilling på fysisk Puppy. Langvarig FPS, minnepress, skjermorientering og
samspill med lyd må testes på maskinvare. Vieweren emulerer ikke disse forholdene.

LLM-integrasjonen bruker de samme følelsesnavnene som før. Ingen nye servernavn
eller promptendringer kreves for å velge eksisterende følelser. At produsentens
server faktisk sender f.eks. proud er fortsatt avhengig av serveroppsettet.
