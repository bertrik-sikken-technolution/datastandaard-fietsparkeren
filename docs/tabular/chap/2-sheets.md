# Column Definitions

## Layer `SurveyArea`

<aside class="advisement">Unfinished.</aside>

| Column                   | Relates to | Description                                                |
| ------------------------ | ---------- | ---------------------------------------------------------- |
| surveyarea_id            |
| surveyarea_localId       |            | A designation defined by the authority.                    |
| surveyarea_geolocation   |
| surveyarea_parentId      |
| surveyarea_parentLocalId |
| surveyarea_validFrom     |
| surveyarea_validThrough  |            | If filled in, last day for which measurements are allowed. |
| surveyarea_authorityId   |            | Opvragen bij platform.                                     |
| surveyarea_name          |
| ...                      |            | Extra attributes, as defined by authority.                 |

## Layer `ParkingLocation_static`

<aside class="advisement">Unfinished.</aside>

| Column                              | Relates to | Description                                                                               |
| ----------------------------------- | ---------- | ----------------------------------------------------------------------------------------- |
| parkinglocation_id                  | ...        | Identifier                                                                                |
| parkinglocation_localId             |
| parkinglocation_geolocation         |
| parkinglocation_name                |
| parkinglocation_locationFeatureType |            | Space delimited list of localnames of fp:LocationFeatureTypes (`Outdoor`, `Guarded`, ...) |
| parkinglocation_validFrom           |
| parkinglocation_validThrough        |
| parkinglocation_authority           |
| ...                                 |            | Extra attributes, as defined by authority.                                                |

## Layer `Section_static`

<aside class="advisement">Unfinished.</aside>

| Column                    | Relates to | Description |
| ------------------------- | ---------- | ----------- |
| section_id                |
| section_localId           |
| parkinglocation_id        |
| parkinglocation_localId   |
| section_name              |
| section_layout            |
| section_geolocation       |
| section_parkingSystemType |
| section_vehicleOwnerType  |
| section_level             |
| section_validFrom         |
| section_validThrough      |

## Layer `Survey`

<aside class="advisement">Unfinished.</aside>

| Column                     | Relates to | Description |
| -------------------------- | ---------- | ----------- |
| survey_id                  |
| survey_localId             |
| survey_name                |
| survey_authority           |
| survey_contractors         |
| survey_surveyAreas_id      |
| survey_surveyAreas_localId |

The reference to
the dataset license,  
what measurement protocol was used, and
which vehicle categories were used
cannot be indicated in the tabular exchange format.

## Layer `DynamicSection`

<aside class="advisement">Unfinished.</aside>

| Column                       | Relates to     | Process  | Required | Description                                                                      |
| ---------------------------- | -------------- | -------- | -------- | -------------------------------------------------------------------------------- |
| section_id                   | @id            |          |          | The URL identifier of the section.                                               |
| section_localId              | dct:identifier | ws-split |          | Any local identifier of the section.                                             |
| parent_id                    | @id            |          |          | The URL identifier of the parent section or parent facility.                     |
| parent_localId               | dct:identifier | ws-split |          | Any local identifier of the parent section or parent facility.                   |
| section_name                 | s:name         | lang     |          | The human-readable name of the section.                                          |
| section_layout               |                |          |          | The URL identifier of the layout protocol.                                       |
| section_beheerder            |                |          |          | The URL identifier of the ...                                                    |
| section_parkingSystemType    |                |          |          | The singular URL identifier of the parking system type for this parking section. |
| section_vehicleOwnerType     |                |          |          | The singular URL identifier of the vehicle owner type for this parking section.  |
| section_level                |                |          |          | The building level of the section.                                               |
| section_validFrom            |                |          |          | Datetime marking the start of this parking section.                              |
| section_validThrough         |                |          |          | Datetime marking the end of this parking section.                                |
| section_authority            |                |          |          | The singular URL identifier of the authority that created this section.          |
| survey_id                    | @id            |          |          | The URL identifier of the survey for which this section was measured.            |
| observation_id               | @id            |          |          | The URL identifier of the observation.                                           |
| observation_observationType  |                |          |          |
| observation_observationFrom  |                |          |          | Datetime when the observation started.                                           |
| observation_observationUntil |                |          |          | Datetime when the observation ended.                                             |
| observation_contractorId     |                |          |          | The URL identifier of the observing party.                                       |
| observation_note             |                |          |          | Any free text notes for the observation.                                         |
| occupation_id                | @id            |          |          | The URL identifier of the occupation measurement.                                |
| occuptation_measurementType  |                |          |          | The URL identifier of the measurement type.                                      |
| occupation_totalParked       |                |          |          | The number of parked vehicles in the section.                                    |
| <var>VehicleCategoryId</var> |                |          |          | The number of parked vehicles of the specified type in the section.              |

The variable <var>VehicleCategoryId</var> is replaced by the ID of the concrete vehicle categories used in the current Survey.
Unknown columns SHOULD correspond with the ID of a defined canonical vehicle.
