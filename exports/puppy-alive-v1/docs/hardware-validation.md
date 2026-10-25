# Maskinvare, tester og senere aktivering

## Verifisert i valgt konfigurasjon/kilde

- ESP32-S3; defaults velger 16 MiB QIO-flash, Octal PSRAM ved 80 MHz.
- Offisiell utviklerside beskriver N16R8 (16 MiB flash / 8 MiB PSRAM).
  Faktisk montert modul og fri heap må avleses på mottatt robot.
- Puppy-kode initialiserer GC9A01, 240×240 og 16 bpp/RGB565.
- SPI3, konfigurert 80 MHz klokke og transaksjonskø 10. Dette er ingen måling
  av faktisk panel-FPS eller elektrisk stabilitet.
- Emote har 30 FPS standard, doble 240×16 PSRAM-striper, 6 KiB ekstern taskstack
  og to interne DMA-bounce-buffere. EAF-data kan i tillegg caches.
- partitions/16m.csv: to OTA appslots à 0x3f0000, assets på 0x800000 størrelse 8M.

## Testnivåer

Implementert: generator, profiler, EAF/GIF/PNG, manifester, viewer, host-engine.
PC-testet: faktiske bildefiler, loop, dimensjoner, sirkelmargin, ingen tomme
frames, rammelikhet, JS-dekoder, browser uten nett og lokal file://, state-machine.
Validert mot embedded kildekode: uendret installert C-dekoder kompilert på host;
original asset-packer kjørt isolert med original font, layout og WakeNet.
Fysisk testet: **ingenting** i Alive. Ingen firmware er flashet.

Rapporter angir tall og resultater. Automatiske fingeravtrykk bekrefter ikke
at alle mennesker tolker hver følelse riktig; kontaktark og bevegelsesprøver
inngår i visuell vurdering. PC-rasterytelse er ikke en ESP32-ytelsesmåling.

## Før senere integrasjon

1. Ta full backup av firmware, NVS og original assets, og noter board/region.
2. Avles modul, panelrevisjon, faktisk flash/PSRAM og oppstartslogger.
3. Bruk standardprofilens separate manifest. Kopier bare prefiksfilene
   puppy-alive-v1-*.eaf fra standard staging til Puppy emoji-katalogen, og bytt
   manifest manuelt i en egen testgren med original backup. Ikke bruk alle
   PC-intensiteter i 8 MiB-partisjonen.
4. Bygg med samme board/region. Flash først etter separat beslutning når roboten
   er tilgjengelig. Denne leveransen utfører ingen av disse aktiveringene.
5. Test oppstart, Wi-Fi begge regioner, kalibrering, reset, remote og listen før
   samtale. Test ukjent navn, samtalestatus som avbryter emotion og reconnect.
6. Mål fri/minste intern heap, PSRAM, største allokerbare blokk, frame pacing,
   lyd samtidig, varme og watchdog. Test lang neutral-loop og hyppige skifter.
7. Gjenopprett originalmanifest og bygg/flash originale assets for rollback.

Kandidatpakken er kontrollert med domestic Wi-Fi-ressurs. International original
fil bevares og vises i gammel viewer, men international kandidat-packing må
kjøres særskilt før slik firmware distribueres. Extended har mindre plassmargin
og er ikke standardvalget. Ingen motor, kamera, mikrofon, lyd, OTA eller
serverendringer inngår. Dynamiske overganger krever separat runtime-arbeid.
