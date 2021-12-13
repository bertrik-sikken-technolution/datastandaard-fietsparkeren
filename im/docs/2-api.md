# REST-API

Het informatiemodel beschreven in <a href='#dataschema-s'></a> bepaalt in algemene zin de objecttypen en eigenschappen van deze datastandaard.
Een implementatie van de datastandaard MOET ook een REST-API volgens deze sectie aanbieden.
Een implementatie MAG ook andere uitwisselingmogelijkheden aanbieden, zolang dat op een manier is die consistent is met de semantiek van <a href='#dataschema-s'></a>.

## Algemene uitgangspunten

Deze REST-API veronderstelt communicatie over HTTP [[rfc7231]] en met gebruik van JSON [[ECMA-404]].

### Kardinaliteit

De aangegeven kardinaliteit geldt voor bij het insturen van gegevens.
Bij het uitleveren van gegevens MAG een implementatie sommige gegevens achterwege laten.

_De rest van deze sectie is niet-normatief._

Een implementatie kan bijv. uit overwegingen op het gebied van privacy of concurrentiegevoeligheid bepaalde gegevens achterwege laten.

### Onbekende eigenschappen of waardes

Een implementatie MAG onbekende waardes of onverwachte datatypes afwijzen.
Een user agent MAG onbekende waardes of onverwachte datatypes afwijzen.

Een implementatie MAG altijd meer eigenschappen versturen naar een client.
Een user agent MOET onbekende eigenschappen accepteren.

Een implementatie MOET meer eigenschappen accepteren van een client.
Een implementatie MAG onbekende eigenschappen afwijzen.

### Resource identifiers

Verschillende objecttypen hebben de eigenschap `id` (namelijk:
{{Organisation.id}},
{{ParkingFacility.id}},
{{Section.id}},
{{Survey.id}},
{{SurveyArea.id}}).
De waarde hiervan wordt ook wel [[=ResourceIdentifier=]] genoemd.
Ook worden koppelingen tussen objecttypen gelegd o.b.v. de waarde van deze eigenschap.

Steeds geldt bij `id` het volgende, wanneer er een nieuwe [[=Resource=]] wordt aangemaakt:

- Een implementatie MAG een door de client gegenereerde [[=ResourceIdentifier=]] accepteren.
- Een implementatie MAG een door de client gegenereerde [[=ResourceIdentifier=]] afwijzen als het niet aan eigen eisen voldoet.
- Een implementatie MOET het `id` genereren indien het niet ingestuurd is.

Steeds geldt bij `id` het volgende, wanneer er al een Resource met die `id` bestaat:

- Indien een implementatie eigenaarschap (<a href='#eigenaarschap'></a>) bijhoudt, en de inzender is niet eigenaar, dan MOET het verzoek worden afgewezen (`400 Bad Request`).
- Indien een implementatie eigenaarschap (<a href='#eigenaarschap'></a>) bijhoudt, en de inzender is wel eigenaar, dan MAG de resource worden geüpdatet.

Steeds geldt bij eigenschappen die verwijzen naar een [[=ResourceIdentifier=]]:

- Een implementatie MOET het verzoek afwijzen als er geen resource met die `id` bestaat.

<aside class="note" title="VeiligStallen">

De implementatie van VeiligStallen genereert {{Survey.id}}s op basis van gemeente-code, {{Survey.authority}} en {{Survey.contractor}}.

</aside>

### Eigenaarschap

Een implementatie kan op verschillende wijzes eigenaarschap van een [[=Resource=]] vaststellen en bijhouden.

- Een implementatie MAG authentication- en authorization-gegevens (<a href='#beveiliging'></a>) gebruiken om eigenaarschap van een Resource vast te leggen.
- Een implementatie MAG updateverzoeken aan een resource accepteren van een eigenaar van die resource.
- Een implementatie MAG updateverzoeken afwijzen als het niet aan eigen eisen voldoet.
- Een implementatie MAG updateverzoeken aan een resource afwijzen als de eigenaar van die resource, niet het verzoek heeft verstuurd.

Verschillende objecttypen hebben de eigenschap `authority` (namelijk:
{{Survey.authority}},
{{SurveyArea.authority}},
{{ParkingFacility.authority}},
{{Section.authority}}).

Steeds geldt bij een resource met een `authority` het volgende:

