## Dataschema’s

De data in de datastandaard valt uiteen in drie hoofdblokken:

1. data over het onderzoek (_survey_)
2. statische data (_static data_): dit betreft enerzijds onderzoeksgebieden (_survey areas_) en anderzijds gegevens over de meetgebieden (_parking facilities_ en _sections_): deze data verandert zelden of nooit.
3. dynamische data (_dynamische data_): tellingen en metingen in de meetgebieden

De velden en zoekfuncties die in deze datastandaard zijn opgenomen, MOETEN worden ondersteund door de dataportal. Het staat de dataportals vrij om zelf extra datavelden en extra functionaliteit te bieden.

| Field         | Type        | Required | Description                                           |
| ------------- | ----------- | -------- | ----------------------------------------------------- |
| `survey`      | Survey      | no       | Gegevens over het onderzoek                           |
| `staticData`  | StaticData  | no       | Statische data (sectienaam, adres, positie, ...)      |
| `dynamicData` | DynamicData | no       | Dynamische data (bezettingsdata, fietstellingen, ...) |
| {.data}       |

### Surveys – Onderzoeken

Het datablok Survey bevat data over het onderzoek. Geïnitieerd door wie? Uitgevoerd door wie? Wanneer? Waar? Al deze informatie kan worden ingestuurd, maar is niet verplicht.

Bij het insturen van een Survey kan de dataportal ervoor kiezen de gebruiker zijn eigen `Survey.id` te laten kiezen. Inzender van de nieuwe Survey wordt eigenaar van deze inzending. Als dat het geval is, dient er een 400-error (Bad request) gegenereerd te worden als er een `Survey.id` worden gekozen dat al bestaat. In geval de inzender de eigenaar is van de bestaande survey, vindt er een update plaats. Ondersteuning van deze functionalitet is door het dataportal zelf in te bepalen.

