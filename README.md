
# Digipolis Event design & style requirements v0.0.1

## Document historiek

Versie       | Auteur                 | Datum      | Opmerkingen
------       | -------                | -----      | ------------
0.0.1        | Peter Claes - Tom Dierckx - Steven Van Denbroeck - Dries Droesbeke            | 5/08/2019 | Initial commit

## Cheat sheet


Dit hoofdstuk geeft een beknopte versie van de event requirements; meer
details en voorbeelden van elk item vind je verder in het document.

### Taal

Engels (Payload keys, topic, namespace, headers)

### Formaat Event documentatie

AsyncAPI v1.2.0, JSON en YAML

### Payload

-   JSON (= application/json), tenzij geen andere mogelijkheid (file/…)
    -   keys
        -   dubbele quotes
        -   camelCase
        -   geen dots (.)
        -   niet starten met een cijfer
        -   attribuut is optioneel indien de value geen meerwaarde heeft EN waarde null heeft
    -   values
        -   null
        -   string : dubbele quotes
        -   datum en timestamp : "YYYY-MM-DDThh:mm:ss+01:00" (RFC3339)
        -   duration : "PYYYY-MM-DDThh:mm:ss" (ISO8601)   
            vb. "P0003-04-06T12:00:00" (3 jaar, 4 maanden, 6 dagen, 12 uur)
        -   geospatiale data : bijvoorbeeld lengte en breedte coördinaten (rfc7946)  
            vb. {"type": "Point", "coordinates": [100.0, 0.0]}
        -   Arrays worden altijd geëncapsuleerd in een object
    -   Hiërarchische structuur (objecten)

### Channel

#### EventHandler - Namespace

#### Kafka - Topic

schema identifier (=version) in the header
<projectname>-<tenant>.<subject>.<format>
  
example 1:          
ocapi-antwerpen.deliverable.json
ocapi-mortsel.deliverable.json

Topicstructuur vastleggen is voor beheer, waarom niet met een moniker werken of uuid?
drunk.dingo

### Headers

Casing is verplicht.

Verplichte headers:
- Correlation-Id
- Timestamp: timestamp tot op seconden nauwkeurig altijd in RFC3339 formaat => 2012-01-01T01:00:00+01:00
- Content-Type


### Error model

In sommige gevallen kan er een error event verzonden worden. Deze moet in het volgend formaat zijn opgemaakt.
```
{
    "type": "string"                           (verplicht, URIformaat),
    "title": "string"                          (verplicht),
    "status": number                           (verplicht, HTTP response code),
    "identifier": "string"                     (verplicht, uniek),
    "code": "string"                           (verplicht, foutcode voor ontwikkelaars),
    "extraInfo": "custom"                      (optioneel, bv een array van validatiefouten)
}
```

### AsyncAPI

We gebruiken in de eerste versie AsyncAPI om de beschrijving te doen van de events. 
AsyncAPI is vergelijkbaar met OpenAPI 3.0 .


## Event design & style requirements

Waarom dan de nood aan zulke Event design & style requirements?

Het is belangrijk bij het implementeren van Events dat het ontwikkelteam consistent is gelijkaardig aan de API requirements ;) in :
-   formatteringen van datums, geolocaties,...
-   het optioneel maken van velden die niet voor alle consumers even nuttig zijn

## Event body

### JSON conventies

Hanteer onderstaande conventies voor berichten die JSON als payload formaat hebben.

-   Gebruik steeds dubbele quotes bij keys.
```prettyprint
"company" : "Digipolis"
```

-   Gebruik steeds dubbele quotes bij string values.
```prettyprint
"company" : "Digipolis"
```

-   Gebruik steeds camelCase om keys weer te geven.
```prettyprint
"addressLine" : "Generaal Armstrongweg"
```

-   Gebruik geen dots "." in keys.
```prettyprint
"address.street" : "Generaal Armstrongweg" wordt NIET toegelaten (gebruik dan hiërarchieën)
```

-   Keys mogen niet starten met cijfers. Dit verlaagt immers de leesbaarheid.
```prettyprint
"5street" : "Generaal Armstrongweg" wordt NIET toegelaten
```

-   Verwijder `null` waardes uit de event representatie indien deze geen betekenis hebben.
```prettyprint
"middleName" : null
```

-   Toon lege waardes in de event representatie.
```prettyprint
"middleName" : "",
"orders" : []
```

-   Encapsuleer arrays steeds in een object. Dit omdat bepaalde frameworks niet goed overweg kunnen met native arrays
```json
{
   "adresses":[
      {
         "street":"Generaal Armstrongweg",
         "number":2,
         "zip":2000,
         "city":"Antwerpen"
      },
      {
         "street":"Britselei",
         "number":57,
         "zip":2000,
         "city":"Antwerpen"
      }
   ]
}
```

### Datums en timestamps

Formatteer datums en timestamps steeds volgens [RFC3339](https://www.ietf.org/rfc/rfc3339.txt). JSON definieert immers geen standaard formaat voor datums en timestamps.
```prettyprint
"finishedAt" : "2012-01-01T01:00:00+01:00"
```
Het is aangewezen om de timezone "+01:00" te gebruiken maar "2012-01-01T00:00:00Z" is ook mogelijk.

### Durations

Durations worden geformatteerd volgens ISO8601.
``` prettyprint
"duration" : "P0003-04-06T12:00:00" (3 jaar, 4 maanden, 6 dagen, 12 uur)
```

### Geospatiale data

Alle geospatiale data wordt steeds geformatteerd volgens [RFC 7946](https://tools.ietf.org/html/rfc7946)

Deze standaard laat toe om van eenvoudige locatie objecten in een longitude, latitude array...

``` prettyprint
     {
         "type": "Point",
         "coordinates": [100.0, 0.0]
     }
```

tot meer complexe geospatiale objecten zoals bijvoorbeeld een polygoon (en nog veel meer) te definiëren.

``` prettyprint
     {
         "type": "Polygon",
         "coordinates": [
             [
                 [100.0, 0.0],
                 [101.0, 0.0],
                 [101.0, 1.0],
                 [100.0, 1.0],
                 [100.0, 0.0]
             ],
             [
                 [100.8, 0.8],
                 [100.8, 0.2],
                 [100.2, 0.2],
                 [100.2, 0.8],
                 [100.8, 0.8]
             ]
         ]
     }
```

### Hiërarchie

Kies steeds voor een hiërarchisch gestructureerde event representatie in plaats van een vlakke structuur. Een hiërarchische voorstelling biedt als grote voordeel dat het overzichtelijker en duidelijker is voor events met een groot aantal velden. Daarnaast biedt het als grote voordeel het hergebruik van entiteiten over Events of APIs heen.

Als algemene richtlijn kan je stellen dat wanneer het aantal velden \> 15 je best naar een hiërarchische structuur kan overgaan indien dit meer duidelijkheid schept.

Een voorbeeld van een hiërarchische structuur:
```json
{
  "company": "Google",
  "website": "http://www.google.com/",
  "address": {
    "line1": "111 8th Ave",
    "line2": "4th Floor",
    "state": "NY",
    "city": "New York",
    "zip": "10011"
   }
 }
```

met daar tegenover dezelfde representatie maar dan in een vlakke structuur:
```json
{
  "company": "Google",
  "website": "http://www.google.com/",
  "addressLine1": "111 8th Ave",
  "addressLine2": "4th Floor",
  "state": "NY",
  "city": "New York",
  "zip": "10011"
}
```
