# Dataschema’s

Deze datastandaard definieert een aantal objecttypen die betrekking hebben op fietsparkeervoorzieningen en tellingen van gestalde fietsen.
Het omvat definities, gebruik en API’s voor:

1. Uitgebreide beschrijvingen van fietstypen ({{Vehicle}}).
2. Fietsenstallingen en fietsparkeervoorzieningen ({{ParkingFacility}}), ook op detailniveau ({{Section}});
3. Tellingen binnen die (deel)voorzieningen ({{DynamicParkingFacility}} en {{DynamicSection}});
4. Vastgestelde onderzoeks(deel)gebieden ({{SurveyAreas}}) en tellingen ({{Survey}});

<figure>
<img src="docs/objecten.drawio.svg">
<figcaption>Overzicht relaties tussen statische en dynamische gegevenselementen</figcaption>
</figure>

## Onderzoeken en inwinningen

Een {{Survey}} groepeert een bron: het representeert een onderzoek of inwinning, incidenteel of doorlopend, in opdracht van een bepaalde {{Organisation}}.
Daarbij MOGEN één of meerdere onderzoeksgebieden, {{SurveyAreas}}, worden meegegeven: een geografische afbakening van een gebied, dat voor bepaalde rapportages of inzichten nuttig is.
Daarvoor worden dan overlappende {{ParkingFacilities}} meegenomen: een SurveyArea is niet een telgebied zelf.

Een implementatie MAG de inzender ‘eigenaar’ laten worden van een Survey:
dat houdt in dat andere inzenders NIET gegevens aan dat Survey MOGEN koppelen.

_De rest van deze sectie is niet normatief._

Gegevens over het onderzoek —
geïnitieerd door wie?
Uitgevoerd door wie?
Wanneer?
Waar?
—
kan worden ingestuurd, maar is niet verplicht.

### <dfn>`Survey`

Een Survey is een onderzoek of inwinning die incidenteel of doorlopend in opdracht van een bepaalde opdrachtgever wordt uitgevoerd door een of meerdere aannemers.
Het vormt de bron van de meeste datasoorten in de datastandaard.

Een Survey

| Eigenschap                                                | Type                     | Kardinaliteit | Beschrijving                                                                |
| --------------------------------------------------------- | ------------------------ | ------------- | --------------------------------------------------------------------------- |
| <dfn data-dfn-for="Survey">id                             | `string`                 | 0..1          | Een [[=ResourceIdentifier=]].                                               |
| <dfn data-dfn-for="Survey">name                           | `string`                 | 1             | Naam van het onderzoek.                                                     |
| <dfn data-dfn-for="Survey">authority                      | `string`                 | 1             | {{Organisation.id}} van de opdrachtgever.                                   |
| <dfn data-dfn-for="Survey">contractors                    | `string[]`               | 1..N          | {{Organisation.id}} van de uitvoerder.                                      |
| <dfn data-dfn-for="Survey">license                        | `string`                 | 0..1          | Licentie voor het gebruik van de data (vormvrij).                           |
| <dfn data-dfn-for="Survey">surveyArea                     | `string[]`               | 0..N          | De {{SurveyArea.id}} van de onderzoeksgebieden die dit onderzoek aandraagt. |
| <dfn data-dfn-for="Survey">distinguishesVehicleCategories | {{CanonicalVehicle}}`[]` | 0..N          | De voertuigtypes die voor dit onderzoek worden onderscheiden in tellingen.  |
| {.data def }                                              |

Opdrachtgever en uitvoerder zijn constant over de levensduur van de Survey.
De datastandaard stelt geen eisen aan de vervanging van een Survey.

### <dfn>`Organisation`

Een Organisation representeert een opdrachtgever of een uitvoerende instantie.

| Eigenschap                            | Type     | Kardinaliteit | Beschrijving                  |
| ------------------------------------- | -------- | ------------- | ----------------------------- |
| <dfn data-dfn-for="Organisation">id   | `string` | 0..1          | Een [[=ResourceIdentifier=]]. |
| <dfn data-dfn-for="Organisation">name | `string` | 1             | Naam van de instantie         |
| {.data def}                           |

### <dfn>`SurveyArea`

Een Onderzoeksgebied is een geometrische afbakening, waarmee individuele stallingsvoorzieningen samen geselecteerd kunnen worden voor verdere verwerking.
Bijvoorbeeld een stationsgebied, een strandboulevard of een uitgaansgebied.
De datastandaard biedt deze mogelijkheid zodat verschillende partijen dezelfde analyses zouden kunnen uitvoeren, op basis van dezelfde contouren van een gebied.
Ook voor historische vergelijkingen kunnen SurveyAreas worden gebruikt.