- Een implementatie MAG updateverzoeken accepteren als er een geldige {{Survey}} bestaat, waarvan de {{Survey.authority}} overeenkomt met de {{ParkingFacility.authority}} of {{Section.authority}}.

_De rest van deze sectie is niet normatief._

Deze authority zou de eigenaar van de resources moeten zijn, maar in de regel voeren aannemers de data in.
Daarom kan een vergelijking plaatsvinden om

### Geldigheid door de tijd

Verschillende objecttypen hebben het eigenschap-paar `validFrom` (geldig vanaf) en `validThrough` (geldig tot en met):
{{SurveyArea.validFrom}}, {{ParkingFacility.validFrom}}, {{Section.validFrom}},
{{SurveyArea.validThrough}}, {{ParkingFacility.validThrough}}, {{Section.validThrough}}.

Hierbij geldt steeds het volgende:

- Een implementatie MOET een update afwijzen als de waarde {{DynamicParkingFacility.timestamp}} voortijdig is aan de waarde van {{ParkingFacility.validFrom}}.
- Een implementatie MOET een update afwijzen als de waarde {{DynamicSection.timestamp}} voortijdig is aan de waarde van {{Section.validFrom}}.

- Een implementatie MOET een update afwijzen als de waarde {{DynamicParkingFacility.timestamp}} natijdig is aan de waarde van {{ParkingFacility.validThrough}}.
- Een implementatie MOET een update afwijzen als de waarde {{DynamicSection.timestamp}} natijdig is aan de waarde van {{Section.validThrough}}.

- De wijziging van een kenmerk maakt een nieuwe instantie noodzakelijk:
  - Het tijdstip van de wijziging minus één is de `validThrough` van de oude instantie.
    - Vanwege de “tot en met”-betekenis, MOET er één eenheid van de meetresolutie worden afgetrokken.
    - Bijvoorbeeld: bij een seconderesolutie: - 1 seconde.
    - Bijvoorbeeld: bij een dagresolutie: - 1 dag.
  - Het tijdstip van de wijziging is de `validFrom` van de nieuwe instantie.

### <dfn>`ResultWrapper`</dfn>

Endpoints met meerdere resultaten MOETEN in een `ResultWrapper` gestoken zijn.
Implementaties MOGEN metadata over de resultaten meegeven aan de `ResultWrapper`, zoals paginering, gebruikte filters, etc.
Endpoints met expliciet één resultaat MOGEN NIET via een `ResultWrapper` uitgeleverd worden.

| Eigenschap                               | Type | Kardinaliteit | Beschrijving           |
| ---------------------------------------- | ---- | ------------- | ---------------------- |
| <dfn data-dfn-for="ResultWrapper">result | `[]` | 1..N          | Verzameling van items. |
| {.data def}                              |

_De rest van deze sectie is niet normatief._

De ResultWrapper zorgt ervoor dat het type van wat een GET-request retourneert altijd een object (`{}`) is.

<aside class='note' title="Een mogelijke definitie van ResultWrapper in TypeScript">

```js
class ResultWrapper<T> {
  result: T[];
}
```

</aside>
<aside class='example' title="Gewrapped resultaat meervoudige bevraging">

```http
GET /static
```

```json
{
  "result": [{ "id": "..." }, { "id": "..." }]
}
```

</aside>
<aside class='example' title="Niet-gewrapped resultaat enkelvoudige bevraging">

```http
GET /organisations/defietsentellers
```

```json
{
  "id": "defietsentellers",
  "name": "De Fietsentellers BV"
}
```

</aside>

### Base-URL

Een implementatie MOET een base-URL bepalen waarvan de REST-API-endpoint worden aangeboden.
API-endpoints en resources worden daarvandaan geserveerd.
Bijvoorbeeld: `https://fiets-api.example.org/rest/v2/`, wat bij `/organisations` verwordt tot `https://fiets-api.example.org/rest/v2/organisations`.

Een implementatie MAG een versienummer in de base-url opnemen.

<aside class='note' title='VeiligStallen'>

In het geval van de pilot in VeiligStallen is de base-url `https://remote.veiligstallen2.nl/rest/api`.

</aside>

### REST-methodes

Een implementatie MOET de HTTP-methodes beschreven bij elk endpoint ondersteunen.
Een implementatie MAG ook andere HTTP-methodes bij dezelfde endpoints ondersteunen.

_De rest van deze sectie is niet normatief._

Als uitgangspunt geldt steeds het volgende:

