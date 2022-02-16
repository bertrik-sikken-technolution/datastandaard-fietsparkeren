# REST-API

Het informatiemodel beschreven in <a href='#dataschema-s'></a> bepaalt in algemene zin de objecttypen en eigenschappen van deze datastandaard.
Een implementatie van de datastandaard MOET ook een REST-API volgens deze sectie aanbieden.
Een implementatie MAG ook andere uitwisselingmogelijkheden aanbieden, zolang dat op een manier is die consistent is met de semantiek van <a href='#dataschema-s'></a>.

## Algemeen

### Wijze van communicatie

Deze REST-API veronderstelt communicatie over HTTP [[rfc7231]] en met gebruik van JSON [[ECMA-404]].

### REST-methodes

Een implementatie MOET de HTTP-methodes beschreven bij elk endpoint ondersteunen.
Een implementatie MAG ook andere HTTP-methodes bij dezelfde endpoints ondersteunen.

_De rest van deze sectie is niet normatief._

Als uitgangspunt geldt steeds het volgende:

- de HTTP-methode GET vraagt gegevens op
- de HTTP-methode POST voegt nieuwe gegevens toe
- de HTTP-methode PUT updatet een bestaande resource, aangegeven via een bepaalde identifier in de URL.

  Bij PUT worden alleen gewijzigde kenmerken meegegeven.

### Kardinaliteit

De aangegeven kardinaliteiten in het informatiemodel geldt voor bij het insturen van gegevens.
Bij het uitleveren van gegevens MAG een implementatie sommige gegevens achterwege laten.

_De rest van deze sectie is niet normatief._

Een implementatie kan bijv. uit overwegingen op het gebied van privacy of concurrentiegevoeligheid bepaalde gegevens achterwege laten.

### Base-URL

Een implementatie MOET een base-URL bepalen waarvan de REST-API-endpoint worden aangeboden.
API-endpoints en resources worden daarvandaan geserveerd.
Bijvoorbeeld: `https://fiets-api.example.org/rest/v2/`, wat bij `/organisations` verwordt tot `https://fiets-api.example.org/rest/v2/organisations`.

Een implementatie MAG een versienummer in de base-url opnemen.

<aside class='note' title='VeiligStallen'>

In het geval van de pilot in VeiligStallen is de base-url `https://remote.veiligstallen2.nl/rest/api`.

</aside>

<section class='informative'>

### Relaties tussen objecttypen

Onderstaande <a href='#relatie-objecttypen'></a> verduidelijkt de koppelingen die er bestaan tussen de verschillende objecttypen.

<figure id='relatie-objecttypen'><img src='docs/objecten.drawio.svg'><figcaption>Overzicht relaties tussen statische en dynamische gegevenselementen.</figcaption></figure>

</section>

## Algoritmes voor eigenschappen en waardes

### Onbekende eigenschappen of waardes

Een implementatie MAG onbekende waardes of onverwachte datatypes afwijzen.
Een user agent MAG onbekende waardes of onverwachte datatypes afwijzen.

Een implementatie MAG altijd meer eigenschappen versturen naar een client.
Een user agent MOET onbekende eigenschappen accepteren.

Een implementatie MOET meer eigenschappen accepteren van een client.
Een implementatie MAG onbekende eigenschappen afwijzen.

_De rest van deze sectie is niet normatief._

Deze regels zorgen ervoor dat de datastandaard flexibel genoeg is voor toekomstige uitbreidingen.

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

De implementatie van VeiligStallen genereert {{Survey.id}}s op basis van CBS-gemeente-code, {{Survey.authority}} en {{Survey.contractor}}.

</aside>

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

### Geldigheid van geo-kenmerken