<aside class=note>
Geografische zoekopdrachten zijn ook mogelijk in de <a href='#rest-api'></a>, zonder dat er van SurveyAreas gebruik wordt gemaakt.
</aside>

Het is een hulpmiddel om {{ParkingFacilities}} in een bepaald gebied te selecteren.
Er is geen geadminstreerde relatie tussen onderzoeksgebieden en stallingsvoorzieningen: dat is puur een geografische relatie.

| Eigenschap                                   | Type                             | Kardinaliteit | Beschrijving                                                                                           |
| -------------------------------------------- | -------------------------------- | ------------- | ------------------------------------------------------------------------------------------------------ |
| <dfn data-dfn-for="SurveyArea">id            | `string`                         | 0..1          | Een [[=ResourceIdentifier=]].                                                                          |
| <dfn data-dfn-for="SurveyArea">geoLocation   | GeoJSON                          | 1..1          | Geografische afbakening volgens [[rfc7946]].                                                           |
| <dfn data-dfn-for="SurveyArea">parent        | {{SurveyArea.id}} (`string`)     | 0..1          | Verwijzing naar een onderzoeksgebied van een hogerliggende orde.                                       |
| <dfn data-dfn-for="SurveyArea">validFrom     | [[rfc3339]] date-time (`string`) | 0..1          | Begin geldigheid                                                                                       |
| <dfn data-dfn-for="SurveyArea">validThrough  | [[rfc3339]] date-time (`string`) | 0..1          | Einde geldigheid                                                                                       |
| <dfn data-dfn-for="SurveyArea">authority     | {{Organisation.id}} (`string`)   | 1..1          | Eigenaar van dit onderzoeksgebied. Alleen deze organisatie mag wijzigingen aanbrengen aan deze sectie. |
| <dfn data-dfn-for="SurveyArea">name          | `string[]`                       | 0..N          | Naam die de eigenaar of inwinner aan dit onderzoeksgebied geeft.                                       |
| <dfn data-dfn-for="SurveyArea">alternateName | `string[]`                       | 0..N          | Alternatieve namen of IDs die de eigenaar of inwinner aan dit onderzoeksgebied geeft.                  |
| {.data def}                                  |                                  |               |

## Stallingen, parkeervoorzieningen

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

### <dfn>`ParkingFacility`

Een ParkingFacility is elke plek waar voertuigen geparkeerd kunnen worden, bedoeld of onbedoeld: zoals een fietsenstalling, op het maaiveld, op een plein of een trottoir.
Een ParkingFacility MAG onderverdeeld worden in één of meerdere {{Sections}}.
Een ParkingFacility MOET een geometrie hebben, minimaal een punt op de kaart, maar bij voorkeur een vlakcontour.

Verdere kenmerken van de locatie, zoals openingstijden of bewaakt/onbewaakt worden op dit niveau geregistreerd en worden later verder uitgewerkt.

Wijzigingen in de onderliggende secties betekenen expliciet niet dat de parkeerlocatie wijzigt.
Andersom betekent een nieuwe parkeerlocatie _wel_ dat ook de secties nieuw aangemaakt moeten worden -- zij verwijzen immers naar de parkeerlocatie waar ze toe behoren.

Voor historische vergelijkingen kan op basis van de geometrie en/of `alternateName` bepaald worden of het om (min of meer) dezelfde parkeerlocatie gaat.

| Eigenschap                                          | Type                             | Kardinaliteit | Beschrijving                                                             |
| --------------------------------------------------- | -------------------------------- | ------------- | ------------------------------------------------------------------------ |
| <dfn data-dfn-for="ParkingFacility">id              | `string`                         | 1             | Een [[=ResourceIdentifier=]].                                            |
| <dfn data-dfn-for="ParkingFacility">geoLocation     | GeoJSON                          | 1..1          | Geografische afbakening volgens [[rfc7946]].                             |
| <dfn data-dfn-for="ParkingFacility">name            | `string`                         | 0..1          | Namen voor de voorziening.                                               |
| <dfn data-dfn-for="ParkingFacility">alternateName   | `string[]`                       | 0..N          | Alternatieve namen of IDs van de eigenaar of inwinner.                   |
| <dfn data-dfn-for="ParkingFacility">securityFeature | {{SecurityFeature}}`[]`          | 0..N          | Beveiligingskenmerken                                                    |
| <dfn data-dfn-for="ParkingFacility">allows          | {{Vehicle}}`[]`                  | 1..N          | Toegestane voertuigtypen voor deze parkeerlocatie.                       |
| <dfn data-dfn-for="ParkingFacility">validFrom       | [[rfc3339]] date-time (`string`) | 0..1          | Zie <a href='#geldigheid-door-de-tijd'></a>.                             |
| <dfn data-dfn-for="ParkingFacility">validThrough    | [[rfc3339]] date-time (`string`) | 0..1          | Zie <a href='#geldigheid-door-de-tijd'></a>.                             |
| ~~<dfn data-dfn-for="ParkingFacility">authority~~   | {{Organisation.id}}              | 0..1          | Alleen deze eigenaar mag wijzigingen aanbrengen aan deze parkeerlocatie. |
| {.data def}                                         |

