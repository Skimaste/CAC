# CAC: Koronararterieforkalkning

## Projektbeskrivelse

Dette repository indeholder koden og dokumentationen til projektet med titlen "Koronararterieforkalkning," udviklet af Frederik Nordberg Jensen, Emil Juul Elberg og Jonas Sadeghi Knudsen i april 2024.

### Overblik

Projektet fokuserer på at analysere CT-scanningsdata af kvinder, der gennemgår behandling for brystkræft. Formålet er specifikt at vurdere tilstanden af koronararterieforkalkning (CAC) ud fra disse scanninger og tilnærme patienternes Agatston-score. Agatston-scoren er en måleenhed inden for kardiologi, der angiver den samlede volumen og densitet af calciumaflejringer i hjertets arterier. En højere Agatston-score indikerer en større risiko for hjerteproblemer.

Udover den medfødte risiko kan koronararterieforkalkning give udfordringer under strålebehandling for brystkræft. Hvis de ydre arterier i hjertet er forkalkede, øges risikoen for hjertesygdomme betydeligt på grund af udsættelse for stråling. Derfor kan tilpasning af strålebehandlingen være gavnlig i visse tilfælde.

Dog er evaluering af koronar forkalkning ikke en standardpraksis i denne sammenhæng på grund af dens opfattede irrelevans og den ressourcekrævende manuelle proces. Projektet sigter mod at automatisere denne evaluering for at give yderligere information, som klinikere kan overveje under behandlingsplanlægning, samtidig med at patienterne informeres om deres hjertetilstand uden behov for en separat hjerte-CT-scanning.

### Videnskabelig problemformulering

Det primære mål er at udvikle en model, der kan forudsige brystkræftpatienters Agatston-score nøjagtigt og kategorisere dem i en af fem risikokategorier baseret på scoren. Lineære modeller har vist sig utilstrækkelige til denne opgave, da forholdet mellem Agatston-scoren og data er ikke-lineært. For at imødekomme dette trænes et neuralt netværk til at forudsige Agatston-scoren, hvilket udnytter dets ikke-lineære egenskaber og potentiale for finjustering baseret på brugen.

### Beskrivelse af data

Datasættet består af automatisk udtrukne oplysninger fra cirka 1300 CT-scanninger af brystkræftpatienter. Disse scanninger består af hundreder af 2D tværsnit eller "skiver". En maskine har automatisk behandlet disse skiver for hver patient, segmenteret hjertet og identificeret pixels med en Houndsfield-enhed (HU) større end 130. HU er en måleenhed for radiodensitet i en CT-scanning, hvor pixels med lav HU svarer til blødt væv og muskelmasse, mens calcium og knoglemasse har en HU større end 130. Derudover er der manuelt afledte funktioner, herunder Agatston-scoren, tildelt af en vejledende kliniker baseret på en grundig undersøgelse af CT-billederne.

### Udfordringer med data

Udfordringer opstår, fordi CT-scanningerne ikke blev taget specifikt til dette formål, hvilket fører til varierende klarhed af hjertebillederne sammenlignet med standard hjerte-CT-scanninger. Problemer såsom bevægelsesartefakter og forvrængninger fra medicinske implantater kan påvirke nøjagtigheden af den automatiske funktionsekstraktionsproces og potentielt føre til falske positive for koronar forkalkning.

### Projektmål

Det primære mål er at give en pålidelig indikation af tilstedeværelsen af koronararterieforkalkning. Denne information kan hjælpe klinikere med at beslutte, om yderligere undersøgelse af en patients CT-scanning er berettiget. Derudover er et sekundært mål at træne et regressionsneuralt netværk til at forudsige en mere præcis Agatston-score. Dette sigter mod at give klinikere nøjagtige og håndterbare data, så de kan give tidlige advarsler til patienterne, inden potentielle hjertesygdomme udvikler sig. Projektets kode og dokumentation vil blive gjort tilgængelig på et GitHub-repository for bredere adgang.

## Repositorystruktur

Repositoryet er struktureret som følger:

- `code/`: Indeholder koden til dataforbehandling, modeltræning og evaluering.
- `data/`: Indeholder datasættet, der bruges til træning og test af modellerne.
- `docs/`: Indeholder projektets dokumentation, herunder denne README-fil.
- `results/`: Gemmer resultaterne af modelvurderinger og eventuelle visualiseringer.
- `models/`: Gemmer trænede modeller til fremtidig brug.

## Kom godt i gang

For at komme i gang med projektet skal du følge disse trin:

1. Klon repositoryet til din lokale maskine ved hjælp af `git clone`.
2. Installer de nødvendige afhængigheder angivet i `requirements.txt`.
3. Udforsk mappen `code/` for at forstå projektets struktur og funktionaliteter.
4. Henvis til dokumentationen i mappen `docs/` for detaljerede oplysninger om data, metode og resultater.

## Bidragydere

- Frederik Nordberg Jensen
- Emil Juul Elberg
- Jonas Sadeghi Knudsen

## Licens

Dette projekt er licenseret under [MIT-licensen](LICENSE).

---

Ved eventuelle spørgsmål eller problemer bedes du kontakte projektets vedligeholdere. Vi byder bidrag og feedback velkommen for at forbedre projektet yderligere. Tak for din interesse og støtte!