- de HTTP-methode GET vraagt gegevens op
- de HTTP-methode POST voegt nieuwe gegevens toe
- de HTTP-methode PUT updatet een bestaande resource, aangegeven via een bepaalde identifier in de URL.

  Bij PUT worden alleen gewijzigde kenmerken meegegeven.

<section class='informative'>
<h3>Relaties tussen objecttypen</h3>

Onderstaande <a href='#relatie-objecttypen'></a> verduidelijkt de koppelingen die er bestaan tussen de verschillende objecttypen.

<figure id='relatie-objecttypen'><img src='docs/objecten.drawio.svg'><figcaption>Overzicht relaties tussen statische en dynamische gegevenselementen.</figcaption></figure>

</section>

## REST-API: organisaties

De volgende
Representeert de {{Organisation}}s die partij zijn bij een onderzoek of inwinning.

| HTTP-methode                   | Beschrijving                                                    |
| ------------------------------ | --------------------------------------------------------------- |
| <dfn>GET `/organisations`      | Toon geregistreerde organisaties.                               |
| <dfn>POST `/organisations`     | Voeg een organisatie toe.                                       |
| <dfn>GET `/organisations/{id}` | Toon organisaties waar {{Organisation.id}} = <var>id</var>.     |
| <dfn>PUT `/organisations/{id}` | Update de organisatie waar {{Organisation.id}} = <var>id</var>. |
| {.data def }                   |

<pre class='example json' title='POST /organisations' data-include='examples/organisations-post.json' data-include-format='text'></pre>

## REST-API: onderzoeken en inwinningen

Registreer en beheer {{Survey}}s en {{SurveyAreas}}s en leg de koppeling tussen beide.

| HTTP-methode             | Beschrijving                                         |
| ------------------------ | ---------------------------------------------------- |
| <dfn>GET `/surveys`      | Toon alle bestaande Surveys.                         |
| <dfn>POST `/surveys`     | Voeg een Survey toe.                                 |
| <dfn>GET `/surveys/{id}` | Toon Survey waar {{Survey.id}} = <var>id</var>.      |
| <dfn>PUT `/surveys/{id}` | Update de Survey waar {{Survey.id}} = <var>id</var>. |
| {.def}                   |

| HTTP-methode                        | Beschrijving                                                                                                |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| <dfn>GET `/surveys/{id}/areas`      | Toon SurveysAreas die voorkomen bij {{Survey.surveyArea}} bij de Survey waar {{Survey.id}} = <var>id</var>. |
| <dfn>POST `/surveys/{id}/areas`     | Voeg een SurveyArea toe aan een Survey waar {{Survey.id}} = <var>id</var>.                                  |
| <dfn>PUT `/surveys/{id}/areas/{id}` | Update de SurveyArea waar {{SurveyArea.id}} = <var>id₂</var>.                                               |
| {.def }                             |

<pre class='example json' title='POST /surveys' data-include='examples/surveys-post.json' data-include-format='text'></pre>
<pre class='example json' title='PUT /surveys' data-include='examples/surveys-id-put.json' data-include-format='text'></pre>

<pre class='example json' title='POST /surveys/{id}/areas' data-include='examples/surveys-id-areas-post.json' data-include-format='text'></pre>
<pre class='example json' title='POST /surveys/{id}/areas/{id}/subareas' data-include='examples/surveys-id-areas-id-subareas-post.json' data-include-format='text'></pre>

<aside class='issue'>
TODO: Op deze manier zijn SurveyAreas niet te bevragen zonder een Survey waar ze bij horen.
</aside>
<aside class='issue'>
TODO: /subareas zijn niet een fenomeen. Wijzig het API-voorbeeld.
</aside>

## REST-API: stallingen, parkeervoorzieningen

Registreer en beheer {{ParkingFacilities}} en bijbehorende {{Section}}s.

