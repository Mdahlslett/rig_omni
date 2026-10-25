# Kontrollert Luwu-versjon og EAF-kompatibilitet

Kontrollert 25. oktober 2026:

| Ressurs | Commit / resultat |
|---|---|
| LuwuDynamics/rig_omni | c2b42cd73ef65f5cd2a158d6949f0273869454dc, samme som lokal HEAD |
| LuwuDynamics/rig_puppy | 5b5adc5a6343a73d9912ac9c3697c04c2f1c49f3 |
| LuwuDynamics/RIG-Muse | 8ffe1054d09b2234dfbd51c955e9bdb01e62e5a9 |

Hovedgrunnlaget er [rig_omni](https://github.com/LuwuDynamics/rig_omni/tree/c2b42cd73ef65f5cd2a158d6949f0273869454dc).
Undersøkt skjerminitialisering, board_config, EmoteDisplay, application,
CMake, partisjoner, defaults, analyze_eaf, asset-packer og installert emote/gfx-
komponent. De eldre repoenes relevante filer er lest, ikke bygget eller kjørt.

[rig_puppy](https://github.com/LuwuDynamics/rig_puppy/tree/5b5adc5a6343a73d9912ac9c3697c04c2f1c49f3)
bruker eldre lulu-esp32s3-mapper. Konverteringsskriptet sender GIF til
https://esp32-gif.espressif.com/api/v1/convert; nettjenesten ble ikke brukt,
og ingen brukerressurser ble lastet opp. Eldre EAF-dokumentasjon omtaler også
LVGL/GIF og lip-sync; dette er ikke brukt som bevis for dagens Puppy-runtime.

[RIG-Muse](https://github.com/LuwuDynamics/RIG-Muse/tree/8ffe1054d09b2234dfbd51c955e9bdb01e62e5a9)
er eksperimentell, bruker Muse Gadget SDK og annen rig_pface/rig_show-arkitektur.
Der finnes faktisk en [PC UI-simulator](https://github.com/LuwuDynamics/RIG-Muse/blob/8ffe1054d09b2234dfbd51c955e9bdb01e62e5a9/esp32/simulator/README.md).
Dokumentasjonen beskriver en 412×412 SenseCAP Watcher-visning med SDL og
hostadaptere, uten emulering av CPU, lyd, kamera, radio eller minnepress.
Dette er relevant referanse, ikke en bekreftet rig_omni/Puppy EAF-emulator.
Ingen kode fra Muse er kopiert inn i Alive.

Offisielle sider som ble hentet:
- https://wiki.xgorobot.com/kb/the-rig-series-robot/rig_en_puppy_overview
- https://wiki.xgorobot.com/kb/pet-puppy/dd8c2c95a4664d56bd48e361e7bbefdf
- https://wiki.xgorobot.com/kb/pet-puppy/cmsra8xe8008hjilg21lcgdzt

Utviklersiden omtaler ESP32-S3 WROOM N16R8. Agentportalen er for konfigurasjon;
den beviser ikke at vår PC-viewer kan snakke med offisiell backend uten aktivering.

## Faktisk ressursvei

puppy_board initialiserer SPI3 og GC9A01, så EmoteDisplay. Build velger boardets
240_240/emote.json og layout.json, regionens Wi-Fi-fil, font og WakeNet. Packer
produserer index.json og assets.bin/board_assets.bin i assets-partisjonen.
Manifestnavn kobles til fil; dette er ikke C-arrays av hver ansiktsramme.
`emote_get_emoji_data_by_name` feiler ved ukjent navn; den undersøkte SetEmotion-
grensen etablerer ingen eksplisitt fallback til neutral.

Systemposter beholdt: icon_speaker_zzz, listen, nvs_reset, wificonfig,
calibration, scanning, launch og remote_mode. Wi-Fi har domestic/international
filvarianter under én logisk manifestpost. Antall fysiske filer og GIF-er er
ikke antall logiske systemtilstander.

## EAF

Container: magic 89 45 41 46, u32LE antall tabellposter, additiv u32LE checksum
og bodylengde. Tabellposter er u32LE størrelse/offset; offsets er fra etter
hele tabellen. ZZ-markør før hver payload. Bildesignatur _S\0, seks versjonsbyte,
8-bit palette, u16LE bredde/høyde/blokkantall/blokkhøyde. Åtte blokker på inntil
32 rader for 240×240. 256 BGRA-farger. RLE-blokk har encoding 0 og count/index-
par. Lengder og offset valideres. Originalenes _C er kontroll, ikke bildedata.
FPS og loop finnes i manifestet, ikke GIF-timing eller EAF-containeren.
Kun 8-bit RLE eksporteres; ingen påstand om å støtte alle mulige EAF-codecs.

`tools/gif_to_eaf.py` gjenbrukes. Identiske payloads deler offset; hver tick
beholdes. Den faktiske C-dekoderen peker hver tabellpost til dens angitte offset,
så det er ingen antakelse om monotont stigende eller unike offsets.

Valideringskjeden: kilderamme → lokal encoder → eksisterende strenge JS-dekoder
→ eksakt RGBA-hash; i tillegg kompileres installert uendret gfx_eaf_dec.c på
host med stubber kun for ESP logging/error/config og kjøres mot de samme
rammene. Dette tester reell C-dekoding, ikke LCD/SPI/FreeRTOS. Optional JPEG og
heatshrink er deaktivert i hosttesten fordi Alive bare bruker RLE.
Se validation-c-decoder.json for kildehash og faktisk testutfall.

## Lisenser

V2-rigg og eksisterende verktøy er lokale prosjektressurser og gjenbrukes.
Ingen ekstern robotøynepakke eller proprietær grafikk er kopiert. Espressifs
EAF-komponent har Apache-2.0-markører; C-dekodertesten kompilerer installert
komponent uendret. Ingen komponentkilde relisensieres. RIG-Muse har egen LICENSE;
den er undersøkt, men ingen Muse-kode er tatt inn. LICENSE i rot ble ikke funnet
på de kontrollerte rig_omni/rig_puppy-URL-ene. Dette er ikke bevis for at grafikk
er fritt lisensiert. Originalenes eierskap og rettigheter beholdes; kommersiell
videredistribusjon av originalgrafikk må avklares med rettighetshaver.
