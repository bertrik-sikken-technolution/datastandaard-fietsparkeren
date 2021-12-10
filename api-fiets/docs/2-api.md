# REST-API

Het informatiemodel beschreven in <a href='#dataschema-s'></a> bepaalt in algemene zin de objecttypen en eigenschappen van deze datastandaard.
Een implementatie van de datastandaard MOET ook een REST-API volgens deze sectie aanbieden.
Een implementatie MAG ook andere uitwisselingmogelijkheden aanbieden, zolang dat op een manier is die consistent is met de semantiek van <a href='#dataschema-s'></a>.

Deze REST-API veronderstelt communicatie over HTTP [[rfc7231]] en met gebruik van JSON [[ECMA-404]].

## Algemeen

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

### Relaties tussen objecttypen

Onderstaande <a href='#relatie-objecttypen'></a> verduidelijkt de koppelingen die er bestaan tussen de verschillende objecttypen.

<figure id='relatie-objecttypen'><img src='docs/objecten.drawio.svg'><figcaption>Overzicht relaties tussen statische en dynamische gegevenselementen.</figcaption></figure>

## API 3 — data opslaan

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
Een id van de survey mag door exploitant zelf gekozen worden. Suggestie gebruik [CBS codes](https://www.cbs.nl/nl-nl/onze-diensten/methoden/classificaties/overig/gemeentelijke-indelingen-per-jaar/indeling-per-jaar/gemeentelijke-indeling-op-1-januari-2020) en unieke gegevens uit het onderzoek, bijv. < CBS*nr_gemeente >*< jaartal > => 0202_2020.

<pre class='example json' title='Body zonder surveyId' data-include='../examples/API3/requests/POST_new_survey_without_id.json' data-include-format='text'></pre>

De instantie die de survey instuurt, wordt 'eigenaar' van dit onderzoek.

Een Survey met een `surveyId` dat al bestaat, zal resulteren in een 409-error (Conflict). Behalve als deze POST wordt gedaan door de eigenaar van het onderzoek. In dat geval wordt de bestaande survey overschreven.

Indien geen `surveyId` is gegeven, maakt de server zelf een id en geeft deze terug in de response.

[Response](./examples/API3/responses/POST_new_survey.json)

<pre class='example json' title="Response" data-include='../examples/API3/responses/POST_new_survey.json' data-include-format='text'></pre>

`POST /static` [Body](./examples/API3/requests/POST_static_section.json)  
Statische secties worden per stuk gepost. De body bevat dus slechts één sectie. Dit om eventuele schrijffouten te kunnen retourneren in de response, bijvoorbeeld als de `id` van de sectie reeds in gebruik is.

<pre class='example json' title="Body" data-include='../examples/API3/requests/POST_static_section.json' data-include-format='text'></pre>

[Response](./examples/API4/POST_static_section.json) - In geval van een succesvolle opslag, bevat de response het opgeslagen object in property `section`.

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
