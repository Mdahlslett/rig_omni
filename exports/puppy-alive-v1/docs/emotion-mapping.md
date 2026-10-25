# LLM og følelsesnavn

Bekreftet i rig_omni/application.cc: innkommende JSON med type `llm` og streng
`emotion` køer `display->SetEmotion(emotion)`. EmoteDisplay sender navnet videre
til `emote_set_anim_emoji`; navneoppslag velger fil, FPS og loop fra assets-index.

Dokumentert av kildekode:

```json
{"type":"llm","emotion":"happy"}
```

Ingen behandling av intensity eller duration_ms finnes i denne grenen. Serveren
velger hvilke navn den faktisk sender; et lokalt ekstra navn gjør ikke at den
begynner å sende navnet. Firmwaregrenen har ingen hardkodet whitelist, så et
nytt navn kan slås opp hvis det pakkes og serveren faktisk sender det. Det er
ikke bevis for at leverandørens server støtter egen navneliste.

Official main ved c2b42cd har 20 ordinære følelser: neutral, happy, sad, angry,
surprised, thinking, winking, cool, laughing, loving, crying, confused,
embarrassed, sleepy, confident, delicious, kiss, relaxed, shocked og silly.
**proud er en tidligere lokal utvidelse**, og bevares som det 21. lokale navnet.
Vieweren merker pakken Original som lokal originalbaseline, som inkluderer proud.

Åtte nye er holdt utenfor standardmanifestet:

| Nytt navn | Kompatibelt offisielt navn |
|---|---|
| curious | thinking |
| skeptical | confused |
| attentive | neutral |
| uncertain | confused |
| excited | happy |
| worried | sad |
| mischievous | silly |
| grateful | loving |

Ved bruk med helt offisiell baseline kan proud mappes til confident.
Mappingfilen er et eksplisitt forslag til integrasjonsadapter, ikke en endring
av serveren eller en påstand om serverens validering. Vieweren bruker aldri
mappingen til å vise feil originalnavn.

Playground kan også simulere:

```json
{"emotion":"curious","intensity":0.65,"duration_ms":3000}
```

Dette er et **lokalt fremtidig grensesnitt**, ikke Luwus wire-protokoll.
Simulatoren validerer navn, intensitet 0–1 og varighet 100–60000 ms.

Dagens firmware: idle/connecting setter neutral, listening setter listen,
STT mens listening setter thinking. TTS-start endrer samtalestatus, men sender
ikke automatisk ny følelsesanimasjon i den grenen. Tilstandsendringer kan
overskrive et LLM-uttrykk. Oppstart, Wi-Fi, NVS-reset, BLE-remote og kalibrering
har egne SetEmotion-kall. Prioriteringsmodellen i Alive er foreslått/hosttestet,
ikke en beskrivelse av en eksisterende firmware-arbiter.