#### Enum <dfn>`SecurityFeature`

| Enum                   | Beschrijving                                  |
| ---------------------- | --------------------------------------------- |
| `CameraSurveillance`   | Camerabewaking aanwezig.                      |
| `LockerService`        | Fietskluizen aanwezig.                        |
| `PersonnelSupervision` | De stalling is bemenst.                       |
| `ElectronicAccess`     | Een elektronisch toegangssysteem is aanwezig. |
| {.data def}            |

### <dfn>`Section`

Een {{ParkingFacility}} bestaat uit 1 of meerdere Sections.
Binnen een sectie komt bij voorkeur één type parkeervoorziening voor: bijv. alleen etagerekken of alleen plekken voor buitenmodelfietsen.
Voordeel hiervan is dat de bezetting (zie <a href='#tellingen-metingen-en-capaciteit'></a>) herleid kan worden tot een bepaald type parkeervoorziening.

Er is een grote vrijheid in het indelen in secties van een ParkingFacility.
Bij digitale systemen kan de onderverdeling die in het systeem gemaakt wordt, leidend zijn.
Bijvoorbeeld: wordt in een parkeerverwijssysteem onderscheid gemaakt tussen boven- en onderlaag in een etagerek,
dan bevelen we aan hiervoor aparte secties aan te maken.

Er is een vaste administratieve koppeling tussen Section en de ParkingFacility waar het deel van uitmaakt.
Daarom is ook een geometrie van een sectie niet noodzakelijk om aan te leveren.
Voor bijvoorbeeld handmatige tellers kan dat wel handig zijn.

| Eigenschap                                  | Type                              | Kardinaliteit | Beschrijving                                                                                                               | ProRail |
| ------------------------------------------- | --------------------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------- | ------- |
| <dfn data-dfn-for='Section'>id              | `string`                          | 0..1          | Een [[=ResourceIdentifier=]].                                                                                              |
| <dfn data-dfn-for='Section'>parkingFacility | {{ParkingFacility.id}} (`string`) | 1             | Koppeling naar de ParkingFacility waar de sectie deel van uitmaakt.                                                        |
| <dfn data-dfn-for="Section">name            | `string`                          | 0..1          | Namen voor de voorziening.                                                                                                 |
| <dfn data-dfn-for="Section">alternateName   | `string[]`                        | 0..N          | Alternatieve namen of IDs van de eigenaar of inwinner.                                                                     |
| <dfn data-dfn-for='Section'>geoLocation     | GeoJSON                           | 0..1          | Geografische afbakening volgens [[rfc7946]].                                                                               |
| <dfn data-dfn-for='Section'>parkingSpaceOf  | {{ParkingSpaceOf}}`[]`            | 1..N          | De types parkeervoorziening.                                                                                               |
| <dfn data-dfn-for='Section'>level           | `number`                          | 0..1          | De etage in de ParkingFacility waar deze sectie zich bevindt. -1 = onder maaiveld, 0 = maaiveld (default), 1 = verdieping. |
| <dfn data-dfn-for='Section'>validFrom       | [[rfc3339]] date-time (`string`)  | 0..1          | Begin geldigheid. Zie <a href='#geldigheid-door-de-tijd'></a>.                                                             |
| <dfn data-dfn-for='Section'>validThrough    | [[rfc3339]] date-time (`string`)  | 0..1          | Einde geldigheid. Zie <a href='#geldigheid-door-de-tijd'></a>.                                                             |
| <dfn data-dfn-for='Section'>authority       | {{Organisation.id}} (`string`)    | 0..1          | Eigenaar van deze sectie.                                                                                                  |
| {.data def}                                 |

Een sectie kan een kortere `validThrough` hebben dan de parkeerlocatie waartoe het behoort:
als bijvoorbeeld een rek vervangen wordt door een buitenmodelvak.
De eigenaar van de sectie (`authority`) is verantwoordelijk hiervoor.
Bij handmatige tellingen is het goed denkbaar dat een veldwerker een extra sectie aanmaakt, die vervolgens goedgekeurd moet worden door de eigenaar.