| HTTP-methode                                     | Query | Beschrijving                                                                                           |
| ------------------------------------------------ | ----- | ------------------------------------------------------------------------------------------------------ |
| <dfn>POST `/parkingfacilities`                   |       | Voeg een ParkingFacility toe.                                                                          |
| <dfn>GET `/parkingfacilities`                    | Ja    | Toon bestaande ParkingFacilities.                                                                      |
| <dfn>GET `/parkingfacilities/{id}`               |       | Toon de ParkingFacility waar {{ParkingFacility.id}} = <var>id</var>.                                   |
| <dfn>POST `/parkingfacilities/{id}/sections`     |       | Voeg een Section toe, waar {{Section.parkingFacility}} = <var>id</var>.                                |
| <dfn>GET `/parkingfacilities/{id}/sections`      | Ja    | Toon alle Sections, waar {{Section.parkingFacility}} = <var>id</var>.                                  |
| <dfn>GET `/parkingfacilities/{id}/sections/{id}` |       | Toon de Section, waar {{Section.parkingFacility}} = <var>id₁</var> en {{Section.id}} = <var>id₂</var>. |
| {.data def }                                     |

<pre class='example json' title='POST /parkingfacilities' data-include='examples/parkingfacilities-post.json' data-include-format='text'></pre>
<pre class='example json' title='POST /parkingfacilities/{id}/sections' data-include='examples/parkingFacilities-id-sections-post.json' data-include-format='text'></pre>

## REST-API: tellingen, metingen en capaciteit

Verkrijg gemeten aantallen fietsen in {{ParkingFacilities}} en bijbehorende {{Section}}s.

| HTTP-methode                                            | Beschrijving                                                                   |
| ------------------------------------------------------- | ------------------------------------------------------------------------------ |
| <dfn>GET `/parkingfacilities/{id}/count`                | Toon de tellingsinformatie voor de {{ParkingFacility.id}} = <var>id</var>.     |
| <dfn>POST `/parkingfacilities/{id}/count`               | Voeg de tellingsinformatie toe voor de {{ParkingFacility.id}} = <var>id</var>. |
| <dfn>GET `/parkingfacilities/{id}/sections/{id}/count`  | Toon de tellingsinformatie voor de sectie {{Section.id}} = <var>id₂</var>.     |
| <dfn>POST `/parkingfacilities/{id}/sections/{id}/count` | Voeg de tellingsinformatie toe voor de sectie {{Section.id}} = <var>id₂</var>. |
| {.data def }                                            |

<pre class='example json' title='GET /parkingfacilities/{id}/count' data-include='examples/parkingfacilities-id-count-get.json' data-include-format='text'></pre>
<pre class='example json' title='GET /parkingfacilities/{id}/sections/{id}/count' data-include='examples/parkingfacilities-id-sections-id-count.json' data-include-format='text'></pre>

# API 4 - data opvragen

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

**Filter op diepte**  
Gedetailleerde informatie over de bezetting geeft behoorlijk grote bomen, die niet voor iedere analist van deze data even interessant is. Als deze analist de waarnemingen van een stationsstalling door de tijd opvraagt, kunnen de responsen enorm groot worden. Als 90% van de data voor de analist niet relevant is, is dit natuurlijk onzinnig.
Daarom stelt de datastandaard een beperkt aantal zoekfunctionaliteiten verplicht. De belangrijkste van deze is dat je de mate van detail, de diepte (depth), bij een zoekopdracht kunt aangeven aan de API.
Stel dat in het dataportal de meest gedetailleerde boom is opgeslagen:

Als een analist alleen geïnteresseerd is in de bezetting van de totale stalling, vraagt hij data op op diepte 1. De API geeft hem dan dit resultaat:

Als een stalling iedere 2 minuten data opslaat en een analist vraagt de data op van een hele maand, scheelt dat uiteraard enorm in de datastroom en daarmee in de performance van de API.

**Filter op inhoud**  
Om data-analisten te helpen hun weg te vinden in de gestaag groeiende data van een dataportal, stelt de datastandaard een aantal zoekfunctionaliteiten verplicht. Zo moet een er minimaal gefilterd kunnen worden op:

- Data binnen een bepaalde tijdspanne (startDate, endDate)
- Data van een bepaalde secties (sectionId)
- Data van een bepaald onderzoek (surveyId)
- Data van een bepaalde opdrachtgever (authorityId)
- Data van een bepaalde dataleverancier (contractorId)
- Data binnen een bepaalde geo-polygoon of geo-punt + radius

Deze filters moeten met elkaar gecombineerd kunnen worden.

De praktijk moet uitwijzen of deze lijst voldoende is om aan alle wensen van de data-analisten te voldoen. Indien nodig zullen er meer zoekfuncties aan deze lijst worden toegevoegd.

