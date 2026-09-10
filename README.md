# AI-samtalsstöd

Ett AI-verktyg som lyssnar på inspelade telefonsamtal, skriver ner vad som sägs, och skapar en färdig, strukturerad journalanteckning automatiskt – på under 30 sekunder.

Syftet är att spara tid för handläggare som idag skriver anteckningar för hand efter varje samtal.

---

## Demo



[![Se demo-video](https://img.youtube.com/vi/v4F_oepzoYU/maxresdefault.jpg)](https://youtu.be/v4F_oepzoYU)



---

## Vad det gör

1. **Tar in ett telefonsamtal** (en ljudinspelning).
2. **Transkriberar** – AI:n lyssnar och skriver ner allt som sägs, ord för ord, på svenska.
3. **Sammanfattar** – texten görs om till en strukturerad journalanteckning i myndighetsstil, skriven i löpande text precis som en riktig journalanteckning (och tar upp bland annat aktuell situation, vad den sökande efterfrågar, uppgifter som framkommit, bedömning och nästa steg).
4. **Sparar** resultatet automatiskt som ett dokument.

Hela kedjan tar ungefär 20 sekunder på en dator med grafikkort.

---

## Så här fungerar det (tekniskt)

- **Transkribering:** bygger på Whisper (modellen `large-v3`), som är mycket bra på svenska – även på tal med brytning och dialekt. Körs lokalt.
- **Sammanfattning:** en språkmodell (Claude) omvandlar transkriptet till en journalanteckning enligt en noggrant utformad instruktion. Modulen är utbytbar – den kan lika gärna köras mot en lokal modell så att ingen data lämnar datorn.
- **Gränssnitt:** en enkel webbsida där man laddar upp en ljudfil och får tillbaka transkribering och journalanteckning.
- **Byggt i** Python (Flask för webbdelen).

En viktig princip: **AI:n skriver bara det som faktiskt sägs i samtalet – den hittar inte på.** Om inget beslut fattas i samtalet skriver den att inget beslut fattats. En handläggare granskar alltid resultatet innan det används.

---

## Om hastighet

Bearbetningen tar ca 20 sekunder i demon, som körs på en vanlig dator med grafikkort. På kraftfullare serverhårdvara går det ännu snabbare. Hastigheten beror på hårdvaran, inte på verktyget i sig.

---

## Från demo till skarp version

I den här demon laddas ljudfilen upp manuellt, för att visa själva tekniken. I en färdig, driftsatt version skulle verktyget kopplas till telefonisystemet, så att samtalen hämtas automatiskt – handläggaren behöver då inte ladda upp något själv.

En sådan skarp version innebär hantering av känsliga personuppgifter och skulle därför köras i en säker, godkänd miljö med rätt dataskyddsåtgärder på plats.

---

## Status

Detta är en fungerande prototyp (proof of concept) byggd för att visa att idén fungerar. Den använder påhittade testsamtal – aldrig riktiga personuppgifter.

---

## Teknisk översikt (för utvecklare)

**Pipeline:** ljudfil → transkribering → sammanfattning → sparad journalanteckning.

**Komponenter:**
- **Transkribering:** faster-whisper med modellen `large-v3`. Modellen laddas in en gång vid serverstart och återanvänds, vilket håller varje anrop snabbt. Kör på GPU (CUDA) med automatisk fallback till CPU.
- **Sammanfattning:** anrop mot en språkmodell via API, med en systemprompt som styr struktur och förhindrar att modellen hittar på information. Sammanfattningssteget är byggt som en utbytbar modul, så motorn kan bytas (moln-API eller lokal modell) genom att ändra en enda inställning.
- **Webbserver:** Flask med waitress som produktionsservern.
- **Utdata:** varje körning sparas till en tidsstämplad textfil.

**Designval:** miljöberoende inställningar (som API-nycklar) läses från miljövariabler, inte hårdkodas – så samma kod kan köras lokalt eller på en server utan ändringar.

**Språk & ramverk:** Python, Flask, faster-whisper.
