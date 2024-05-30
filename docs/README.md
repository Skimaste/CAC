# CAC: Koronararterieforkalkning

## Projektbeskrivelse

Dette repository indeholder koden og de tilhørende modeller til projektet "Koronararterieforkalning" lavet af Frederik Nordberg Jensen, Emil Juul Elberg og Jonas Sadeghi Knudsen.

### Overblik

Projektet fokuserer på at lave opportunistisk screening ved at analysere CT-scanningsdata af kvinder, der gennemgår behandling for brystkræft. Målet er, ved brug af machine learning, at vurdere graden af koronararterieforkalkning (åreforkalkning i hjertet) ud fra disse billeddata og tilnærme patienternes Agatston-score. Agatston-scoren er en måleenhed inden for kardiologi, der angiver den samlede volumen og densitet af kalkaflejringer i hjertets arterier. En højere Agatston-score indikerer en større risiko for hjerteproblemer.

Udover den medfødte risiko kan koronararterieforkalkning give udfordringer under strålebehandling for brystkræft. Hvis de ydre arterier i hjertet er forkalkede, øges risikoen for hjertesygdomme betydeligt på grund af udsættelse for stråling. Derfor kan tilpasning af strålebehandlingen være gavnlig i visse tilfælde.

Dog er evaluering af koronar forkalkning ikke en standardpraksis i denne sammenhæng på grund af dens opfattede irrelevans og den ressourcekrævende manuelle proces. Projektet sigter mod at automatisere denne evaluering for at give yderligere information, som klinikere kan overveje under behandlingsplanlægning, samtidig med at patienterne informeres om deres hjertetilstand uden behov for en separat hjerte-CT-scanning.

### Videnskabelig problemformulering

Det primære mål er at udvikle en model, der kan forudsige brystkræftpatienters Agatston-score og kategorisere dem i to risikokategorier baseret på scoren. Her har vi først forsøgt os med en logistisk model og efterfølgende et neuralt netværk til at forudsige hvilken af de to kategorier de hører til.

### Beskrivelse af data

Datasættet består af automatisk udtrukne data fra cirka 1300 CT-scanninger af brystkræftpatienter. Disse scanninger består af hundredevis af 2D tværsnit eller "slices". En model har automatisk behandlet disse slices for hver patient, segmenteret hjertet og identificeret pixels med en Houndsfield-enhed (HU) større end 130. HU er en måleenhed for radiodensitet i en CT-scanning, hvor pixels med lav HU svarer til blødt væv og muskelmasse, mens calcium og knoglemasse har en HU større end 130. Derudover er der manuelt afledte features, herunder Agatston-scoren, tildelt af en vejledende kliniker baseret på en grundig undersøgelse af CT-billederne.

### Udfordringer med data

Udfordringer opstår, fordi CT-scanningerne ikke er blevet taget specifikt til dette formål, hvilket fører til varierende klarhed af hjertet sammenlignet med standard hjerte-CT-scanninger. Problemer såsom bevægelsesartefakter og forvrængninger fra medicinske implantater (f.eks. pacemaker) kan påvirke nøjagtigheden af den automatiske funktionsekstraktionsproces og potentielt føre til falske positive for koronar forkalkning.

## Resultater 

Gennem vores projekt

### Data pipeline

Vi merger vores datasæt på patient id, for at få total score og hvilke slices der er i hjertet for hver patient.

vi kigger på listen af pixels for hver læsion og finder min, max og mean af pixelværdierne (hounsfield units)



### PCA
Vores eksplorative data analyse består i en PCA på vores dataframe med information fra kun én læsion per patient for lettere fortolkning. Den første læsion er også den vigtiste.
Vi får følgende scree plot:

![Explained_variance](https://github.com/Skimaste/CAC/assets/132779543/8c3c62bc-59b1-4fb5-aedd-ad5bb1cf28dd)

Vi ser heraf, at vi kan forklare ca. 90% af variansen med kun 8 komponenter/features, hvilket tyder på at nogle af vores features ikke er nødvendige.

![PCA_Corelation-matrix](https://github.com/Skimaste/CAC/assets/132779543/009fb0bd-80f5-42da-8e51-f63647bd1f65)

Af korrelation plottet ser vi en stærk korrelation mellem det totale antal pixels og antallet af ct slices med potentielle læsioner hvilket giver god mening. Derudover ses det at afstanden fra centeret af læsionen til hjertekonturen er parvis stærkt negativ korreleret hvilket også er forventet (hvis læsionen er tæt på den ene kant, må den være langt fra den modsatte). Til sidst ses også en stærk korellation mellem min, max og mean pixel value. 

Til sidst ses har vi lavet et biplot: 






### Logistisk regression

### Neuralt Netværk

### Konklusion

## Repositorystruktur

Repositoryet er struktureret som følger:

- `code/`: Indeholder koden til dataforbehandling, modeltræning og evaluering.
- `docs/`: Indeholder projektets dokumentation, herunder denne README-fil.
- `images/`: Gemmer billeder af modelvurderinger og visualiseringer.
- `models/`: Gemmer trænede modeller til fremtidig brug.

## Kom godt i gang

For at komme i gang med projektet skal du følge disse trin:

1. Klon repositoryet til din lokale maskine ved hjælp af `git clone`.
2. Udforsk mappen `code/` for at forstå projektets struktur og funktionaliteter.
3. Se dokumentationen i mappen `docs/` for detaljerede oplysninger om data, metode og resultater.

## Bidragydere

- Frederik Nordberg Jensen
- Emil Juul Elberg
- Jonas Sadeghi Knudsen

## Licens

Dette projekt er licenseret under [MIT-licensen](LICENSE).

---

Ved eventuelle spørgsmål eller problemer bedes du kontakte projektets vedligeholdere. Vi byder bidrag og feedback velkommen for at forbedre projektet yderligere. Tak for din interesse og støtte!