### <dfn>`ParkingSpaceOf`

Omdat niet individuele parkeerplekken gemodelleerd worden, representeert ParkingSpaceOf een homogene groep parkeerplekken.
De homogeniteit kan op basis van type parkeervoorziening ({{ParkingSpaceType}}) zijn, op basis van het voertuigtype waarvoor het bedoeld is ({{Vehicle}}) of een combinatie van beide.

| Eigenschap                                  | Type            | Kardinaliteit | Beschrijving                                                                                 | ProRail |
| ------------------------------------------- | --------------- | ------------- | -------------------------------------------------------------------------------------------- | ------- |
| <dfn data-dfn-for='ParkingSpaceOf'>type     | `string`        | 0..1          | Type parkeervoorziening, zie enum {{ParkingSpaceType}}.                                      | \*      |
| <dfn data-dfn-for='ParkingSpaceOf'>vehicles | {{Vehicle}}`[]` | 0..N          | Deze parkeervoorziening is uitsluitend voor voertuigen voldoend aan de genoemde beperkingen. | \*      |
| {.data def}                                 |

Minstens één van {{ParkingSpaceOf.type}} of {{ParkingSpaceOf.vehicles}} MOET voorkomen bij een ParkingSpaceOf.

#### Enum <dfn>`ParkingSpaceType`

| Enum    | Beschrijving      | ProRail |
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

### <dfn>`ParkingSpace`

Individuele parkeerplekken zijn voorzien als een toekomstige uitbreiding.

## Tellingen, metingen en capaciteit

Dynamische data zijn de tellingen of metingen van het aantal plekken en/of het aantal geparkeerde voertuigen.
Dynamische data kan per sectie aangeleverd worden en per parkeerlocatie of per sectie uitgeleverd worden.
Teldata van de secties van één parkeerlocatie moeten dan wel op hetzelfde moment geüpdatet worden.

<aside class='note' title='Stallingscapaciteit'>

De parkeercapaciteit van een sectie lijkt op het eerste gezicht statisch.
Toch is deze op advies van telinstanties niet opgenomen in de statische data.
Wegwerkzaamheden en herindelingen van straten hebben te vaak invloed op de parkeercapaceit van straatsecties.
In bewaakte stallingen kan het voorkomen dat bepaalde secties gedurende bepaalde periodes gereserveerd zijn.

</aside>

<aside class='issue'>

De naamgeving van de DynamicX-elementen kan anders, aangezien de oppositie dynamisch-statisch elders niet meer voorkomt.
Tegelijk is het verschil tussen `DynamicParkingFacility`, `DynamicSection` minimaal.
Volstaat wellicht een algemeen `Measurement`?

</aside>

### <dfn>`DynamicParkingFacility`

Deze telling of meting registreert het aantal geparkeerde voertuigen en/of de capaciteit daarvoor.
In onderstaande tabel staat Kard.<sub>capaciteit</sub> voor de kardinaliteit van de eigenschappen van een capaciteitstelling;
Kard.<sub>bezetting</sub> staat voor de kardinaliteit van de eigenschappen bij een bezettingstelling.

| Eigenschap                                                           | Type                              | Kard.<sub>bezetting</sub> | Kard.<sub>capaciteit</sub> | Beschrijving                                                                            |
| -------------------------------------------------------------------- | --------------------------------- | ------------------------- | -------------------------- | --------------------------------------------------------------------------------------- |
| <dfn data-dfn-for='DynamicParkingFacility'>parkingFacility           | `string`                          | 1                         | 1                          | {{ParkingFacility.id}} waarop deze telling betrekking heeft.                            |
| <dfn data-dfn-for='DynamicParkingFacility'>survey                    | `string`                          | 1                         | 1                          | {{Survey.id}} waartoe deze meting behoort.                                              |
| <dfn data-dfn-for='DynamicParkingFacility'>timestamp                 | [[rfc3339]] date-time (`string`)  | 1                         | 1                          | Tijdstip van de meting, zie <a href='#geldigheid-door-de-tijd'></a>.                    |
| <dfn data-dfn-for='DynamicParkingFacility'>note                      | {{Note}}                          | 0..1                      | 0..1                       | Notities over de meting in deze sectie                                                  |
| <dfn data-dfn-for='DynamicParkingFacility'>parkingCapacity           | `number`                          | 0                         | 1                          | Totaal aantal plekken, verplicht als één of meerdere onderdelen dit gemeten hebben.     |
| <dfn data-dfn-for='DynamicParkingFacility'>capacityPerParkingSpaceOf | {{CapacityPerParkingSpaceOf}}`[]` | 0                         | 1..N                       | Capaciteit per type parkeervoorziening, over de hele stalling. Sommering bij indienen.  |
| <dfn data-dfn-for='DynamicParkingFacility'>totalParked               | `number`                          | 1                         | 0                          | Aantal getelde voertuigen, verplicht als één of meerdere onderdelen dit gemeten hebben. |
| <dfn data-dfn-for='DynamicParkingFacility'>count                     | {{VehicleTypeCount}}`[]`          | 1..N                      | 0                          | Verzameling van tellingen.                                                              |
| <dfn data-dfn-for='DynamicParkingFacility'>vacantSpaces              | `number`                          | 0..1                      | 0                          | Aantal vrije plekken.                                                                   |
| <dfn data-dfn-for='DynamicParkingFacility'>occupiedSpaces            | `number`                          | 0..1                      | 0                          | Aantal bezette plekken, verplicht als één of meerdere onderdelen dit gemeten hebben.    |
| {.data def}                                                          |

