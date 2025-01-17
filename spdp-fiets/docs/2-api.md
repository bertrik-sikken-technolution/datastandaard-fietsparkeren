## API

### Algemeen - wrapping

Alle responses van GET-requests met meer dan één resultaat zijn gestoken in een wrapper-object `result`. 
Implementaties kunnen zo metadata over de resultaten, zoals paginering, gebruikte filters etc. meegeven.

Responses van GET-requests die expliciet om één object vragen, wrappen dit resultaat niet.

<aside class='example' title="Gewrapped resultaat meervoudige bevraging">

```
`GET /static`
{
  "result": [
    { "id": ..., ... },
    { "id": ..., ... }
  ]
}
```

</aside>
<aside class='example' title="Niet-gewrapped resultaat enkelvoudige bevraging">

```
`GET /organiations/defietsentellers`
{
  "id": "defietsentellers,
  "name: "De Fietsentellers BV"
}
```

</aside>

### API 3 — data opslaan

API 3 is de ontvangende zijde van de dataportal API.
Straattellers of stallingssoftware kunnen met POST-requests hun data opsturen naar het dataportal.
In de body's van deze POST-requests dienen ze gebruik te maken van in [](#dataschema-s) beschreven objecten.

De [Survey](#surveys), [statische data](#statische-data) en [dynamische data](#dynamische-data) MOETEN afzonderelijk worden ingediend middels een POST-request.

Alle genoemde requests beginnen met een /. Voor deze schuine streep komt uiteraard de base-url van de API. 

<aside class='note'>

In het geval van de pilot in VeiligStallen is de base-url `https://remote.veiligstallen2.nl/rest/api`.

</aside>

Een organisatie opslaan:  
`POST /organisations` [Body zonder surveyId](./examples/API3/requests/POST_new_organisation.json)  
[Response](./examples/API3/responses/POST_new_organisation.json)  

Een survey opslaan:  
`POST /surveys` [Body met surveyId](./examples/API3/requests/POST_new_survey_with_id.json)  
Een id van de survey mag door exploitant zelf gekozen worden. Suggestie gebruik [CBS codes](https://www.cbs.nl/nl-nl/onze-diensten/methoden/classificaties/overig/gemeentelijke-indelingen-per-jaar/indeling-per-jaar/gemeentelijke-indeling-op-1-januari-2020) en unieke gegevens uit het onderzoek, bijv. < CBS_nr_gemeente >_< jaartal > => 0202_2020.  

<pre class='example json' title='Body zonder surveyId' data-include='../examples/API3/requests/POST_new_survey_without_id.json' data-include-format='text'></pre>

De instantie die de survey instuurt, wordt 'eigenaar' van dit onderzoek. 

Een Survey met een `surveyId` dat al bestaat, zal resulteren in een 409-error (Conflict). Behalve als deze POST wordt gedaan door de eigenaar van het onderzoek. In dat geval wordt de bestaande survey overschreven.

Indien geen `surveyId` is gegeven, maakt de server zelf een id en geeft deze terug in de response.  

[Response](./examples/API3/responses/POST_new_survey.json)  

<pre class='example json' title="Response" data-include='../examples/API3/responses/POST_new_survey.json' data-include-format='text'></pre>

`POST /static` [Body](./examples/API3/requests/POST_static_section.json)  
Statische secties worden per stuk gepost. De body bevat dus slechts één sectie. Dit om eventuele schrijffouten te kunnen retourneren in de response, bijvoorbeeld als de `id` van de sectie reeds in gebruik is.  

<pre class='example json' title="Body" data-include='../examples/API3/requests/POST_static_section.json' data-include-format='text'></pre>  
[Response](./examples/API4/POST_static_section.json)  - In geval van een succesvolle opslag, bevat de response het opgeslagen object in property `section`.  

Het `id` van een statische sectie is, net als het `surveyID`, door de opsturende instantie zelf samen te stellen. Suggestie: prefix het sectionId met het `surveyId` om een unieke id te garanderen, bijvoorbeeld: `< surveyId >_< straatnaam >`

Als er al een sectie met hetzelfde `id` bestaat, krijgt de response code 409 (Conflict). Tenzij de POST wordt gedaan door de owner van de sectie. In dat geval wordt de bestaande sectie overschreven.

Het is niet verplicht veld `id` mee te geven in de request. Als er geen `id` gegeven is, wordt deze door de API gegenereerd en is deze zichtbaar in de respons, dat immers het volledig opgeslagen object bevat.

Statische secties worden per stuk gepost. De body bevat dus slechts één sectie. Dit om eventuele schrijffouten te kunnen retourneren in de response, bijvoorbeeld als de `id` van de sectie reeds in gebruik is.  
  
Updaten van een statische sectie kan met PUT. Hier hoeven alleen de verschillen meegegeven te worden, ongewijzigde velden dus niet.  
`PUT /static/:staticsectionid` [Body](./examples/API3/requests/PUT_static_section.json)  

`POST /dynamic` [Body](./examples/API3/requests/POST_dynamic_data.json)  
Dynamische data, dus de daadwerkelijke metingen, worden in bulk opgeslagen. De body bevat dus een array aan metingen.  

<pre class='example json' title="Body" data-include='../examples/API3/requests/POST_dynamic_data.json' data-include-format='text'></pre>

Door het veld surveyId mee te geven aan de meting, worden deze gekoppeld aan een onderzoek.  
Zonder het veld surveyId worden de data niet gekoppeld aan een onderzoek en zullen ze dus ook niet gevonden worden bij een zoekopdracht naar een surveyId!

### API 4 - data opvragen

Op dezelfde manier het insturen van data kan de data ook weer worden opgevraad. De POST-requests veranderen in GET-requests.

<aside class='example' title='GET-requests om data op te vragen'>

Je kunt dus zowel dynamische als statische data afzonderlijk koppelen aan een onderzoek.

Dynamische data kan sterk gecomprimeerd worden door alleen de totalen op te slaan. Dit zal in de praktijk vaak voorkomen.

`POST /dynamicdata`

<aside class='example json' title="Body met alleen totalen" data-include='../examples/API3/requests/POST_compressed_dynamic_sections.json' data-include-format='text'></aside>

```
GET /surveys?query_params
GET /static?query_params
GET /dynamic?query_params
```

</aside>

De `query_params` dienen als zoekopdrachten, zodat heel specifiek naar data gezocht kan worden. Dat kan op diverse manieren. De datastandaard stelt een minimaal aantal zoekopties verplicht:

__Filter op diepte__  
Gedetailleerde informatie over de bezetting geeft behoorlijk grote bomen, die niet voor iedere analist van deze data even interessant is. Als deze analist de waarnemingen van een stationsstalling door de tijd opvraagt, kunnen de responsen enorm groot worden. Als 90% van de data voor de analist niet relevant is, is dit natuurlijk onzinnig.
Daarom stelt de datastandaard een beperkt aantal zoekfunctionaliteiten verplicht. De belangrijkste van deze is dat je de mate van detail, de diepte (depth), bij een zoekopdracht kunt  aangeven aan de API. 
Stel dat in het dataportal de meest gedetailleerde boom is opgeslagen:  
 
Als een analist alleen geïnteresseerd is in de bezetting van de totale stalling, vraagt hij data op op diepte 1. De API geeft hem dan dit resultaat:  
 
Als een stalling iedere 2 minuten data opslaat en een analist vraagt de data op van een hele maand, scheelt dat uiteraard enorm in de datastroom en daarmee in de performance van de API.  

__Filter op inhoud__  
Om data-analisten te helpen hun weg te vinden in de gestaag groeiende data van een dataportal, stelt de datastandaard een aantal zoekfunctionaliteiten verplicht. Zo moet een er minimaal gefilterd kunnen worden op:  
* Data binnen een bepaalde tijdspanne (startDate, endDate)
* Data van een bepaalde secties (sectionId)
* Data van een bepaald onderzoek (surveyId)
* Data van een bepaalde opdrachtgever (authorityId)
* Data van een bepaalde dataleverancier (contractorId)
* Data binnen een bepaalde geo-polygoon of geo-punt + radius

Deze filters moeten met elkaar gecombineerd kunnen worden.  
  
De praktijk moet uitwijzen of deze lijst voldoende is om aan alle wensen van de data-analisten te voldoen. Indien nodig zullen er meer zoekfuncties aan deze lijst worden toegevoegd.  

__Filter op tijdspanne__  
Gebruik bij het zoeken de url-parameters startDate en endData. Tijdstippen worden altijd doorgegeven in [[ISO8601]] timestamp formaat.  
Dus:  
`?startDate=2020-01-01T0:00:00&endDate=2020-02-01T0:00:00`  

__Filter op sectieId__   
Als een analist alleen data van bepaalde statische secties:  
`?staticSectionId=:staticSectionId`  
Of meerdere secties:  
`?staticSectionId=staticSectionId1,staticSectionId2,staticSectionId3`  

__Filter op privateId__   
Als een analist alleen data wil van een bepaald fietstype, bijvoorbeeld gewone fietsen:  
`?privateId=:privateId`  
Of meerdere secties:  
`?privateId=privateId1,privateId2,privateId3`  

__Filter op onderzoek__  
Data die ingestuurd wordt door een dataleverancier kan worden voorzien van een onderzoeksID (surveyID). Als deze ID gegeven is, is het uiteraard mogelijk hierop te filteren:  
`?surveyID=:surveyID`  

__Filter op opdrachtgever__  
Hetzelfde idee al filteren op onderzoek: als de data voorzien is van de ID van een opdrachtgever, kan hierop gefilterd worden:  
`?authorityID=:authorityID`  

__Filter op dataleverancier__  
De dataportal kan de data die wordt ingestuurd voorzien van de ID van de leverancier van deze data. Als dat gebeurt, kan er uiteraard gezocht worden op dit ID:  
`?contractorID=:contractorId`  

__Geo zoekopdrachten__
Als de statische data van secties is voorzien van exacte geo-coördinaten, moeten deze secties gevonden kunnen worden aan de van een geo-zoekopdracht.  

__Filter op geo-data: polygoon__  
Geef een polygoon mee in de query, aan de hand waarvan de corresponderende secties worden gezocht.  
Er zijn twee manieren voor deze query:  
* zoek secties die de gegeven polygoon overlappen: relation=intersects. Dit is de default.  
`?geopolygon=4.895168,52.370216,4.895168,53.370216,5.895168,53.370216,4.895168,52.370216&relation=intersects`  
* zoek secties die geheel binnen de gegeven polygoon vallen: relation=within.  
`?geopolygon=4.895168,52.370216,4.895168,53.370216,5.895168,53.370216,4.895168,52.370216&relation=within`  

__Sorteren__  
Dynamische data moet gesorteerd kunnen worden opgevraagd. Dat kan met de parameters `orderBy` en `orderDirection`:  
`dynamicdata?orderBy=timestamp&orderDirection=ASC` - sorteert op timestamp van oudste naar nieuwste  
`dynamicdata?orderBy=count.numberOfVehicles&orderDirection=DESC` - sorteert op aantal voertuigen van hoog naar laag  

#### Een overzicht van alle query parameters
| param             | type        | values                                                 |
| ----------------- |---------- | ----------------------------------------------------- |
| `surveyId`            | string    | Alleen data van dit onderzoek                            |
| `authorityId`   | string    | Alleen data van deze opdrachtgever                     |
| `contarctorId`  | string    | Alleen data van deze dataleverancier                  |
| `depth`             | number    | Aantal te bevragen sectie-lagen vanaf gegeven pad  default = 1                                           |
| `startDate`            | UTC timestamp    | Selectie op timestamp. Section.timestamp >= startDate     |
| `endDate`              | UTC timestamp    | Selectie op timestamp. Section.timestamp <= endDate        |
|                          |                  |                                                                |
| `geoPolygon`        | list met coördinaten | lat1,lng1,lat2,lng2,lat3,lng3,...,...,lat1,lng1    |
| `geoRelation`      | string    | 'intersects' (default) of 'within'    |
|                          |                  |                                                                |
| `orderBy`         | string    | veldnaam waarop gesorteerd wordt    |
| `orderDirection`        | string    | ASC (default) of DESC, in combinatie met orderBy    |
{.data}

Query-params dienen hoofdletterongevoelig te zijn, dus authorityid=abc is hetzelfde als authorityID=abc

### API 5 - realtime data lezen vanuit dataportal voor t.b.v. webapplicaties
Gebruikers van API 5 - de datastroom tussen het dataportal en de webapplicaties - zijn vooral geïnteresseerd in realtime data. Per secties dus slechts één resultaat. 
Het endpoint voor API 5 is `/latest`.

Het gaat in de `/latest`-requests altijd om dynamische data, dus `dynamicData` kan worden weggelaten uit het pad.



