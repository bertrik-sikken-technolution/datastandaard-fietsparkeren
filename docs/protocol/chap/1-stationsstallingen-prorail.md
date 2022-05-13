# Stationsstallingen (ProRail)

Dit protocol is gemaakt in 2021-2022 voor de ProRail stationstallingtelling najaar 2022.
Het beschrijft hoe gebieden ingemeten moeten worden, welke typen voertuigen worden onderscheiden en meer.

## Voertuigtypen

## Indeling ParkingLocation en Section

- Secties aggregeren per ( _Type parkeersysteem_ , _Voor voertuigtype_)

## Naamgeving ParkingLocation

<!-- De naam (`s:name`) gegeven aan een voorziening (`fp:ParkingLocation`) bestaat uit:

- ProRail Referentiesysteem Hectometerpunt [(Geocode)][geocode]
- "-"
- ProRail-stallingsnummer (aangevuld met voorloopnullen tot twee cijfers bij t/m 99 voorzieningen, tot drie cijfers bij meer dan 100 voorzieningen.)

[geocode]: https://data.overheid.nl/dataset/prorail-referentiesysteem

<aside class='example'>

```json
{
  "@type": "ParkingLocation",
  "name": "545-023.0-01" // Roosendaal
}
```

</aside> -->

## Invulinstructie tabel

- `SurveyArea`
  - `surveyarea_localId`: Stationsafkorting (bijv. Rsd = Roosendaal) + evt. Windrichting (bijv. ZO = Zuidoost)
  - `surveyarea_name`: Aangeleverd door ProRail.
  - `surveyarea_authorityId`: ProRail's code = `DDAC9859-B98A-4D22-8F77B86AABDB5D7B`
  - `surveyarea_xtrainfo`: Stationscode o.b.v. kilometerlint (bijv. 545-023.0 = Roosendaal)
  - `surveyarea_surveyAreaType`: `stationsgebied`, `windrichting`
  - `surveyarea_status`: `new`, `approved`, `rejected`
- `ParkingLocation_static`
  - `parkinglocation_localId`: Stationsafkorting (bijv. Rsd = Roosendaal) + volgnummer.
    - Het volgnummer is zero-left-padded (01, 02, .., 09, 10, .., 99).
    - Volgnummer < 60: Locaties in beheer bij ProRail.
  - `parkinglocation_name`: Afkomstig van ProRail.
  - `parkinglocation_securityFeature`: CameraSurveillance wordt niet gebruikt.
  - `parkinglocation_status`: `new`, `approved`, `rejected`
- `Section_static`

  - `section_localId`: Stationsafkorting (bijv. Rsd = Roosendaal) + volgnummer ParkingLocation + volgnummer
    - Anders dan bij ParkingLocation, is geen speciale manier waarop het eigen volgnummer wordt opgebouwd.
  - `section_layout`: `"A"`
  - `section_parkingSystemType`: Code uit de enum `fp:ParkingSystemType` (`x` `r` `rb` `b` `bb` `o` `ob` `k` `n` `v` `a`)
  - `section_level`: 1 = boven niveau 0, -1 = onder niveau 0. Niveau 0 is begane grond, stationsplein, etc.