- Een implementatie ZOU een update MOETEN afwijzen als
  de waarde van {{ParkingFacility.geoLocation}} (1) overlapt met de {{ParkingFacility.geoLocation}} (2)
  waarbij ook de [[[#geldigheid-door-de-tijd]]] elkaar overlappen,
  behorende bij eenzelfde {{Survey}}.

  - Een implementatie MAG aan de gebruiker een melding geven als dit gebeurt.

## Beveiliging, toegang, privacy

### Eigenaarschap

Een implementatie kan op verschillende wijzes eigenaarschap van een [[=Resource=]] vaststellen en bijhouden.

- Een implementatie ZOU authenticatie- en authorisatie-gegevens (<a href='#beveiliging'></a>) MOETEN gebruiken om eigenaarschap van een Resource vast te leggen.
- Een implementatie ZOU updateverzoeken aan een resource MOETEN accepteren van een eigenaar van die resource.
- Een implementatie MAG updateverzoeken afwijzen als het niet aan eigen eisen voldoet.
- Een implementatie ZOU updateverzoeken aan een resource MOETEN afwijzen als de eigenaar van die resource, niet het verzoek heeft verstuurd.

Verschillende objecttypen hebben de eigenschap `authority` (namelijk:
{{Survey.authority}},
{{SurveyArea.authority}},
{{ParkingFacility.authority}},
{{Section.authority}}).

Steeds geldt bij een resource met een `authority` het volgende:

- Een implementatie MAG updateverzoeken accepteren als er een geldige {{Survey}} bestaat, waarvan de {{Survey.authority}} overeenkomt met de {{ParkingFacility.authority}} of {{Section.authority}}.

_De rest van deze sectie is niet normatief._

Deze authority zou de eigenaar van de resources moeten zijn, maar in de regel voeren aannemers de data in.
Daarom kan een vergelijking plaatsvinden om niet-matriële wijzigingen aan Resources

<section class='informative'>

### Beveiliging

Waarschijnlijk wil een dataportal het gebruik van de API beveiligen, zeker als het gaat om het insturen van data (de POST-requests van API 3).
Om discussies over de voor- en nadelen van diverse authenticatie-protocollen te voorkomen, schrijft de datastandaard niet voor op welke manier deze beveiliging uitgevoerd dient te worden.

Het is aan te raden om `authority` en `contractor` te gebruiken voor authenticatie.
Deze `id`’s kunnen dan dienen om de ingestuurde data te labellen en daar zoekopdrachten aan te koppelen.
Als bijvoorbeelde contractor met id `de_fietstellers_bv` dynamische data instuurt, kan elke {{Section}} uitgebreid worden met een veld `Section.contractor` met de waarde `"de_fietstellers_bv"`.

</section>

## Responses

### <dfn>`ResultWrapper`</dfn>`<T>`

Endpoints met meerdere resultaten MOETEN de [[=Response=]] in een `ResultWrapper` gestoken zijn.
Implementaties MOGEN metadata over de resultaten meegeven aan de `ResultWrapper`, zoals paginering, gebruikte filters, etc.
Endpoints met expliciet één resultaat MOGEN NIET via een `ResultWrapper` uitgeleverd worden.

| Eigenschap                               | Type  | Kardinaliteit | Beschrijving                    |
| ---------------------------------------- | ----- | ------------- | ------------------------------- |
| <dfn data-dfn-for="ResultWrapper">result | `T[]` | 1..N          | Verzameling van één type items. |
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

### Resultaten filteren

Sommige endpoints leveren grote lijsten resultaten terug.
Voor gebruiksgemak en efficiëntie, kunnen queryparameters worden toegevoegd

Voor elk endpoint met een GET-methode, waar meerdere resultaten terugkomen, geldt het volgende:

- Elk GET endpoint met meerdere resultaten (en dus verpakt in een {{ResultWrapper}}) MOET filtering o.b.v. queryparameters ondersteunen.

Voor queryparameters geldt in het algemeen:

- Een implementatie MOET de bij een endpoint aangegeven queryparameters ondersteunen.
- Een implementatie MAG extra queryparameters ondersteunen.
- Een implementatie MOET onbekende queryparameters accepteren.
- Een implementatie MOET de response waarbij onbekende queryparameters niet zijn verwerkt, accepteren.
- Een implementatie MOET queryparameter-namen hoofdletterongevoelig parseren, dus {{authorityid}}`=abc` is hetzelfde als {{AuthorityID}}`=abc`.
- Een implementatie MAG queryparameters combineren voor een specifiekere filtering.

<aside class='issue'>
Het is zeer verboos en niet per se duidelijk om bij elk GET-endpoint aan te geven welke filters mogelijk zijn.
Misschien beter om algemene filters (<a href='#sortering-en-paginering'></a>) altijd van toepassing verklaren?
</aside>

#### Tijdsspanne

Deze queryparameters filteren de gegevenslijst op gegevens geregistreerd of geldig binnen een bepaalde tijdspanne.

| Parameter      | Type                             | Waarde                                                                                                         |
| -------------- | -------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| <dfn>startDate | [[rfc3339]] date-time (`string`) | Filteren op tijstip van de meting, waarden hebben een `timestamp` of `validFrom` >= <var>startDate</var>.      |
| <dfn>endDate   | [[rfc3339]] date-time (`string`) | Filteren op tijstip van de meting, waarden hebben een `timestamp` of een `validThrough` <= <var>endDate</var>. |
| {.data def}    |

De standaardwaardes zijn implementatie-specifiek.

<aside class='example'>

`?startDate=2020-01-01T0:00:00&endDate=2020-02-01T0:00:00`

</aside>

#### Sortering en paginering

Deze queryparameters sorteren de gegevenslijst op een bepaalde volgorde.
Optioneel kan met {{limit}} en {{offset}} paginering worden toegepast.

| Parameter           | Type     | Waarde                                                                                                 |
| ------------------- | -------- | ------------------------------------------------------------------------------------------------------ |
| <dfn>orderBy        | `string` | Veldnaam (of veld-pad) waarop gesorteerd wordt.                                                        |
| <dfn>orderDirection | `string` | `ASC` (default) of `DESC`, in combinatie met {{orderBy}}.                                              |
| <dfn>limit          | `number` | Beperkt het aantal resultaten tot dit aantal.                                                          |
| <dfn>offset         | `number` | Wanneer het aantal resultaten beperkt is, worden het aantal <var>offset</var> resultaten overgeslagen. |
| {.data def}         |

De standaardwaardes zijn implementatie-specifiek.

<aside class='example'>

- Sorteer op timestamp van oudste naar nieuwste

  `dynamicdata?orderBy=timestamp&orderDirection=ASC`

- Sorteer op aantal voertuigen van hoog naar laag

  `dynamicdata?orderBy=count.numberOfVehicles&orderDirection=DESC`

</aside>

#### Geo-zoekopdrachten

Als objecttypen voorzien zijn van exacte geo-coördinaten, MOET een implementatie deze objecttypen kunnen filteren op basis van een geo-zoekopdracht.

| Parameter        | Type                                                | Waarde                                          |
| ---------------- | --------------------------------------------------- | ----------------------------------------------- |
| <dfn>geoPolygon  | kommagescheiden lijst van breedtegraad, lengtegraad | lat1,lng1,lat2,lng2,lat3,lng3,...,...,lat1,lng1 |
| <dfn>geoRelation | `string`                                            | `intersects` (default) of `within`              |
| {.data def}      |

<aside class='example'>

- Zoek secties die de gegeven polygoon overlappen: relation=intersects.

  `?geopolygon=4.895168,52.370216,4.895168,53.370216,5.895168,53.370216,4.895168,52.370216&relation=intersects`

- Zoek secties die geheel binnen de gegeven polygoon vallen: relation=within.

  `?geopolygon=4.895168,52.370216,4.895168,53.370216,5.895168,53.370216,4.895168,52.370216&relation=within`

</aside>

#### Inhoudelijk

Alle tellingsgegevens worden in de context van een {{Survey}} ingevoerd (cf. <a href='#relatie-objecttypen'></a>).
Ook {{SurveyArea}}s hebben een relatie tot een Survey.

Een implementatie MOET kunnen filteren op {{authorityID}}, {{contractorID}} en {{surveyID}}.

| Parameter         | Type     | Waarde                                                         |
| ----------------- | -------- | -------------------------------------------------------------- |
| <dfn>surveyID     | `string` | Alleen data van een onderzoek met deze {{Survey.id}}.          |
| <dfn>authorityID  | `string` | Alleen data van de opdrachtgever met deze {{Organisation.id}}. |
| <dfn>contractorID | `string` | Alleen data van de aannemer met deze {{Organisation.id}}.      |
| {.data def}       |

## REST-API: organisaties

De volgende endpoints representeren de {{Organisation}}s die partij zijn bij een onderzoek of inwinning.

| HTTP-methode                   | Type                                    | Beschrijving                                                    |
| ------------------------------ | --------------------------------------- | --------------------------------------------------------------- |
| <dfn>GET `/organisations`      | {{ResultWrapper}}`<`{{Organisation}}`>` | Toon geregistreerde organisaties.                               |
| <dfn>POST `/organisations`     | {{Organisation}}                        | Voeg een organisatie toe.                                       |
| <dfn>GET `/organisations/{id}` | {{Organisation}}                        | Toon organisaties waar {{Organisation.id}} = <var>id</var>.     |
| <dfn>PUT `/organisations/{id}` | `Partial<`{{Organisation}}`>`           | Update de organisatie waar {{Organisation.id}} = <var>id</var>. |
| {.data def }                   |

GET ondersteunt filters:

<ul class='filter-list' data-sort>
<li>{{orderBy}}
<li>{{orderDirection}}
<li>{{limit}}
<li>{{offset}}
</ul>

<pre class='example json' title='POST /organisations' data-include='examples/organisations-post.json' data-include-format='text'></pre>

## REST-API: onderzoeken en inwinningen

Registreer en beheer Surveys en SurveyAreas en leg de koppeling tussen beide.

| HTTP-methode             | Type                              | Beschrijving                                         |
| ------------------------ | --------------------------------- | ---------------------------------------------------- |
| <dfn>GET `/surveys`      | {{ResultWrapper}}`<`{{Survey}}`>` | Toon alle bestaande Surveys.                         |
| <dfn>POST `/surveys`     | {{Survey}}                        | Voeg een Survey toe.                                 |
| <dfn>GET `/surveys/{id}` | {{Survey}}                        | Toon Survey waar {{Survey.id}} = <var>id</var>.      |
| <dfn>PUT `/surveys/{id}` | `Partial<`{{Survey}}`>`           | Update de Survey waar {{Survey.id}} = <var>id</var>. |
| {.data def}              |

**Standaardfilters voor <a href='#dfn-get-surveys'>GET `/surveys`</a>**

<ul class='filter-list' data-sort>
<li>{{orderBy}}
<li>{{orderDirection}}
<li>{{limit}}
<li>{{offset}}
<li>{{geoPolygon}}
<li>{{geoRelation}}
<li>{{surveyID}}
<li>{{authorityID}}
<li>{{contractorID}}
<li>{{startDate}}
<li>{{endDate}}
</ul>

| HTTP-methode                  | Type                                  | Beschrijving                                                                                                 |
| ----------------------------- | ------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| <dfn>GET `/survey-areas`      | {{ResultWrapper}}`<`{{SurveyArea}}`>` | Toon SurveysAreas die voorkomen bij {{Survey.surveyAreas}} bij de Survey waar {{Survey.id}} = <var>id</var>. |
| <dfn>POST `/survey-areas`     | {{SurveyArea}}                        | Voeg een SurveyArea toe aan een Survey waar {{Survey.id}} = <var>id</var>.                                   |
| <dfn>PUT `/survey-areas/{id}` | `Partial<`{{SurveyArea}}`>`           | Update de SurveyArea waar {{SurveyArea.id}} = <var>id</var>.                                                 |
| {.data def }                  |

**Standaardfilters voor <a href='#dfn-get-surveys-id-areas'>GET `/surveys/{id}/areas`</a>**

<ul class='filter-list' data-sort>
<li>{{orderBy}}
<li>{{orderDirection}}
<li>{{limit}}
<li>{{offset}}
<li>{{geoPolygon}}
<li>{{geoRelation}}
<li>{{surveyID}}
<li>{{authorityID}}
<li>{{contractorID}}
<li>{{startDate}}
<li>{{endDate}}
</ul>

<pre class='example json' title='POST /surveys' data-include='examples/surveys-post.json' data-include-format='text'></pre>
<pre class='example json' title='PUT /surveys' data-include='examples/surveys-id-put.json' data-include-format='text'></pre>

<pre class='example json' title='POST /surveys/{id}/areas' data-include='examples/surveys-id-areas-post.json' data-include-format='text'></pre>
<pre class='example json' title='POST /surveys/{id}/areas' data-include='examples/surveys-id-areas-parent-post.json' data-include-format='text'></pre>

<aside class='issue'>
Op deze manier zijn {{SurveyArea}}s niet te bevragen zonder een {{Survey}} waar ze bij horen.
</aside>

## REST-API: stallingen, parkeervoorzieningen

Registreer en beheer ParkingFacilities en bijbehorende Sections.

| HTTP-methode                                        | Type                                       | Beschrijving                                                                                            |
| --------------------------------------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------- |
| <dfn>POST `/parkingfacilities`                      | {{ParkingFacility}}                        | Voeg een ParkingFacility toe.                                                                           |
| <dfn>GET `/parkingfacilities`                       | {{ResultWrapper}}`<`{{ParkingFacility}}`>` | Toon bestaande ParkingFacilities.                                                                       |
| <dfn>GET `/parkingfacilities/{id}`                  | {{ParkingFacility}}                        | Toon de ParkingFacility waar {{ParkingFacility.id}} = <var>id</var>.                                    |
| <dfn>POST `/parkingfacilities/{id}/sections`        | {{Section}}                                | Voeg een Section toe, waar {{Section.parkingFacility}} = <var>id</var>.                                 |
| <dfn>GET `/parkingfacilities/{id}/sections`         | {{ResultWrapper}}`<`{{Section}}`>`         | Toon alle Sections, waar {{Section.parkingFacility}} = <var>id</var>.                                   |
| <dfn>GET `/parkingfacilities/{pfid}/sections/{sid}` | {{Section}}                                | Toon de Section, waar {{Section.parkingFacility}} = <var>pfid</var> en {{Section.id}} = <var>sid</var>. |
| {.data def }                                        |

<pre class='example json' title='POST /parkingfacilities' data-include='examples/parkingfacilities-post.json' data-include-format='text'></pre>
<pre class='example json' title='POST /parkingfacilities/{id}/sections' data-include='examples/parkingFacilities-id-sections-post.json' data-include-format='text'></pre>

## REST-API: tellingen, metingen en capaciteit

TODO: Algoritme voor combineren van onderdelen.

Verkrijg gemeten aantallen fietsen in {{ParkingFacilities}} en bijbehorende {{Section}}s.

| HTTP-methode                                               | Type                                              | Beschrijving                                                      |
| ---------------------------------------------------------- | ------------------------------------------------- | ----------------------------------------------------------------- |
| <dfn>GET `/parkingfacilities/{id}/count`                   | {{ResultWrapper}}`<`{{DynamicParkingFacility}}`>` | Toon alle tellingen waar {{ParkingFacility.id}} = <var>id</var>.  |
| <dfn>GET `/parkingfacilities/{id}/latest`                  | {{DynamicParkingFacility}}                        | Toon alle tellingen waar {{ParkingFacility.id}} = <var>id</var>.  |
| <dfn>POST `/parkingfacilities/{id}/count`                  | {{DynamicParkingFacility}}                        | Voeg een telling toe waar {{ParkingFacility.id}} = <var>id</var>. |
| <dfn>GET `/parkingfacilities/{pfid}/sections/{sid}/count`  | {{ResultWrapper}}`<`{{DynamicSection}}`>`         | Toon alle tellingen waar {{Section.id}} = <var>sid</var>.         |
| <dfn>GET `/parkingfacilities/{pfid}/sections/{sid}/latest` | {{DynamicSection}}                                | Toon alle tellingen waar {{Section.id}} = <var>sid</var>.         |
| <dfn>POST `/parkingfacilities/{pfid}/sections/{sid}/count` | {{DynamicSection}}                                | Voeg een telling toe waar {{Section.id}} = <var>sid</var>.        |
| {.data def }                                               |

Gebruikers van API 5 — de datastroom tussen het dataportal en de webapplicaties — zijn vooral geïnteresseerd in realtime data.
Per {{ParkingFacility}} of {{Section}} dus slechts één resultaat, het meest recente.
Voor deze gebruikers zijn de endpoints
<a href='#dfn-get-parkingfacilities-id-latest'>GET `/parkingfacilities/{id}/latest`</a>
en
<a href='#dfn-get-parkingfacilities-id-sections-id-latest'>GET `/parkingfacilities/{id}/sections/{id}/latest`</a>
gemaakt.

<pre class='example json' title='GET /parkingfacilities/{id}/count' data-include='examples/parkingfacilities-id-count-get.json' data-include-format='text'></pre>
<pre class='example json' title='GET /parkingfacilities/{id}/sections/{id}/count' data-include='examples/parkingfacilities-id-sections-id-count.json' data-include-format='text'></pre>

<aside class='issue'>
TODO:
Moet er ook een algemene mogelijkheid zijn om statistieken over alle parkingfacilities gevonden met Query Parameters?

Eerder stond dit ook:

> **Filter op inhoud**  
> Om data-analisten te helpen hun weg te vinden in de gestaag groeiende data van een dataportal, stelt de datastandaard een aantal zoekfunctionaliteiten verplicht. Zo moet een er minimaal gefilterd kunnen worden op:
>
> - Data van een bepaalde secties (sectionId)
>
> De praktijk moet uitwijzen of deze lijst voldoende is om aan alle wensen van de data-analisten te voldoen. Indien nodig zullen er meer zoekfuncties aan deze lijst worden toegevoegd.
>
> **Filter op sectieId**  
> Als een analist alleen data van bepaalde statische secties:  
> `?staticSectionId=:staticSectionId`  
> Of meerdere secties:  
> `?staticSectionId=staticSectionId1,staticSectionId2,staticSectionId3`

</aside>
