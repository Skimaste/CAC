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

Data er givet ved tre forskellige CSV filer, som alle har patient_id som primær index. 

Da data ikke er optimal i forhold til at lave og træne modeller på, valgte vi at transformere vores data til en samlet dataframe med en linje for hver patient. Her har vi også valgt at gøre det frit hvor mange læsioner, startende fra den største, man vil have med for hver patient. 

For hver patient er der først inkluderet syv datapunkter, som er afhængige af patienten og ikke den specifikke læsion:

* patient_id: identificerer den specifikke patient (ikke CPR-nr)
* total_score: Agatston score
* label: 1 hvis total_score ≥ 100 og 0 hvis total_score < 100
* total_hhua_pixels: antal hhua (high Hounsfield unit area) pixels over 130
* cts_with_cac: slices fra scanningen hvor der er læsioner
* slice_thickness: fysiske distance mellem hvert slice i mm (vi fjerner alle data punkter hvor scanning har en slice_thickness = 5)
* pixel_spacing_1: fysiske sidelængde på hver pixel i mm

(pixels er kvadratiske, derfor er kun en sidelængde nødvendig)

Herefter tilføjes der, for hver læsion valgt med, 11 datapunkter til linjen for hver patient:

* hhua_i_area: Arealet af den i’te største læsion hos en patient udregnet ved antal pixels i den givne læsion ganget med kvadratet af pixel_spacing_1
* hhua_i_dist_x1: afstanden (i mm) fra centrum af den givne læsion til venstre kant af kassen, vi har afsat omkring hjertet
* hhua_i_dist_x2: afstanden (i mm) fra centrum af den givne læsion til højre kant af kassen, vi har afsat omkring hjertet
* hhua_i_dist_y1: afstanden (i mm) fra centrum af den givne læsion til bunden af kassen, vi har afsat omkring hjertet
* hhua_i_dist_y2: afstanden (i mm) fra centeret af den givne læsion til toppen af kassen, vi har afsat omkring hjertet
* hhua_i_dist_z1: afstanden (i mm) fra centrum af den givne læsion til toppen af hjertet. Dette gøres ved (max(ct_numbers_in_heart) - hhua_i_ct_number) · slice_thickness
* hhua_i_dist_z2: afstanden (i mm) fra centrum af den givne læsion til bunden af hjertet. Dette gøres ved (min(ct_numbers_in_heart) - hhua_i_ct_number) · slice_thickness
* hhua_i_ct_number: på hvilket slice denne læsion er
* hhua_i_max_pixel_value: højeste pixel-værdi (hounsfield unit) for denne læsion
* hhua_i_mean_pixel_value: gennemsnitlige pixel-værdi (hounsfield unit) for denne læsion
* hhua_i_min_pixel_value: laveste pixel-værdi (hounsfield unit) for denne læsion

Alle disse er vist på billedet nedenfor:

