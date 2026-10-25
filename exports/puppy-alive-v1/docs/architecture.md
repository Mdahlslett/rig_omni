# Arkitektur og videreutvikling

## Kilde og ansvar

- `emotions.py`: én definisjonstabell med alle 29 navn, tre parametriske nivåer,
  kompatibilitetsmapping og følelsesspesifikke mikrosekvenser. Gjenbruker V2s
  anatomiske rigg og nøkkelposisjoner fra redigerbar JSON, aldri base64.
- `animation.py`: 20 kodeprimitiver, timing, easing, grenser og deterministisk
  idle-plan. Hver hendelse har varighet, styrke, forsinkelse, easing og valgfri
  liste over tillatte følelser.
- `rendering.py`: gjenbruker V2-renderer; forbedrer blunk ved å komprimere
  øyehøyde og gli over i lukket lokk. Kvantiserer til RGB565-ekvivalente farger.
- `transitions.py`: ti profiler og interpolasjon mellom vilkårlige posisjoner.
- `export.py`: PNG, GIF, EAF, manifest, hash-baseline og ressursmetadata.
- `validate.py`, JS-codec-test og C-dekodertest: uavhengige kontroller.
- `package.py`: lokal viewer, tre pakker, faktisk original asset-packer.

V2 er en avhengighet og beholdes intakt. Ny renderer kopierer ikke en ekstern
ansiktspakke. Oppdater BASE eller CUES sentralt for å endre uttrykk, og kjør
hele eksporten og testene på nytt. Ikke håndrediger generert data.js/EAF.

## Intensitet

Low/medium/high skalerer avvik fra grunnanatomien med .55/1/1.3. Bevegelses-
amplitude, øyegeometri, asymmetri og mikrostyrke påvirkes. Varighet skaleres
1.14/1/.88, avrundet til hele 30-FPS-ticks. Pupill og øyelokk har harde grenser.
Ingen tre kopier av hver følelsesimplementasjon. Nøytral beholder liten forskjell
mellom nivåene og rolige pauser. Variantenes visuelle lesbarhet må også vurderes
av mennesker; bildefingeravtrykk beviser ikke følelsesgjenkjenning.

## Loop og tid

Første/siste kildepose er like. 30 FPS er manifestets tidsbase; GIF bruker
30/40 ms avrunding og kan slå sammen like holdbilder. EAF-tabellen beholder
hver tick, men deler identiske rammepayloads. Samme seed gir samme PNG/EAF.
Overganger er eksplisitt ikke-loopende og har ulike endepunkter.

## Fremtidig engine – kun hostimplementasjon nå

`tools/expression_viewer/alive/engine.js` er kjørbar PC-tilstandsmaskin.
Prioritet: systemlås > aktiv lokal/LLM-følelse med utløp > samtalestandard
(listening→listen, thinking→thinking) > neutral/idle. Speaking genererer ikke
munnbevegelser. Systemmelding avviser nye følelser; det brukes ingen skjult kø.
Tid fortsetter under systemlås; utløpt følelse går til neutral.

Tilstand: current, intensity, deadline, attention, conversation, system, monotonic now og
historikk. Oppmerksomhet velger low/medium/high ved eksplisitt idle-start i PC-demoen;
kontinuerlig sensorstyrt oppmerksomhet er ikke implementert. Vieweren interpolerer aktuell geometri til ny startpose med valgt
profil. Forhåndsdefinerte overgangsklipp bruker navngitte kilde-/målposer.

Fremtidig embedded adapter kan beholde denne lille state-modellen og velge
forhåndspakkede klipp. Dynamisk tilfeldig idle og vilkårlig geometrisk
interpolasjon krever ny firmwarefunksjonalitet. Ingen slik runtime er lagt til.
`idle_schedule(seed, seconds)` spesifiserer en begrenset vektet plan:
blunk prioriteres, hendelser holdes innenfor klippet, pauser er 2.2–4.1 s.
Et senere system må bevare blunkfase og unngå å starte konkurrerende reaksjoner.

## Ressurser og eksportprofiler

`standard`: 21 medium-følelser som erstatter lokale navn i et separat kandidat-
manifest, inkludert det tidligere lokale proud; systemposter beholdes eksakt.
`extended`: de åtte nye navnene i tillegg, fortsatt bare medium.
`extended-intensity`: alle 87; avvises dersom ressursene overskrider partisjonen.
`preview`: alle 117 klipp, kildeframes og tre pakker på PC, ingen flashgrense.

Se resources.json for faktisk packerstørrelse, ikke bare sum av EAF-filer.
Full RGB565-ramme er 115200 byte. Dagens doble 240×16-striper bruker 15360 byte
PSRAM; to DMA-bounce-buffere bruker ytterligere 15360 byte intern RAM. Dette er
beregnede skjermbuffere, ikke total heap. Dekoder, aktiv EAF-cache, stack,
nettverk, kamera og lyd kommer i tillegg. Browser-RGBA bruker 230400 byte per
unik ramme; bare to valgte EAF-klipp dekodes samtidig. All base64 er likevel
lastet som JS-data, så PC-minneforbruk kan være langt større enn ZIP-størrelsen.
Ingen målt ESP32-FPS eller RAM-topplast påstås.
