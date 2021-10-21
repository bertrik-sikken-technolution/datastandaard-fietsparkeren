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

|{.data}

### Surveys

Het datablok Survey bevat data over het onderzoek. Geïnitieerd door wie? Uitgevoerd door wie? Wanneer? Waar? Al deze informatie kan worden ingestuurd, maar is niet verplicht.

Bij het insturen van een Survey kan de dataportal ervoor kiezen de gebruiker zijn eigen `Survey.id` te laten kiezen. Inzender van de nieuwe Survey wordt eigenaar van deze inzending. Als dat het geval is, dient er een 400-error (Bad request) gegenereerd te worden als er een `Survey.id` worden gekozen dat al bestaat. In geval de inzender de eigenaar is van de bestaande survey, vindt er een update plaats. Ondersteuning van deze functionalitet is door het dataportal zelf in te bepalen.

Het `Survey.id` kan gebruikt worden om data van verschillende secties en bronnen te groeperen door dynamische data te labelen met het veld `Survey.id`. Zie voor meer details [Dynamic Data](#dynamische-data).

> In grote lijnen kun je twee bronnen onderscheiden:
> Enerzijds digitale systemen die permanent data kunnen rapporteren over bezetting,
> en anderzijds tellingen die handmatig door tellers die een momentopname opgeven.

#### `Survey`

Een survey bevat vaste gegevens over een onderzoek.

| Field           | Type   | Required | Description                                                                                                                                                 |
| --------------- | ------ | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`            | string | yes      | Een uuid, random of eventueel samengesteld. Indien bij een POST-request niet gegeven, dan maakt de dataportal zelf een ID en geeft deze terug in de respons |
| `name`          | string | yes      | Naam van het onderzoek                                                                                                                                      |
| `authorityId`   | string | no       | `Organisation.id` van de Opdrachtgever - Alleen te gebruiken voor insturen van data                                                                         |
| `contractorIds` | string | no       | `Organisation.id` van de contractors - Alleen te gebruiken voor insturen van data                                                                           |
| `license`       | string | no       | Licentie van het gebruik van de data                                                                                                                        |
| {.data}         |

#### `Organisation`

De gegevens over een opdrachtgever of een uitvoerende instantie.

| Field   | Type   | Required | Description                                      |
| ------- | ------ | -------- | ------------------------------------------------ |
| `id`    | string | yes      | Unieke id, bijv. CBS-code gemeente of provincie. |
| `name`  | string | yes      | Naam van de instantie                            |
| {.data} |

(Hier zijn optioneel extra velden op te nemen, maar deze zijn geen onderdeel van de datastandaard)

### Statische data

Statische data is die data van secties die niet of nauwelijks aan verandering onderhevig zijn. Dat is bijvoorbeeld het geval bij de geografische afbakening.
Mocht het zo zijn dat de geografische afbakening wijzigt, dan verdient het aanbeveling een nieuwe statische sectie aan te maken. Aanpassingen van bestaande secties gelden namelijk ook voor reeds ingestuurde data, wat kan leiden tot verwarring bij de interpretatie van historische data.
Of en hoe een statische sectie gewijzigd kan worden, valt buiten het bestek van deze standaard en wordt overgelaten aan de dataportals.

De parkeercapaciteit van een sectie lijkt op het eerste gezicht statisch. Toch is deze op advies van telinstanties niet opgenomen in de statische data. Wegwerkzaamheden en herindelingen van straten hebben te vaak invloed op de parkeercapaceit van straatsecties. In bewaakte stallingen kan het voorkomen dat bepaalde secties gedurende bepaalde periodes gereserveerd zijn.

#### `StaticData`

| Field    | Type            | Required | Description                       |
| -------- | --------------- | -------- | --------------------------------- |
| `result` | StaticSection[] | yes      | Verzameling van statische secties |
| {.data}  |

#### Survey Area

Een Onderzoeksgebied is een geometrische afbakening, waarmee individuele parkeerfaciliteiten samen geselecteerd kunnen worden voor verdere verwerking.
Bijvoorbeeld een stationsgebied, een strandboulevard of een uitgaansgebied.
De datastandaard biedt deze mogelijkheid zodat verschillende partijen dezelfde analyses zouden kunnen uitvoeren, op basis van de zelfde contouren van een gebied.
Niet elk onderzoek hoeft zijn Survey Area op te slaan, aangezien het platform geografische afbakeningen in een query ondersteunt.

Het is een hulpmiddel om parkeerlocaties in een bepaald gebied te selecteren.
Er is geen vaste relatie tussen onderzoeksgebieden en parkeerfaciliteiten: dat is puur een geografische relatie.
Ze zijn ook nuttig voor historische vergelijkingen.

| Field          | Type                  | Required | Description                                                                                                   |
| -------------- | --------------------- | -------- | ------------------------------------------------------------------------------------------------------------- |
| `id`           | string                | yes      | Een uuid, random of eventueel samengesteld                                                                    |
| `geoLocation`  | GeoJSON               | no       | Geografische afbakening van deze sectie volgens [[rfc7946]]. Kan gebruikt worden voor geo-zoekopdrachten.     |
| `validFrom`    | [[ISO8601]] timestamp | no       | Begin geldigheid                                                                                              |
| `validThrough` | [[ISO8601]] timestamp | no       | Einde geldigheid                                                                                              |
| `authority`    | OrganisationID        | no       | organisationId: Eigenaar van deze sectie. Alleen deze organistatie mag wijzigingen aanbrengen aan deze sectie |
| `altLabel`     | string[]              | no       | Alternatieve namen of IDs die de eigenaar of inwinner aan dit onderzoeksgebied geeft.                         |
| {.data}        |

#### Parking Facility

Een Parkeerlocatie is een plek waar voertuigen geparkeerd kunnen worden, zoals een fietsenstalling, een plein, een trottoir.
Een parkeerlocatie kan onderverdeeld worden in 1 of meerdere [secties](#sections).
De parkeerlocatie heeft altijd een geometrie, minimaal een punt op de kaart, maar bij voorkeur een contour.
Kenmerken van de locatie, zoals openingstijden of bewaakt/onbewaakt worden op dit niveau geregistreerd (dit wordt later uitgewerkt, in lijn met VeloPark en SPDP).

Als een van onderstaande kenmerken van de parkeerlocatie verandert, moet de eigenaar (`authority`) een nieuwe parkeerlocatie aanmaken.
Het veld `validThrough` wordt dan ingevuld om de voorganger ongeldig te maken.
Wijzigingen in de onderliggende secties betekenen dus expliciet niet dat de parkeerlocatie wijzigt.
Andersom betekent een nieuwe parkeerlocatie wel dat ook de secties nieuw aangemaakt moeten worden -- zij verwijzen immers naar de parkeerlocatie waar ze bijhoren.

Voor historische vergelijkingen, kan op basis van de geometrie en/of `altLabel` bepaald worden of het om (min of meer) dezelfde parkeerlocatie gaat.

| Field             | Type                  | Required    | Description                                                                                               |
| ----------------- | --------------------- | ----------- | --------------------------------------------------------------------------------------------------------- |
| `id`              | string                | yes         | Een uuid, random of eventueel samengesteld                                                                |
| `geoLocation`     | GeoJSON               | no          | Geografische afbakening van deze sectie volgens [[rfc7946]]. Kan gebruikt worden voor geo-zoekopdrachten. |
| `validFrom`       | [[ISO8601]] timestamp | no          | Vanaf dit tijdstip mag er dynamische data in deze sectie worden geschreven                                |
| `validThrough`    | [[ISO8601]] timestamp | no          | Tot dit tijdstip mag er dynamische data in deze sectie worden geschreven                                  |
| `altLabel`        | string[]              | conditional | Alternatieve namen of IDs van de eigenaar of inwinner.                                                    |
| `securityFeature` | `SecurityFeature[]`   | conditional | Beveiligingskenmerken                                                                                     |
| `allowedVehicles` | `Vehicle[]`           | conditional | Toegestane voertuigtypen voor deze parkeerlocatie.                                                        |
| `authority`       | `Organisation.id`     | no          | Alleen deze eigenaar mag wijzigingen aanbrengen aan deze parkeerlocatie.                                  |
| {.data}           |

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
Voordeel hiervan is dat de bezetting (zie dyanmische data) herleid kan worden tot een bepaald type parkeervoorziening.
En dat er specifiek voor bepaalde locaties logische keuzen kunnen worden gemaakt.

Er is een grote vrijheid in het indelen in secties van een parkeerlocatie.
Bij digitale systemen kan de onderverdeling die in het systeem gemaakt wordt, leidend zijn.
Bijvoorbeeld: wordt in een parkeerverwijssysteem onderscheid gemaakt tussen boven- en onderlaag in een etagerek,
dan bevelen we aan hiervoor aparte secties aan te maken.

Er is een vaste administratieve koppeling tussen sectie en de parkeerlocatie waar het deel van uitmaakt.
Daarom is ook een geometrie van een sectie niet noodzakelijk om aan te leveren.
Voor bijvoorbeeld handmatige tellers kan dat wel handig zijn.

| Field             | Type                  | Required | Description                                                                                                   |
| ----------------- | --------------------- | -------- | ------------------------------------------------------------------------------------------------------------- |
| `id`              | string                | yes      | Een uuid, random of eventueel samengesteld                                                                    |
| `parkingFacility` | `ParkingFacility.id`  | yes      | ID die verwijst naar de parkeerlocatie waar het deel van uitmaakt.                                            |
| `altLabel`        | string                | no       | Private id die een contractor hanteert deze sectie                                                            |
| `geoLocation`     | GeoJSON               | no       | Geografische afbakening van deze sectie volgens [[rfc7946]]. Kan gebruikt worden voor geo-zoekopdrachten.     |
| `spaces`          | `Space[]`             | no       | De types parkeervoorziening.                                                                                  |
| `validFrom`       | [[ISO8601]] timestamp | no       | Vanaf dit tijdstip mag er dynamische data in deze sectie worden geschreven                                    |
| `validThrough`    | [[ISO8601]] timestamp | no       | Tot dit tijdstip mag er dynamische data in deze sectie worden geschreven                                      |
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

> **Implementatiespecifiek**:
> Totalen van de parkeerlocatie kunnen dan met de updates vooraf berekend worden of pas bij uitlevering.

#### `DynamicData`

| Field    | Type             | Required | Description                           |
| -------- | ---------------- | -------- | ------------------------------------- |
| `result` | DynamicSection[] | yes      | Verzameling secties met stallingsdata |
| {.data}  |

#### `DynamicParkingFacility`

| Field                      | Type                  | Required    | Description                                                           |
| -------------------------- | --------------------- | ----------- | --------------------------------------------------------------------- |
| `parkingFacility`          | `ParkingFacility.id`  | yes         | Parkeerlocatie waar deze meting over gaat.                            |
| `timestamp`                | [[ISO8601]] timestamp | conditional | Tijdstip van de meting. Alleen verplicht in de stam van de sectieboom |
| `surveyId`                 | string                | conditional | Id van de survey waartoe deze meting behoort                          |
| `parkingCapacity`          | number                | conditional | Totaal aantal plekken, verplicht in de bladeren van de sectieboom     |
| `parkingCapacityTimestamp` | [[ISO8601]] timestamp | no          | Tijdstip van meting aantal plekken                                    |
| `vacantSpaces`             | number                | conditional | Aantal vrije plekken, verplicht in de bladeren van de sectieboom      |
| `occupiedSpaces`           | number                | conditional | Aantal bezette plekken, verplicht in de bladeren van de sectieboom    |
| `count`                    | Count[]               | no          | Verzameling van Count-objecten                                        |
| `space`                    | Space                 | no          | Alleen toegestaan in de bladeren van de sectieboom                    |
| `note`                     | Note                  | no          | Notities over de meting in deze sectie                                |
| {.data}                    |

#### `DynamicSection`

Een telling of meting kan gaan over het aantal parkeerplekken (evt. naar type) en/of
het aantal geparkeerde voertuigen (evt. naar type) in een sectie.

| Field                 | Type                    | Required    | Description                                                                                        |
| --------------------- | ----------------------- | ----------- | -------------------------------------------------------------------------------------------------- |
| `section`             | `Section.id`            | yes         | Sectie waarop deze telling/meting betrekking heeft.                                                |
| `timestamp`           | [[ISO8601]] timestamp   | yes         | Tijdstip van de meting                                                                             |
| `surveyId`            | string                  | yes         | Id van de survey waartoe deze meting behoort                                                       |
| `parkingCapacity`     | number                  | conditional | Totaal aantal plekken, verplicht bij capaciteitstelling.                                           |
| `totalParked`         | number                  | conditional | Totaal aantal geparkeerde voertuigen, verplicht bij telling van het aantal geparkeerde voertuigen. |
| `capacityBySpaceType` | `CapacityBySpaceType[]` | no          | Capaciteit per type parkeervoorziening                                                             |
| `parkedByVehicleType` | `VehicleTypeCount[]`    | no          | Telling per geparkeerd voertuigtype                                                                |
| `occupiedSpaces`      | number                  | no          | Aantal bezette plekken, berekend o.b.v. `capacityBySpaceType` en `parkedByVehicleType`             |
| {.data}               |

De velden surveyId, authorityId en contractorId kunnen gebruikt worden bij het filteren van data bij de zoekopdrachten van de API's 4 en 5.
Je zou bijvoorbeeld kunnen alle metingen van een bepaald onderzoek die zijn uitgevoerd door een bepaalde contractor kunnen opvragen. Zie _API Requests_ voor meer details over zoekopdrachten.

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

| Field         | Type           | Required | Description                  | ProRail |
| ------------- | -------------- | -------- | ---------------------------- | ------- |
| `type`        | string         | no       | Zie tabel Vehicle.type       | \*      |
| `propulsion`  | string[]       | no       | Zie tabel Vehicle.propulsion |
| `appearance`  | string         | no       | Zie tabel Vehicle.appearance | \*      |
| `state`       | VehicleState[] | no       | Zie tabel VehicleState       |
| `accessoires` | Accessoire[]   | no       | Zie tabel Accessoire         | \*      |
| `owner`       | string         | no       | Zie tabel Vehicle.owner      | \*      |
| {.data}       |

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

- Fiets = `{ type: 'f', owner: 'p' }`
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

</aside>