Het `Survey.id` kan gebruikt worden om data van verschillende secties en bronnen te groeperen door dynamische data te labelen met het veld `Survey.id`. Zie voor meer details [Dynamic Data](#dynamische-data).

> In grote lijnen kun je twee bronnen onderscheiden:
> Enerzijds digitale systemen die permanent data kunnen rapporteren over bezetting,
> en anderzijds tellingen die handmatig door tellers die een momentopname opgeven.

#### `Survey`

Een survey bevat vaste gegevens over een onderzoek.

| Field           | Type       | Required | Description                                                                                                                                                 |
| --------------- | ---------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`            | string     | yes      | Een uuid, random of eventueel samengesteld. Indien bij een POST-request niet gegeven, dan maakt de dataportal zelf een ID en geeft deze terug in de respons |
| `name`          | string     | yes      | Naam van het onderzoek                                                                                                                                      |
| `authorityId`   | string     | no       | `Organisation.id` van de Opdrachtgever - Alleen te gebruiken voor insturen van data                                                                         |
| `contractorIds` | string     | no       | `Organisation.id` van de contractors - Alleen te gebruiken voor insturen van data                                                                           |
| `license`       | string     | no       | Licentie van het gebruik van de data                                                                                                                        |
| `surveyArea`    | `string[]` | no       | ID van de onderzoeksgebieden die zijn meegenomen in het onderhavige onderzoek.                                                                              |
| {.data}         |

Opdrachtgever en uitvoerder zijn consistent over de levensduur van de survey.
De datastandaard stelt geen eisen aan de

<aside class="advisement">

De implementatie van VeiligStallen genereert Survey-ID's op basis van gemeente, `authority.id` en `contractor.id`.

</aside>

#### `Organisation`

De gegevens over een opdrachtgever of een uitvoerende instantie.

| Field   | Type   | Required | Description                                      |
| ------- | ------ | -------- | ------------------------------------------------ |
| `id`    | string | yes      | Unieke id, bijv. CBS-code gemeente of provincie. |
| `name`  | string | yes      | Naam van de instantie                            |
| {.data} |

(Hier zijn optioneel extra velden op te nemen, maar deze zijn geen onderdeel van de datastandaard)

### Gebiedsindelingen

Statische data is die data van `SurveyAreas`, `ParkingFacility`s en `Section`s die niet of nauwelijks aan verandering onderhevig zijn.
Het gaat vooral om de geografische afbakening.
Mocht het zo zijn dat de geografische afbakening wijzigt, dan verdient het aanbeveling nieuwe statische items aan te maken.
Aanpassingen van bestaande items gelden namelijk ook voor reeds ingestuurde data, wat kan leiden tot verwarring bij de interpretatie van historische data.
Of en hoe een statische items gewijzigd kan worden, valt buiten het bestek van deze standaard en wordt overgelaten aan de dataportals.

<aside class="advertisement" title="ProRail">

Voor ProRail:
Je definieert een `Survey` en je koppelt er `SurveyArea`s aan.
Vervolgens upload je alle `ParkingFacility`s en `Section`s in de betreffende `SurveyArea`s.
Daarna upload je dynamische data.

**Caveat**:  
bij nieuwe `ParkingFacility`s en `Section`s moet de eigenaar het bestaan ervan goedkeuren.

**Caveat**:  
Controleert het portal `Survey.ID` en `ParkingFacility.ID`.

</aside>

De parkeercapaciteit van een sectie lijkt op het eerste gezicht statisch. Toch is deze op advies van telinstanties niet opgenomen in de statische data. Wegwerkzaamheden en herindelingen van straten hebben te vaak invloed op de parkeercapaceit van straatsecties. In bewaakte stallingen kan het voorkomen dat bepaalde secties gedurende bepaalde periodes gereserveerd zijn.

#### `StaticData`

<aside class="advisement">

Dit moet weg, omdat het alleen bij responses terug komt.
Sowieso is het naamonderdeel Sectie niet meer juist, e.g. `StaticItems`.

</aside>

| Field    | Type            | Required | Description                      |
| -------- | --------------- | -------- | -------------------------------- |
| `result` | StaticSection[] | yes      | Verzameling van statische items. |
| {.data}  |

#### Survey Area

Een Onderzoeksgebied is een geometrische afbakening, waarmee individuele parkeerfaciliteiten samen geselecteerd kunnen worden voor verdere verwerking.
Bijvoorbeeld een stationsgebied, een strandboulevard of een uitgaansgebied.
De datastandaard biedt deze mogelijkheid zodat verschillende partijen dezelfde analyses zouden kunnen uitvoeren, op basis van de zelfde contouren van een gebied.
Niet elk onderzoek hoeft zijn Survey Area op te slaan, aangezien het platform geografische afbakeningen in een query ondersteunt.

Het is een hulpmiddel om parkeerlocaties in een bepaald gebied te selecteren.
Er is geen vaste relatie tussen onderzoeksgebieden en parkeerfaciliteiten: dat is puur een geografische relatie.
Ze zijn ook nuttig voor historische vergelijkingen.

<aside class="example">

Met er een onderscheid gemaakt worden in niveaus, voor bijv.

- Onderzoeksgebied
- Deelonderzoeksgebied

en voor:

- CBS Gemeente
- CBS Buurt
- CBS Wijk

Moet er niet in de datastandaard niet een `@type`-veld hiervoor?
Dan moet je ook de parent ervan aangeven.

</aside>

| Field          | Type                       | Required | Description                                                                                                            |
| -------------- | -------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------- |
| `id`           | string                     | yes      | Een uuid, random of eventueel samengesteld                                                                             |
| `geoLocation`  | GeoJSON (object)           | yes      | Geografische afbakening van deze sectie volgens [[rfc7946]]. Kan gebruikt worden voor geo-zoekopdrachten.              |
| `validFrom`    | [[rfc3339]] timestamp      | no       | Begin geldigheid                                                                                                       |
| `validThrough` | [[rfc3339]] timestamp      | no       | Einde geldigheid                                                                                                       |
| `authority`    | `Organisation.id` (string) | yes      | organisationId: Eigenaar van dit onderzoeksgebied. Alleen deze organistatie mag wijzigingen aanbrengen aan deze sectie |
| `label`        | string[]                   | no       | Naam die de eigenaar of inwinner aan dit onderzoeksgebied geeft.                                                       |
| `altLabel`     | string[]                   | no       | Alternatieve namen of IDs die de eigenaar of inwinner aan dit onderzoeksgebied geeft.                                  |
| `parent`       | `SurveyArea.id` (string)   | no       | Verwijzing naar een onderzoeksgebied van een hogerliggende orde.                                                       |
| {.data}        |

#### Parking Facility

Een Parkeerlocatie is een plek waar voertuigen geparkeerd kunnen worden, zoals een fietsenstalling, een plein of een trottoir.
Een parkeerlocatie kan onderverdeeld worden in 1 of meerdere [secties](#sections).
De parkeerlocatie heeft altijd een geometrie, minimaal een punt op de kaart, maar bij voorkeur een contour.
Kenmerken van de locatie, zoals openingstijden of bewaakt/onbewaakt worden op dit niveau geregistreerd (dit wordt later uitgewerkt, in lijn met VeloPark en SPDP).

Als een van onderstaande kenmerken van de parkeerlocatie verandert, moet de eigenaar (`authority`) een nieuwe parkeerlocatie aanmaken.
Het veld `validThrough` wordt dan ingevuld om de voorganger ongeldig te maken.
Wijzigingen in de onderliggende secties betekenen dus expliciet niet dat de parkeerlocatie wijzigt.
Andersom betekent een nieuwe parkeerlocatie wel dat ook de secties nieuw aangemaakt moeten worden -- zij verwijzen immers naar de parkeerlocatie waar ze bijhoren.

Voor historische vergelijkingen, kan op basis van de geometrie en/of `altLabel` bepaald worden of het om (min of meer) dezelfde parkeerlocatie gaat.

<aside class="note">

Wie bepaalt `altLabel` vs. `label`, vs. de `id`s van alles?
De eigenaar; alternatieve IDs van de inwinnaar moet die zelf beheren.

</aside>

| Field             | Type                  | Required    | Description                                                                                               |
| ----------------- | --------------------- | ----------- | --------------------------------------------------------------------------------------------------------- |
| `id`              | string                | yes         | Een uuid, random of eventueel samengesteld                                                                |
| `geoLocation`     | GeoJSON               | no          | Geografische afbakening van deze sectie volgens [[rfc7946]]. Kan gebruikt worden voor geo-zoekopdrachten. |
| `validFrom`       | [[rfc3339]] timestamp | no          | Dit item is geldig vanaf dit tijdstip.                                                                    |
| `validThrough`    | [[rfc3339]] timestamp | no          | Dit item is geldig tot dit tijdstip.                                                                      |
| `altLabel`        | string[]              | conditional | Alternatieve namen of IDs van de eigenaar of inwinner.                                                    |
| `securityFeature` | `SecurityFeature[]`   | conditional | Beveiligingskenmerken                                                                                     |
| `allowedVehicles` | `Vehicle[]`           | conditional | Toegestane voertuigtypen voor deze parkeerlocatie.                                                        |
| `authority`       | `Organisation.id`     | no          | Alleen deze eigenaar mag wijzigingen aanbrengen aan deze parkeerlocatie.                                  |
| {.data}           |

##### Beveiligingsvoorzieningen

| SecurityFeature       | Desc                          |
| --------------------- | ----------------------------- |
| Camera Surveillance   | Camerabewaking                |
| Locker Service        | Aanwezigheid van fietskluizen |
| Personnel Supervision | Bemenste stalling             |
| Electronic Access     | Toegang                       |
| Unsupervised          | Onbewaakte stalling (!)       |
| {.data}               |

#### Section

Een parkeerlocatie bestaat uit 1 of meerdere secties.
Binnen een sectie komt bij voorkeur één type parkeervoorziening voor.
Bijvoorbeeld: alleen etagerekken of alleen plekken voor buitenmodelfietsen.
Voordeel hiervan is dat de bezetting (zie dynamische data) herleid kan worden tot een bepaald type parkeervoorziening.
En dat er specifiek voor bepaalde locaties logische keuzen kunnen worden gemaakt.

Er is een grote vrijheid in het indelen in secties van een parkeerlocatie.
Bij digitale systemen kan de onderverdeling die in het systeem gemaakt wordt, leidend zijn.
Bijvoorbeeld: wordt in een parkeerverwijssysteem onderscheid gemaakt tussen boven- en onderlaag in een etagerek,
dan bevelen we aan hiervoor aparte secties aan te maken.

Er is een vaste administratieve koppeling tussen sectie en de parkeerlocatie waar het deel van uitmaakt.
Daarom is ook een geometrie van een sectie niet noodzakelijk om aan te leveren.
Voor bijvoorbeeld handmatige tellers kan dat wel handig zijn.

> Ook hier altLabel, label en id expliciet maken.

| Field             | Type                  | Required | Description                                                                                                   |
| ----------------- | --------------------- | -------- | ------------------------------------------------------------------------------------------------------------- |
| `id`              | string                | yes      | Een uuid, random of eventueel samengesteld                                                                    |
| `parkingFacility` | `ParkingFacility.id`  | yes      | ID die verwijst naar de parkeerlocatie waar het deel van uitmaakt.                                            |
| `altLabel`        | string                | no       | Private id die een contractor hanteert deze sectie                                                            |
| `geoLocation`     | GeoJSON               | no       | Geografische afbakening van deze sectie volgens [[rfc7946]]. Kan gebruikt worden voor geo-zoekopdrachten.     |
| `spaces`          | `Space[]`             | no       | De types parkeervoorziening.                                                                                  |
| `validFrom`       | [[rfc3339]] timestamp | no       | Vanaf dit tijdstip mag er dynamische data in deze sectie worden geschreven                                    |
| `validThrough`    | [[rfc3339]] timestamp | no       | Tot dit tijdstip mag er dynamische data in deze sectie worden geschreven                                      |
| `level`           | number                | no       | De etage (-1, 0 = begane grond, 1, etc.) in de parkeerlocatie waar deze sectie zich bevindt.                  |
| `authority`       | OrganisationID        | no       | organisationId: Eigenaar van deze sectie. Alleen deze organistatie mag wijzigingen aanbrengen aan deze sectie |
| {.data}           |

Een sectie kan een kortere `validThrough` hebben dan de parkeerlocatie waartoe het behoort:
als bijvoorbeeld een rek vervangen wordt door een buitenmodelvak.
De eigenaar van de sectie (`authority`) is verantwoordelijk hiervoor.
Bij handmatige tellingen is het goed denkbaar dat een veldwerker een extra sectie aanmaakt, die vervolgens goedgekeurd moet worden door de eigenaar.

<div class='issue' data-number="4"></div>

#### Places

Individuele parkeerplekken (_Places_) zijn voorzien als een toekomstige uitbreiding.

### Dynamische data

Dynamische data zijn de tellingen of metingen van het aantal plekken en/of het aantal geparkeerde voertuigen.
Dynamische data kan per sectie aangeleverd worden en per parkeerlocatie of per sectie uitgeleverd worden.
Teldata van de secties van één parkeerlocatie moeten dan wel op hetzelfde moment geüpdatet worden.

Merk op dat de totalen van de parkeerlocatie samen met de secties aangeleverd wordt:
het eerste tellings/metingsobject (in de ingestuurde JSON-array) beschrijft de parkeerlocatie, de andere de

> Biedt `@type` hier een oplossing voor?
> Of toch een JSON-objectstructuur?

> **Implementatiespecifiek**:
> Totalen van de parkeerlocatie kunnen dan met de updates vooraf berekend worden of pas bij uitlevering.

#### `DynamicData`

> Ook hier laten vallen, evt. naamgeving aanpassen.

| Field    | Type             | Required | Description                           |
| -------- | ---------------- | -------- | ------------------------------------- |
| `result` | DynamicSection[] | yes      | Verzameling secties met stallingsdata |
| {.data}  |

#### `DynamicParkingFacility`

> Maak onderscheid "Required" afhankelijk van of je instuurt of uitleest.

| Field                 | Type                    | Required    | Description                                                                             |
| --------------------- | ----------------------- | ----------- | --------------------------------------------------------------------------------------- |
| `parkingFacility`     | `ParkingFacility.id`    | yes         | Parkeerlocatie waar deze meting over gaat.                                              |
| `timestamp`           | [[rfc3339]] timestamp   | conditional | Tijdstip van de meting.                                                                 |
| `surveyId`            | string                  | conditional | Id van de survey waartoe deze meting behoort                                            |
| `parkingCapacity`     | number                  | conditional | Totaal aantal plekken, verplicht als één of meerdere onderdelen dit gemeten hebben.     |
| `vacantSpaces`        | number                  | conditional | Aantal vrije plekken.                                                                   |
| `occupiedSpaces`      | number                  | conditional | Aantal bezette plekken, verplicht als één of meerdere onderdelen dit gemeten hebben.    |
| `totalParked`         | number                  | conditional | Aantal getelde voertuigen, verplicht als één of meerdere onderdelen dit gemeten hebben. |
| `count`               | Count[]                 | no          | Verzameling van Count-objecten                                                          |
| `capacityBySpaceType` | `CapacityBySpaceType[]` | no          | Capaciteit per type parkeervoorziening, over de hele stalling. Sommering bij indienen.  |
| `space` **kan eruit** | Space                   | no          | Alleen toegestaan als één of meerdere onderdelen dit gemeten hebben.                    |
| `note`                | Note                    | no          | Notities over de meting in deze sectie                                                  |
| {.data}               |

Parkeercapaciteit wordt dynamisch geregistreerd, zodat kleine veranderingen aan de parkeervoorzieningen niet tot een nieuwe ParkingFacility leidt.

#### `DynamicSection`

Een telling of meting kan gaan over het aantal parkeerplekken (evt. naar type) en/of
het aantal geparkeerde voertuigen (evt. naar type) in een sectie.

| Field                 | Type                    | Required    | Description                                                                                        |
| --------------------- | ----------------------- | ----------- | -------------------------------------------------------------------------------------------------- |
| `section`             | `Section.id`            | yes         | Sectie waarop deze telling/meting betrekking heeft.                                                |
| `timestamp`           | [[rfc3339]] timestamp   | yes         | Tijdstip van de meting                                                                             |
| `surveyId`            | string                  | yes         | Id van de survey waartoe deze meting behoort                                                       |
| `parkingCapacity`     | number                  | conditional | Totaal aantal plekken, verplicht bij capaciteitstelling.                                           |
| `totalParked`         | number                  | conditional | Totaal aantal geparkeerde voertuigen, verplicht bij telling van het aantal geparkeerde voertuigen. |
| `capacityBySpaceType` | `CapacityBySpaceType[]` | no          | Capaciteit per type parkeervoorziening                                                             |
| `parkedByVehicleType` | `VehicleTypeCount[]`    | no          | Telling per geparkeerd voertuigtype                                                                |
| `occupiedSpaces`      | number                  | no          | Aantal bezette plekken, berekend o.b.v. `capacityBySpaceType` en `parkedByVehicleType`             |
| {.data}               |

De velden surveyId, authorityId en contractorId kunnen gebruikt worden bij het filteren van data bij de zoekopdrachten van de API's 4 en 5.
Je zou bijvoorbeeld kunnen alle metingen van een bepaald onderzoek die zijn uitgevoerd door een bepaalde contractor kunnen opvragen. Zie _API Requests_ voor meer details over zoekopdrachten.

#### `DynamicPlace`

Dit is voor later voorzien.

### `CapacityBySpaceType`

| Field              | Type         | Required | Description                                                                                  | ProRail |
| ------------------ | ------------ | -------- | -------------------------------------------------------------------------------------------- | ------- |
| `type`             | `Space.type` | no       | SpaceTypeID; tenminste 1 veld dient gegeven te zijn                                          | \*      |
| `vehicles`         | `Vehicle[]`  | no       | Deze parkeervoorziening is uitsluitend voor voertuigen voldoend aan de genoemde beperkingen. | \*      |
| `numberOfVehicles` | number       | no       | Capaciteit                                                                                   | \*      |
| {.data}            |

> **Voorbeeld**
> Etagerek-onder met capaciteit voor 10 huurfietsen `{ numberOfVehicles: 10, type: 'o', vehicles: [ { type: 'f', owner: 'h' } ] }`
>
> Vak voor 15 deelsnor- of -bromfietsen: `{ numberOfVehicles: 15, type: 'v', vehicles: [ { type: 'sb', owner: 'h' } ] }`

> **Vraag**  
> Kunnen we opvragen, alle plekken, ongeacht fysiek voorkomen, die bedoeld zijn voor `vehicleType: { type: 'f', owner: 'h' } `

#### `Space.type`

| ID      | spaceType         | ProRail |
| ------- | ----------------- | ------- |
| `x`     | geen voorziening  |
| `r`     | rek               | \*      |
| `e`     | etagerek          | \*      |
| `b`     | etagerek-boven    | \*      |
| `o`     | etagerek-onder    | \*      |
| `k`     | kluis             | \*      |
| `n`     | nietjes           |
| `v`     | vak               | \*      |
| `w`     | van fietsenwinkel |
| `a`     | anders            |
| {.data} |

### `Vehicle`

<div class='issue' data-number="9"></div>

| Field          | Type                                            | Required | Description                                     | ProRail |
| -------------- | ----------------------------------------------- | -------- | ----------------------------------------------- | ------- |
| `type`         | string                                          | no       | Zie tabel Vehicle.type                          | \*      |
| `propulsion`   | string[]                                        | no       | Zie tabel Vehicle.propulsion                    |
| `appearance`   | string                                          | no       | Zie tabel Vehicle.appearance                    | \*      |
| `state`        | VehicleState[]                                  | no       | Zie tabel VehicleState                          |
| `accessoires`  | Accessoire[]                                    | no       | Zie tabel Accessoire                            | \*      |
| `owner`        | string                                          | no       | Zie tabel Vehicle.owner                         | \*      |
| `wasCanonical` | `CanonicalVehicle.canonicalIdentifier` (string) | no       | Vertaald uit een survey-specifiek voertuigtype. |
| {.data}        |

#### `CanonicalVehicle`

Alternatief voor expliciete voertuigtyperingen.

| Field                 | Type      | Required | Description                                        | ProRail |
| --------------------- | --------- | -------- | -------------------------------------------------- | ------- |
| `canonicalIdentifier` | string    | yes      | Geregistreerd voertuigtype voor metingsdoeleinden. |
| `label`               | string    | no       | Leesbare naam van voertuigtype                     |         |
| `vehicle`             | Vehicle[] | yes      | Zie tabel Vehicle                                  |
| {.data}               |

Dan kun je op plekken waar `Vehicle` object wordt verwacht, óók een `CanonicalVehicle.canonicalIdentifier` string hebben.

<aside class="example" title="Canonieke voertuigen">

Eigenschappen die niet vermeld worden bij canonieke voertuigen, zijn niet bepalend geweest voor de categorie-indeling.
Dat betekent dus dat bij bijv. onderstaande _Normfiets_, het best een elektrische trapondersteuning (`propulsion = "s"`) of een fietstas (`accessoire.type = "t"`) kan hebben.
Het was alleen niet een kenmerkend onderscheid binnen de telling waarin de canonieke voertuigen worden onderkend.

```json
{
  "canonicalIdentifier": "nl.prorail.fietsparkeren.type.A",
  "label": "Normfiets",
  "vehicle": [
    {
      "type": "f",
      "owner": "p"
    }
  ]
}
```

```json
{
  "canonicalIdentifier": "nl.prorail.fietsparkeren.type.B",
  "label": "Beperkt afwijkend",
  "vehicle": [
    {
      // kratje voor
      "type": "f",
      "accessoires": [{ "type": "k", "position": "v" }],
      "owner": "p"
    },
    {
      // kinderzitje achter
      "type": "f",
      "accessoires": [{ "type": "z", "position": "a" }],
      "owner": "p"
    }
  ]
}
```

```json
{
  "canonicalIdentifier": "nl.prorail.fietsparkeren.type.C",
  "label": "Sterk afwijkend",
  "vehicle": [
    {
      "type": "f",
      "appearance": "x"
    }
  ]
}
```

```json
{
  "canonicalIdentifier": "nl.prorail.fietsparkeren.type.D",
  "label": "Snor- of bromfiets",
  "vehicle": [
    {
      "type": "sb",
      "owner": "p"
    }
  ]
}
```

</aside>

#### `Accessoire`

| Field      | Type   | Required | Description                            | ProRail |
| ---------- | ------ | -------- | -------------------------------------- | ------- |
| `type`     | string | no       | Zie tabel Vehicle.accessoire.typen     | \*      |
| `position` | string | no       | Zie tabel Vehicle.accessoire.positions |
| {.data}    |

##### `Vehicle.accessoire.typen`

| ID      | Omschrijving               | ProRail |
| ------- | -------------------------- | ------- |
| `z`     | kinderzitje                |
| `t`     | fietstas                   |
| `b`     | bagagedrager               |
| `k`     | krat/mand                  |
| `p`     | Beperkt afwijkend, nl. k,z | \*      |
| {.data} |

##### `Vehicle.accessoire.positions`

| ID      | Omschrijving |
| ------- | ------------ |
| `v`     | voor         |
| `a`     | achter       |
| {.data} |

#### `VehicleState`

| Field      | Type   | Required | Description                      |
| ---------- | ------ | -------- | -------------------------------- |
| `type`     | string | no       | Zie tabel Vehicle.state.type     |
| `position` | string | no       | Zie tabel Vehicle.state.position |
| {.data}    |

##### `Vehicle.state.type`

| ID      | Omschrijving |
| ------- | ------------ |
| `w`     | wrak         |
| `l`     | lekke band   |
| `z`     | zonder zadel |
| ...     | ...          |
| {.data} |

##### `Vehicle.state.position`

| ID      | Omschrijving |
| ------- | ------------ |
| `v`     | voor         |
| `a`     | achter       |
| {.data} |

#### `Vehicle.parkState`

| ID      | Omschrijving       | ProRail |
| ------- | ------------------ | ------- |
| `i`     | In voorziening     | \*      |
| `p`     | nabij voorziening  | \*      |
| `x`     | buiten voorziening | \*      |
| {.data} |

#### `Vehicle.type`

Classificatie naar wettelijke voertuigcategorie.

| ID      | Voertuigtype          | Omschrijving                                                                | ProRail                    |
| ------- | --------------------- | --------------------------------------------------------------------------- | -------------------------- |
| `f`     | fiets                 | Geen kenteken en geen verzekeringsplaatje                                   | \*                         |
| `s`     | snorfiets             | kentekenplaat is blauw, met een wit kader, wit opschrift en een hologram    | \*                         |
| `b`     | bromfiets             | kentekenplaat is geel, met een zwart kader, zwart opschrift en een hologram | \*                         |
| `sb`    | snor- of bromfiets    | kentekenplaat is geel of blauw, onderscheid wordt niet gemaakt.             | \*                         |
| `m`     | motorfiets            | NL geel, internationaal anders                                              |
| `g`     | gehandicaptenvoertuig | driewieler, scootmobiel, rolstoel, etc                                      | (zie `Vehicle.appearance`) |
| `a`     | anders                |                                                                             |
| {.data} |

#### `Vehicle.propulsion`

| ID      | Aandrijving            | Omschrijving                          |
| ------- | ---------------------- | ------------------------------------- |
| `s`     | Spierkracht            | bv traditionele fiets of voetganger   |
| `e`     | Elektrisch             | bv e-bike                             |
| `s`     | Elektrisch ondersteund | bv e-bike                             |
| `b`     | Verbrandingsmotor      | bv traditionele bromfiets, motorfiets |
| {.data} |

#### `Vehicle.appearance`

| ID      | Verschijningsvorm              | ProRail |
| ------- | ------------------------------ | ------- |
| `k`     | Kinderfiets                    |
| `r`     | Racefiets                      |
| `l`     | Ligfiets                       |
| `b`     | Bakfiets / Transportfiets      |
| `f`     | Fietskar                       |
| `v`     | Vouwfiets                      |
| `m`     | Mountainbike                   |
| `d`     | Driewieler                     |
| `t`     | Tandem                         |
| `g`     | Gehandicaptenvoertuig          |
| `x`     | Sterk afwijkend: nl. b,f,d,t,g | \*      |
| {.data} |

#### `Vehicle.owner`

| ID      | Eigenaar | Omschrijving                 | ProRail |
| ------- | -------- | ---------------------------- | ------- |
| `p`     | Privé    | Privéfiets                   | (std.)  |
| `l`     | Lease    | Leasefiets, zoals Swap Bikes |
| `h`     | Huur     | Huurfiets, zoals OV-fiets    | \*      |
| {.data} |

### `VehicleTypeCount`

| Field              | Type    | Required | Description                              | ProRail |
| ------------------ | ------- | -------- | ---------------------------------------- | ------- |
| `vehicle`          | Vehicle | yes      | Voertuigtype                             | \*      |
| `parkState`        | string  | no       | Zie tabel Vehicle.parkState              | \*      |
| `numberOfVehicles` | number  | yes      | Aantal gestalde voertuigen van dit type. | \*      |
| {.data}            |

> **Voorbeeld**:
> 6 particuliere fietsen in het rek: `{ vehicle: {type: 'f', owner: 'p'}, numberOfVehicles: 6, parkState: 'i'}`;
>
> 4 particuliere fietsen buiten het rek: `{ vehicle: {type: 'f', owner: 'p'}, numberOfVehicles: 4, parkState: 'x'}`;
>
> 100 normale fietsen buiten voorziening: `{ vehicle: { type: 'f' }, numberOfVehicles: 100, parkState: 'x' }`;

### `Note`

| Field                 | Type    | Required | Description                               |
| --------------------- | ------- | -------- | ----------------------------------------- |
| `isOpen`              | boolean | no       | is deze area, bijv. een stalling geopend? |
| `isHoliday`           | boolean | no       | Vakantie?                                 |
| `isEvent`             | boolean | no       | Evenement (markt, kermis, ...)?           |
| `isUnderConstruction` | boolean | no       | Werkzaamheden?                            |
| `remark`              | string  | no       | Vrij tekstveld                            |
| {.data}               |

<aside class='example' title="ProRail">

De gewenste voorzieningtypes voor ProRail:

- Rek = `{ type: 'r', vehicle: [{ owner: 'p', type: 'f' }] }` (voor normfietsen van normale mensen)
- Etagerek = `{ type: 'e', vehicle: [{ owner: 'p', type: 'f' }] }`
- Vak = `{ type: 'v', vehicles: [{ type: 'sb'}, { appearance: 'x' }] }` (snor-, brom- en sterk afwijkend model fiets)
- Kluis = `{ type: 'k', vehicles: [{ type: 'f' }]}`
- Buitenmodel (beperkt afwijkend)-rek = `{ type: 'r', vehicles: [{ type: 'f', accessoire: [{type: 'p'}] }] }`
- Buitenmodel (beperkt afwijkend)-rek etage = `{ type: 'o', vehicles: [{ type: 'f', accessoire: [{type: 'p'}] }] }`
- Deelfiets-rek = `{ type: 'r', vehicle: [{ type: 'f', owner: 'h' }] }`
- Deelfiets-vak = `{ type: 'v', vehicle: [{ type: 'f', owner: 'h' }] }`

De gewenste fietstypes voor ProRail:

- **Normfiets**
  - Fiets = `{ type: 'f', owner: 'p' }`
- **Beperkt afwijkend**
  - Fiets met kratje voor = `{ type: 'f', owner: 'p', accessoires: [ { type: 'k', position: 'v' } ]}`
  - Fiets met kinderzitje achter = `{ type: 'f', owner: 'p', accessoires: [ { type: 'z', position: 'a' } ]}`
- Geel kenteken = `{ type: 'b', owner: 'p' }`
- Blauw kenteken = `{ type: 's', owner: 'p' }` (Is dat te onderscheiden tov huur?)
- Deelfiets = `{ type: 'f', owner: 'h' }`
- Deelscooter = `{ type: 'sb', owner: 'h' }`
- Bakfiets = `{ type: 'f', appearance: 'x' }`
- Buitenmodelfiets = `{ type: 'f', accessoire: [{ type: 'p' }] }`
- Vierwieler = `{ type: 'f', appearance: 'x' }` (???)
- Overig = `{ type: 'a' }`

Is afwezigheid v/h kenmerk 'accessoire' afwezigheid v/e accessoire?
Of alleen voor fietsen (`{ type: 'f' }`)?
Moet je uitsluiten bij een _telling_ dat het privéfietsen zijn, en/of moet dat ook bij een capaciteitsmeting.

#### Maar wat is genoeg:

Voertuigen

| id  | type                                   | privé/huur |
| --- | -------------------------------------- | ---------- |
| A   | normfiets                              |
| B   | beperkt afwijkend (kratje/kinderzitje) |
| C   | sterk afwijkend                        |
| D   | snor-/bromfiets                        |

> Met OWL-CE deze groepen definiëren, die dan uiteindelijk in de data toch compact te maken.
> `ProRailVehicleType2021`

Rek/Vak/etc.

parkState

</aside>