<aside class='issue'>
{{DynamicParkingFacility.parkingCapacity}} en {{DynamicParkingFacility.capacityPerParkingSpaceOf}}:
  deze zijn 'optioneel', omdat het meteen samen doorgegeven kan worden.
</aside>

<aside class='issue' data-dfn-for='DynamicParkingFacility'>
{{DynamicParkingFacility.vacantSpaces}} en {{DynamicParkingFacility.occupiedSpaces}}: 
  is dit een koppeling met de capaciteit? 
  Alsdan, hoe zit het met verouderde capaciteitsmetingen dan?
Redmer: kunnen we alleen {{DynamicParkingFacility.totalParked}} verplicht maken?
Otto: Of duidelijk maken dat deze waardes afgeleid zijn van een capaciteitsmeting?

Redmer: of {{vacantSpaces}} en {{occupiedSpaces}} uit elkaar trekken: één berekend, één gemeten.

Redmer: of de API het laten berekenen.

</aside>

### <dfn>`DynamicSection`

Een telling of meting kan gaan over het aantal parkeerplekken (evt. naar type) en/of
het aantal geparkeerde voertuigen (evt. naar type) in een sectie.

| Eigenschap                                                | Type                             | Kardinaliteit | Beschrijving                                                                                       | ProRail |
| --------------------------------------------------------- | -------------------------------- | ------------- | -------------------------------------------------------------------------------------------------- | ------- |
| <dfn data-dfn-for='DynamicSection'>section                | `string`                         | 1             | {{Section.id}} waarop deze telling betrekking heeft.                                               |
| <dfn data-dfn-for='DynamicSection'>dynamicParkingFacility | `string`                         | 1             | {{DynamicParkingFacility.id}} waarvan deze sectie deel uitmaakt.                                   |
| <dfn data-dfn-for='DynamicSection'>timestamp              | [[rfc3339]] date-time (`string`) | 1             | Tijdstip van de meting, zie <a href='#geldigheid-door-de-tijd'></a>.                               |
| <dfn data-dfn-for='DynamicSection'>survey                 | `string`                         | 1             | {{Survey.id}} waartoe deze meting behoort.                                                         |
| <dfn data-dfn-for='DynamicSection'>parkingCapacity        | `number`                         | 0..1          | Totaal aantal plekken, verplicht bij capaciteitstelling.                                           |
| <dfn data-dfn-for='DynamicSection'>totalParked            | `number`                         | 0..1          | Totaal aantal geparkeerde voertuigen, verplicht bij telling van het aantal geparkeerde voertuigen. |
| <dfn data-dfn-for='DynamicSection'>parkedByVehicleType    | {{VehicleTypeCount}}`[]`         | 0..N          | Telling per geparkeerd voertuigtype                                                                |
| <dfn data-dfn-for='DynamicSection'>occupiedSpaces         | `number`                         | 0..1          | Aantal bezette plekken, berekend o.b.v. `capacityPerParkingSpaceOf` en `parkedByVehicleType`       |
| {.data def}                                               |

Hierbij geldt het volgende:

- Bij een capaciteitstelling is het veld {{DynamicSection.parkingCapacity}} VERPLICHT.
- Bij een telling van het aantal geparkeerde voertuigen is het veld {{DynamicSection.totalParked}} VERPLICHT.

_De rest van deze sectie is niet normatief._