![Data_opsætning](https://github.com/Skimaste/CAC/assets/132779543/05dfbca1-6bf7-4e0b-a31b-746ef72a57a2)

### PCA
Vores eksplorative data analyse består i en PCA på vores dataframe med information fra kun én læsion per patient for lettere fortolkning. Den første læsion er også den vigtiste.
Vi får følgende scree plot:

![Explained_variance](https://github.com/Skimaste/CAC/assets/132779543/8c3c62bc-59b1-4fb5-aedd-ad5bb1cf28dd)

Vi ser heraf, at vi kan forklare ca. 90% af variansen med kun 8 komponenter/features, hvilket tyder på at nogle af vores features er mindre vigtige.

![PCA_Corelation-matrix](https://github.com/Skimaste/CAC/assets/132779543/009fb0bd-80f5-42da-8e51-f63647bd1f65)

Af korrelation plottet ser vi en stærk korrelation mellem det totale antal pixels og antallet af ct slices med potentielle læsioner hvilket giver god mening. Derudover ses det at afstanden fra centeret af læsionen til hjertekonturen er parvis stærkt negativ korreleret hvilket også er forventet (hvis læsionen er tæt på den ene kant, må den være langt fra den modsatte). Til sidst ses også en stærk korellation mellem min, max og mean pixel value. 

Til sidst ses har vi lavet et biplot: 

![PCA_bi-plot](https://github.com/Skimaste/CAC/assets/132779543/1713f668-ee3b-4d68-a24f-b42cb2298706)

Heraf ses det at PCA ikke formår at lave en klar opdeling af de to klasser. Disse resultater fortæller os at vi ikke vil få meget gavn af dimensionsreducering, da de fleste af vores features er nødvendige til at forklare variansen, og PCA vil derfor potentielt set sænke vores models nøjagtighed. Vi vælger derfor ikke at bruge PCA i vores logistiske regressionsmodel.

### Logistisk regression

Vi vælger først at lave et dataframe med information fra alle læsioner. Det højeste antal læsioner i en patient er 158, så vi bruger zero padding på patienter med færre læsioner end dette. Ved inklusion af 158 læsioner opnår vi 1742 features.

Vi laver først et plot af standardafvigelsen for hver feature:

![Standard_deviation_feautures](https://github.com/Skimaste/CAC/assets/132779543/4014ca18-ee11-420d-8c3f-5a93f5a7012f)

Vi ser heraf at fordelingen er heterogen, så vi vil derfor standardisere vores data med Sklearns standardscaler som bruger formlen $$z = (x - u) / s,$$ hvor u er mean og s er standardafvigelsen. 

Vi ser desuden at fordelingen af de to klasser er skæv:

![Distributaion_of_classes](https://github.com/Skimaste/CAC/assets/132779543/5189f764-28e1-4dbf-9159-789aef425b32)

Dette viser os, at en naiv model, som prædikterer nul hver gang, vil være korrekt ca. 85% af tiden. Dette ville give en lav loss for vores model, så for at undgå at vores model også gætter nul på alle patienter tilføjer vi class weights, som vægter minoritetsklassen mere. Dette gør at vores loss funktion bliver "straffet" ekstra ved falske negative. Vi vælger her at lave balancerede class weights, så vi heller ikke får for mange falske positive. Disse udregnes med formlen weight_for_class_i = total_samples / (num_samples_in_class_i * num_classes).

Ud af de 1742 features forventer vi at kun nogle få af disse faktisk har en indflydelse på vores model, så vi tilføjer lasso regularization (l1 regularization) i vores logistiske regressionsmodel for at tvinge nogle af vores weights til at være nul. For at finde den optimale regularization-styrke bruger vi 10-fold crossvalidation og vælger den værdi som giver den laveste test loss:

![Logistic_regularization_strength](https://github.com/Skimaste/CAC/assets/132779543/a68e8c47-3587-4055-9c7b-4a9944e2fd70)

Med dette in mente opbygger vi vores logistiske regressionsmodel ved at opdele vores data i 80% træningsdata og 20% testdata. Vi bruger den samme random_state igennem vores projekt så vi kan sammenligne vores modeller bedre. 

Efter træning af vores model vil vi finde den cutoff værdi som giver det bedste trade-off mellem accuracy, specificity og sensitivity. Dette har vi gjort ved at teste 25 forskellige cutoff-offsets og plottet resultaterne:

![Cutoff_logistic](https://github.com/Skimaste/CAC/assets/132779543/017c5902-3762-447d-a941-66887b4da67b)

Et cutoff offset på 0.0 svarer til standard cutoff-værdien 0.5, og cutoff værdien udregnes 0.5 - cutoff_offset.
Givet plottet ser vi det bedste trade-off ved offset = -0.083. Dette giver os en cutoff værdi på 0.5 - (-0.083) = 0.583.

Denne cutoff værdi giver os følgende confusion matrix for vores test set:

![Logistic_confusion_matrix](https://github.com/Skimaste/CAC/assets/132779543/92b9f968-15e0-48c4-910e-d20e134858ca)

Efter at have kigget på vores models endelige weights, ser vi at mange weights er 0, og vi kan konkludere at vi kun har brug for information fra 29 lesions for at få nøjagtig de samme resultater:

![Logistic_confusion_matrix_29_lesions](https://github.com/Skimaste/CAC/assets/132779543/05881de9-8384-47f7-bd40-724bb2ab28c2)

Vi ser heraf at vores model med 323 features klarer sig lige så godt som modellen med 1742. Vi kan altså reducere antallet af features med mere end 80% uden at det går udover nøjagtigheden. Dette giver god mening da læsionerne er sorteret efter størst betydning først, og efter et bestemt antal læsioner er modellen i stand til at vide om der er åreforkalkning eller ej. Samtidig er der heller ikke så mange patienter med flere end 29 læsioner.

I en logistisk regressionsmodel som denne, som er hurtig at træne og teste kan vi sagtens bruge alle 1742 features uden problemer, men når vi skal opbygge vores neurale netværk er dette en brugbar information, da neurale netværk er meget langsommere at træne og har let ved at overfitte ved høj-dimensionel data. 

### Neuralt Netværk

Givet resultaterne fra vores afsnit med logistisk regression vælger vi at bruge features fra kun de 29 vigtigste læsioner i hver patient. Derudover så vi at lasso regularization havde en gavnlig effekt på vores gennemsnitlige loss, så vi tilføjer også lasso regularization til vores neurale netværk med default styrken 0.01. 

Ved opbygning af netværket starter vi først med et enkelt skjult lag med 16 neuroner og får følgende kurver:

![1hidden_loss](https://github.com/Skimaste/CAC/assets/132779543/3da2a93c-d1ad-42c2-bea4-e826f7f5ad2c)
![1hidden_acc](https://github.com/Skimaste/CAC/assets/132779543/4713c28b-f1e8-4c0c-9b90-8f6d0ec5f6a7)

Dette giver umiddelbart gode resultater, men det ses også at modellen overfitter. 
Vi tilføjer et skjult lag mere:

![2hidden_loss](https://github.com/Skimaste/CAC/assets/132779543/9adca6cb-28e4-4bd3-9cab-7efeabae5185)
![2hidden_acc](https://github.com/Skimaste/CAC/assets/132779543/75dc741e-19fc-42ad-9602-f4a022295860)

Vi ser at denne model opnår en højere test accuracy, men den overfitter stadig. 
For at undgå overfitting tilføjer vi dropout med en sandsynlighed på 20% for at en neuron er slukket i træningsfasen.
Dette giver os følgende kurver:

![2hidden_dropout_loss](https://github.com/Skimaste/CAC/assets/132779543/4662a1d8-ada1-4837-8cad-3ddf067de714)
![2hidden_dropout_acc](https://github.com/Skimaste/CAC/assets/132779543/ebfa0d78-780a-42c6-a290-9badcbf7adbe)

Vi ser at modellen ikke længere overfitter, så vi holder os til to skjulte lag og udforsker nu de andre hyperparametre.

Vi laver, på samme vis som i vores afsnit med logistisk regression, 5-fold crossvalidation for at teste forskellige styrker af lasso regularization:

![neural_network_L1_cv-1](https://github.com/Skimaste/CAC/assets/132779543/d93186dd-ff7a-4cef-afc1-868190a0d364)

Vi ser heraf at standardstyken på 0.01 giver et meget godt tradeoff mellem lav test error og minimal overfitting, så vi vælger at beholde vores lasso regularization styrke på 0.01.

Vi har derudover fundet, at det optimale antal neuroner i de skjulte lag er henholdsvis 32 og 16. Til slut har vi valgt at sænke learning rate til 0.0002 og øget antal af epochs til 1000. Med disse optimeringer får vi følgende kurver for vores færdige model:

![final_loss](https://github.com/Skimaste/CAC/assets/132779543/d156e563-18e3-454c-b15f-116b16dc7ff2)









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
