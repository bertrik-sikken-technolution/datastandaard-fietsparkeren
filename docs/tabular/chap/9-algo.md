# Usage and conventional conversion

This section describes how the tabular exchange format SHOULD be programmatically transformed into the canonical RDF exchange format.
An implementation MAY also transform it into other formats, provided it follows the conversions of this section.

## Notation

The following table describes the algorithm to convert the cell values of a row into the appropriate objects and object values.
Where multiple Transformations are described for the same ( Sheets, Column ) combination, the transformations should be processed top-to-bottom.

`if |value|:` and `elsif |value|:` and `else:` indicate conditional processing in pseudo-code.

## Conversion of columns

| Sheet | Column       | Target                                                                       | Note                                     |
| ----- | ------------ | ---------------------------------------------------------------------------- | ---------------------------------------- |
| \*    | ID           | if empty: Assign a new, unused `@id`.                                        |
| \*    | ID           | else: `@id`.                                                                 |
| \*    | validFrom    | if empty: `TODAY()`                                                          |
| \*    | validThrough | `TODAY() - 1`.                                                               | i.e. yesterday.                          |
| \*    | geometry     | In a tabular format: the intrinsic feature ID of the feature in a shapefile. | Collect the geometry from the shapefile. |

## Changes, updates and corrections

An implementation MAY perform data updates through a sheet. Then, these conditions apply:

- An upload CAN reuse the table row, but replace its id column with the value `new`.
- An implementation MUST set the old item's `validThrough` to yesterday.
- An implementation MUST generate the new item's id and `validFrom` and process all attribute values and relations supplied.