~~De velden surveyId, authorityId en contractorId kunnen gebruikt worden bij het filteren van data bij de zoekopdrachten van de API's 4 en 5.
Je zou bijvoorbeeld kunnen alle metingen van een bepaald onderzoek die zijn uitgevoerd door een bepaalde contractor kunnen opvragen. Zie _API Requests_ voor meer details over zoekopdrachten.~~

### <dfn>`DynamicPlace`

Dit is voor later voorzien.

### <dfn>`CapacityPerParkingSpaceOf`

| Eigenschap                                                     | Type               | Kardinaliteit | Beschrijving        | ProRail |
| -------------------------------------------------------------- | ------------------ | ------------- | ------------------- | ------- |
| <dfn data-dfn-for='CapacityPerParkingSpaceOf'>parkingSpaceOf   | {{ParkingSpaceOf}} | 1             | Type parkeerplaats. | \*      |
| <dfn data-dfn-for='CapacityPerParkingSpaceOf'>numberOfVehicles | `number`           | 1             | Gemeten capaciteit. | \*      |
| {.data def}                                                    |

<aside class='example'>

1. Etagerek-onder met capaciteit voor 10 huurfietsen:

   ```json
   {
     "numberOfVehicles": 10,
     "type": "o",
     "vehicles": [{ "type": "f", "owner": "h" }]
   }
   ```

2. Vak voor 15 deelsnor- of -bromfietsen:

   ```json
   {
     "numberOfVehicles": 15,
     "type": "v",
     "vehicles": [{ "type": "sb", "owner": "h" }]
   }
   ```

</aside>

### <dfn>`VehicleTypeCount`

| Eigenschap                                            | Type        | Kardinaliteit | Beschrijving                             | ProRail |
| ----------------------------------------------------- | ----------- | ------------- | ---------------------------------------- | ------- |
| <dfn data-dfn-for="VehicleTypeCount">vehicle          | {{Vehicle}} | 1             | Voertuigtype                             | \*      |
| <dfn data-dfn-for="VehicleTypeCount">parkState        | `string`    | 0..1          | Zie enum {{VehicleParkState}}.           | \*      |
| <dfn data-dfn-for="VehicleTypeCount">numberOfVehicles | `number`    | 1             | Aantal gestalde voertuigen van dit type. | \*      |
| {.data def}                                           |

<aside class='example'>

1. Zes particuliere fietsen in het rek:

   ```json
   {
     "vehicle": { "type": "f", "owner": "p" },
     "numberOfVehicles": 6,
     "parkState": "i"
   }
   ```

2. Vier particuliere fietsen buiten het rek:

   ```json
   {
     "vehicle": { "type": "f", "owner": "p" },
     "numberOfVehicles": 4,
     "parkState": "x"
   }
   ```

3. Honderd normale fietsen buiten voorziening:

   ```json
   {
     "vehicle": { "type": "f" },
     "numberOfVehicles": 100,
     "parkState": "x"
   }
   ```

</aside>

#### Enum <dfn>`VehicleParkState`

| Enum    | Omschrijving                  | ProRail |
| ------- | ----------------------------- | ------- |
| `i`     | In voorziening                |         |
| `j`     | In voorziening, juist gestald | \*      |
| `k`     | In voorziening, neemt plek in | \*      |
| `p`     | nabij voorziening             | \*      |
| `x`     | buiten voorziening            | \*      |
| {.data} |

“In voorziening” betekent twee dingen, wat voor bepaalde {{Survey}}s geëxpliciteerd moet worden:

- juist gestald in de voorziening;
- neemt 1 plek in de voorziening.

### <dfn>`Note`

Het objecttype Note houdt bijzonderheden over de telling bij.
Zoals of het een feestdag was op moment van telling.

| Eigenschap                                    | Type      | Kardinaliteit | Beschrijving                                         |
| --------------------------------------------- | --------- | ------------- | ---------------------------------------------------- |
| <dfn data-dfn-for='Note'>wasClosed            | `boolean` | 0..1          | Was deze voorziening niet geopend tijdens de meting. |
| <dfn data-dfn-for='Note'>wasHoliday           | `boolean` | 0..1          | Was er een vakantie tijdens de meting.               |
| <dfn data-dfn-for='Note'>wasEvent             | `boolean` | 0..1          | Was er een evenement of feestdag tijdens de meting.  |
| <dfn data-dfn-for='Note'>wasUnderConstruction | `boolean` | 0..1          | Waren er werkzaamheden tijdens de meting.            |
| <dfn data-dfn-for='Note'>remark               | `string`  | 0..1          | Vrij tekstveld.                                      |
| {.data def}                                   |

## Voertuigen