**Filter op tijdspanne**  
Gebruik bij het zoeken de url-parameters startDate en endData. Tijdstippen worden altijd doorgegeven in [[rfc3339]] timestamp formaat.  
Dus:  
`?startDate=2020-01-01T0:00:00&endDate=2020-02-01T0:00:00`

**Filter op sectieId**  
Als een analist alleen data van bepaalde statische secties:  
`?staticSectionId=:staticSectionId`  
Of meerdere secties:  
`?staticSectionId=staticSectionId1,staticSectionId2,staticSectionId3`

**Filter op privateId**  
Als een analist alleen data wil van een bepaald fietstype, bijvoorbeeld gewone fietsen:  
`?privateId=:privateId`  
Of meerdere secties:  
`?privateId=privateId1,privateId2,privateId3`

**Filter op onderzoek**  
Data die ingestuurd wordt door een dataleverancier kan worden voorzien van een onderzoeksID (surveyID). Als deze ID gegeven is, is het uiteraard mogelijk hierop te filteren:  
`?surveyID=:surveyID`

**Filter op opdrachtgever**  
Hetzelfde idee al filteren op onderzoek: als de data voorzien is van de ID van een opdrachtgever, kan hierop gefilterd worden:  
`?authorityID=:authorityID`

**Filter op dataleverancier**  
De dataportal kan de data die wordt ingestuurd voorzien van de ID van de leverancier van deze data. Als dat gebeurt, kan er uiteraard gezocht worden op dit ID:  
`?contractorID=:contractorId`

**Geo zoekopdrachten**
Als de statische data van secties is voorzien van exacte geo-coördinaten, moeten deze secties gevonden kunnen worden aan de van een geo-zoekopdracht.

**Filter op geo-data: polygoon**  
Geef een polygoon mee in de query, aan de hand waarvan de corresponderende secties worden gezocht.  
Er zijn twee manieren voor deze query:

- zoek secties die de gegeven polygoon overlappen: relation=intersects. Dit is de default.  
  `?geopolygon=4.895168,52.370216,4.895168,53.370216,5.895168,53.370216,4.895168,52.370216&relation=intersects`
- zoek secties die geheel binnen de gegeven polygoon vallen: relation=within.  
  `?geopolygon=4.895168,52.370216,4.895168,53.370216,5.895168,53.370216,4.895168,52.370216&relation=within`

**Sorteren**  
Dynamische data moet gesorteerd kunnen worden opgevraagd. Dat kan met de parameters `orderBy` en `orderDirection`:  
`dynamicdata?orderBy=timestamp&orderDirection=ASC` - sorteert op timestamp van oudste naar nieuwste  
`dynamicdata?orderBy=count.numberOfVehicles&orderDirection=DESC` - sorteert op aantal voertuigen van hoog naar laag

#### Een overzicht van alle query parameters

| param            | type                 | values                                                        |
| ---------------- | -------------------- | ------------------------------------------------------------- |
| `surveyId`       | string               | Alleen data van dit onderzoek                                 |
| `authorityId`    | string               | Alleen data van deze opdrachtgever                            |
| `contarctorId`   | string               | Alleen data van deze dataleverancier                          |
| `depth`          | number               | Aantal te bevragen sectie-lagen vanaf gegeven pad default = 1 |
| `startDate`      | UTC timestamp        | Selectie op timestamp. Section.timestamp >= startDate         |
| `endDate`        | UTC timestamp        | Selectie op timestamp. Section.timestamp <= endDate           |
|                  |                      |                                                               |
| `geoPolygon`     | list met coördinaten | lat1,lng1,lat2,lng2,lat3,lng3,...,...,lat1,lng1               |
| `geoRelation`    | string               | 'intersects' (default) of 'within'                            |
|                  |                      |                                                               |
| `orderBy`        | string               | veldnaam waarop gesorteerd wordt                              |
| `orderDirection` | string               | ASC (default) of DESC, in combinatie met orderBy              |

{.data}

Query-params dienen hoofdletterongevoelig te zijn, dus authorityid=abc is hetzelfde als authorityID=abc

### API 5 - realtime data lezen vanuit dataportal voor t.b.v. webapplicaties

Gebruikers van API 5 - de datastroom tussen het dataportal en de webapplicaties - zijn vooral geïnteresseerd in realtime data. Per secties dus slechts één resultaat.
Het endpoint voor API 5 is `/latest`.

Het gaat in de `/latest`-requests altijd om dynamische data, dus `dynamicData` kan worden weggelaten uit het pad.
