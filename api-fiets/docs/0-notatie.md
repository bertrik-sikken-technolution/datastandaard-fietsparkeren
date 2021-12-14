## Notatie

Datatypes zijn de JSON-datatypes ([[ECMA-404]]) en ook de JSON-notatiewijze wordt gebruikt.

Typenotaties van [TypeScript] worden op plekken gebruikt. Daarbij geldt:

- waar `[]` achter een type staat, wordt een lijst van dat type objecten bedoeld.
- waar `{objecttype}<T>` staat, wordt bedoeld dat |T| een generiek type is bij |objecttype|.
- waar `Partial<{type}>` staat, wordt bedoeld dat alle eigenschappen van |type| optioneel zijn.

Waar de waarde een `string` is, uit een beperkte enumeratielijst, worden meerdere waardes aangegeven door ze met spaties te scheiden.

Waar in een URL een `{var}` voorkomt, staat dat deel voor een variabele die ingevuld moet worden.
Als meerdere variabelen dezelfde naam hebben, wordt er met een indexgetal naar de variabele met dat volgnummer verwezen.

<!-- <figure>
<figcaption>Gebruikte namespaces in dit document</figcaption>

| Prefix    | Namespace                                           |
| --------- | --------------------------------------------------- |
| `schema:` | `http://schema.org/`                                |
| `mv:`     | `http://schema.mobivoc.org/#`                       |
| `vp:`     | `https://data.velopark.be/openvelopark/vocabulary#` |
| `vpt:`    | `https://data.velopark.be/openvelopark/terms#`      |

</figure> -->

[typescript]: https://www.typescriptlang.org/