Met {{Vehicle}} kunnen uitgebreid kenmerken van fietsen, bromfietsen en andere voertuigen worden beschreven.
Met {{CanonicalVehicle}} worden prototypische voertuigen voor een telling geconcretiseerd.

_De rest van deze sectie is niet normatief._

{{CanonicalVehicles}} worden gebruikt bij concrete opdrachten met een beperkte tel-resolutie:
bijvoorbeeld als elk voertuig maar in één van vier categorieën wordt geteld:
brom-/snorfiets,
fiets met accessory,
sterk afwijkend buitenmodelfiets,
gewone fiets.
Door bij {{Survey}} te annoteren welke {{CanonicalVehicles}} zijn gebruikt, is te achterhalen welke uitspraken over voertuigtypen gestaafd worden door de data.

### <dfn>`Vehicle`

<div class='issue' data-number="9"></div>

| Eigenschappen                           | Type              | Kardinaliteit | Beschrijving                       | ProRail |
| --------------------------------------- | ----------------- | ------------- | ---------------------------------- | ------- |
| <dfn data-dfn-for='Vehicle'>type        | `string`          | 0..1          | Zie enum {{VehicleType}}           | \*      |
| <dfn data-dfn-for='Vehicle'>propulsion  | `string[]`        | 0..N          | Zie enum {{VehiclePropulsionType}} |
| <dfn data-dfn-for='Vehicle'>appearance  | `string`          | 0..1          | Zie enum {{VehicleAppearanceType}} | \*      |
| <dfn data-dfn-for='Vehicle'>state       | `string[]`        | 0..N          | Zie enum {{VehicleStateType}}      |
| <dfn data-dfn-for='Vehicle'>accessories | {{Accessory}}`[]` | 0..N          | Zie tabel {{Accessory}}            | \*      |
| <dfn data-dfn-for='Vehicle'>owner       | `string`          | 0..1          | Zie enum {{VehicleOwnerType}}      | \*      |
| {.data def}                             |

### <dfn>`CanonicalVehicle`

Alternatief voor expliciete voertuigtyperingen.

| Eigenschappen                                | Type            | Kardinaliteit | Beschrijving                   | ProRail |
| -------------------------------------------- | --------------- | ------------- | ------------------------------ | ------- |
| <dfn data-dfn-for='CanonicalVehicle'>label   | `string`        | 0..1          | Leesbare naam van voertuigtype |         |
| <dfn data-dfn-for='CanonicalVehicle'>vehicle | {{Vehicle}}`[]` | 1..N          | Zie tabel {{Vehicle}}          |
| {.data def}                                  |

<aside class="example" title="ProRail: Canonieke voertuigen">

Eigenschappen die niet vermeld worden bij canonieke voertuigen, zijn niet bepalend geweest voor de categorie-indeling.
Dat betekent dus dat bij bijv. onderstaande _Normfiets_, het best een elektrische trapondersteuning (`propulsion = "s"`) of een fietstas (`accessory.type = "t"`) kan hebben.
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
      "accessories": [{ "type": "k", "position": "v" }],
      "owner": "p"
    },
    {
      // kinderzitje achter
      "type": "f",
      "accessories": [{ "type": "z", "position": "a" }],
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

### <dfn>`Accessory`

| Eigenschappen                          | Type     | Kardinaliteit | Beschrijving                   | ProRail |
| -------------------------------------- | -------- | ------------- | ------------------------------ | ------- |
| <dfn data-dfn-for='Accessory'>type     | `string` | 0..1          | Zie enum {{AccessoryType}}     | \*      |
| <dfn data-dfn-for='Accessory'>position | `string` | 0..1          | Zie enum {{AttributePosition}} |
| {.data def}                            |

#### Enum <dfn>`AccessoryType`

| Enum    | Omschrijving               | ProRail |
| ------- | -------------------------- | ------- |
| `z`     | kinderzitje                |
| `t`     | fietstas                   |
| `b`     | bagagedrager               |
| `k`     | krat/mand                  |
| `p`     | Beperkt afwijkend, nl. k,z | \*      |
| {.data} |

#### Enum <dfn>`AttributePosition`

| Enum    | Omschrijving |
| ------- | ------------ |
| `v`     | voor         |
| `a`     | achter       |
| {.data} |

### <dfn>`VehicleState`

| Eigenschap  | Type     | Kardinaliteit | Beschrijving                   |
| ----------- | -------- | ------------- | ------------------------------ |
| `type`      | `string` | 0..1          | Zie enum {{VehicleStateType}}  |
| `position`  | `string` | 0..1          | Zie enum {{AttributePosition}} |
| {.data def} |

##### Enum <dfn>`VehicleStateType`

| Enum    | Omschrijving |
| ------- | ------------ |
| `w`     | wrak         |
| `l`     | lekke band   |
| `z`     | zonder zadel |
| ...     | ...          |
| {.data} |

#### Enum <dfn>`VehicleType`

Classificatie naar wettelijke voertuigcategorie.

| Enum    | Voertuigtype          | Omschrijving                                                                | ProRail |
| ------- | --------------------- | --------------------------------------------------------------------------- | ------- |
| `f`     | fiets                 | Geen kenteken en geen verzekeringsplaatje                                   | \*      |
| `s`     | snorfiets             | kentekenplaat is blauw, met een wit kader, wit opschrift en een hologram    | \*      |
| `b`     | bromfiets             | kentekenplaat is geel, met een zwart kader, zwart opschrift en een hologram | \*      |
| `s b`   | snor- of bromfiets    | kentekenplaat is geel of blauw, onderscheid wordt niet gemaakt.             | \*      |
| `m`     | motorfiets            | NL geel, internationaal anders                                              |
| `g`     | gehandicaptenvoertuig | driewieler, scootmobiel, rolstoel, etc                                      |
| `a`     | anders                |                                                                             |
| {.data} |

#### Enum <dfn>`VehiclePropulsionType`

| Enum    | Aandrijving            | Omschrijving                          |
| ------- | ---------------------- | ------------------------------------- |
| `s`     | Spierkracht            | bv traditionele fiets of voetganger   |
| `e`     | Elektrisch             | bv e-bike                             |
| `s`     | Elektrisch ondersteund | bv e-bike                             |
| `b`     | Verbrandingsmotor      | bv traditionele bromfiets, motorfiets |
| {.data} |

#### Enum <dfn>`VehicleAppearanceType`

| Enum    | Verschijningsvorm              | ProRail |
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

#### Enum <dfn>`VehicleOwnerType`

| Enum    | Eigenaar | Omschrijving                | ProRail |
| ------- | -------- | --------------------------- | ------- |
| `p`     | Privé    | Privéfiets                  | (std.)  |
| `l`     | Lease    | Leasefiets, zoals Swapfiets |
| `h`     | Huur     | Huurfiets, zoals OV-fiets   | \*      |
| {.data} |

<aside class='example' title="ProRail">

De gewenste voorzieningtypes voor ProRail:

- Rek = `{ type: 'r', vehicle: [{ owner: 'p', type: 'f' }] }` (voor normfietsen van normale mensen)
- Etagerek = `{ type: 'e', vehicle: [{ owner: 'p', type: 'f' }] }`
- Vak = `{ type: 'v', vehicles: [{ type: 'sb'}, { appearance: 'x' }] }` (snor-, brom- en sterk afwijkend model fiets)
- Kluis = `{ type: 'k', vehicles: [{ type: 'f' }]}`
- Buitenmodel (beperkt afwijkend)-rek = `{ type: 'r', vehicles: [{ type: 'f', accessory: [{type: 'p'}] }] }`
- Buitenmodel (beperkt afwijkend)-rek etage = `{ type: 'o', vehicles: [{ type: 'f', accessory: [{type: 'p'}] }] }`
- Deelfiets-rek = `{ type: 'r', vehicle: [{ type: 'f', owner: 'h' }] }`
- Deelfiets-vak = `{ type: 'v', vehicle: [{ type: 'f', owner: 'h' }] }`

De gewenste fietstypes voor ProRail:

- **Normfiets**
  - Fiets = `{ type: 'f', owner: 'p' }`
- **Beperkt afwijkend**
  - Fiets met kratje voor = `{ type: 'f', owner: 'p', accessories: [ { type: 'k', position: 'v' } ]}`
  - Fiets met kinderzitje achter = `{ type: 'f', owner: 'p', accessories: [ { type: 'z', position: 'a' } ]}`
- Geel kenteken = `{ type: 'b', owner: 'p' }`
- Blauw kenteken = `{ type: 's', owner: 'p' }` (Is dat te onderscheiden tov huur?)
- Deelfiets = `{ type: 'f', owner: 'h' }`
- Deelscooter = `{ type: 'sb', owner: 'h' }`
- Bakfiets = `{ type: 'f', appearance: 'x' }`
- Buitenmodelfiets = `{ type: 'f', accessory: [{ type: 'p' }] }`
- Vierwieler = `{ type: 'f', appearance: 'x' }` (???)
- Overig = `{ type: 'a' }`

Is afwezigheid v/h kenmerk 'accessory' afwezigheid v/e accessory?
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
