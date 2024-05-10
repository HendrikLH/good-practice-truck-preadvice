# Good Practice: TruckPreAdvice
[![Made with love for Digital Cargo](https://img.shields.io/badge/Made%20with%20%E2%9D%A4%20for-Digital%20Cargo-dce435)](https://digital-cargo.org)
[![GitHub](https://img.shields.io/github/license/digital-cargo/good-practice-shipment-tracking)](https://creativecommons.org/licenses/by/4.0/)

## Abstract

Today, there´s a lack of transparency for the information on road trucks in the context of cargo transportation. This is often the result of strongly distributed responsibilities and tasks between trucking company, forwarder and air cargo carrier.

The ONE Record standard is supposed to solve this problem by providing a standard tp share any kind of data between any stakeholder. This good practice document describes the methodology for providing and consuming TruckPreAdvice Information via ONE Record, making this data effortlessly accessible to others. Based on the ONE Record API version 2.0.0 and the ONE Record Data Model version 3.0.0, this document provides guidance on how to implement this data in an easy-to-use and standardized manner.

## Introduction

In the dynamic world of logistics and cargo, shipment tracking stands as a cornerstone, ensuring visibility, predictability, and trust within the supply chain. 
Yet, as businesses expand and systems diversify, the industry faces a challenge: the myriad of non-standardized tracking systems, each requiring unique integration and understanding. This fragmentation not only complicates operations but also escalates costs and reduces efficiency.

Initiated and moderated by the International Air Transportation Association (IATA), in 2022, major stakeholders of the supply chain decided to aim for a renewed data sharing infrastructure for the global logistics networks by 2026.
Enter the ONE Record standard, which aims to unify, streamline and improve shipping data across the industry. 
By leveraging the ONE Record standard, stakeholders can draw on a unified data model and API that promotes seamless integration across various platforms and improves collaboration between various organizations. 
This standardization comes with a number of benefits, from reducing the complexity and cost of custom integrations to enhancing transparency and trust.
It lays the foundation for standardization, enabling a consistent data model and API across diverse platforms, thereby streamlining integrations and collaborations.
This uniformity heightens transparency, allowing stakeholders to effortlessly interpret shipment data, fostering trust throughout the supply chain.
Moreover, the standardized approach curtails complexities tied to integration, conserving both time and resources that might otherwise be diverted to bespoke solutions. 


### Scope

This good practice details the application of the ONE Record standard specifically in the context of TruckPreAdvice. 

**What this document covers:**

- **Business context**: Assumptions, prerequisites, and the broader business scenario where this good practice is applicable.
- **Technical examples**: Detailed descriptions and examples of the API calls, data model classes, data mappings, and their applications in the context of shipment tracking.

**What this document does not cover:**

- **Compelete implementations**: This good practice includes sample code to support knowledge transfer, it does not provide detailed implementation or out-of-the-box software.
- **Comparison with other standards**: This good practice describes the implementation with the ONE Record Standard. A comparison with other standards in the industry is not covered.
- **Vendor-specific implementations**: This document focuses on the standard itself and does not address specific third-party tools or solutions based on the ONE Record standard.
- **Complete technical specifications**: This document focuses solely on the ONE Record aspects pertinent to shipment tracking and doesn't encompass the entire technical breadth of the standard.
- **Industry-wide statistics**: This document does not provide exhaustive industry data or statistics on the adoption or performance of the ONE Record standard.

As the industry evolves, it is imperative for stakeholders to keep up to date on subsequent versions or changes to the standard.

**Target audience**

This document is intended for anyone interested in this topic. 

**Geographical coverage**

This shipment tracking best practice is globally applicable, unhindered by regional or national distinctions. 
With no legal or operational barriers to its adoption, the outlined solution is primed for worldwide deployment. 
As a result, companies of any size, at any location, can take advantage of the standardized workflows and increased efficiencies created by ONE Record.

### Variants

No relevant variants known.

## Background

### Business Process

The business process for this use case is quite simple. A trucker provides the truck and driver information plus the shipments on a truck, a GHA retrieves this information and shares dock assignments. This process is very general and can cover export e.g. acceptance at GHA, import pickups at GHA, incoming Road Feed Service, Forwarder´s pick up at Shipper. Principally, it can be used wherever the information of a transport using a TransportMeans (e.g. truck, train, AGV) is to be shared.

To examplify the use case, the following setting was assumed: 

- A Trucker wants to preAdvice

data was shared

Trucker


For this specific use case, the following information is 

### ONE Record Standard

The implementation of shipment tracking as described in this good practice is based entirely on the [ONE Record standard](https://github.com/IATA-Cargo/ONE-Record).

This good practice incorporates data classes of the [ONE Record cargo ontology v3.0.0](https://onerecord.iata.org/ns/cargo)
and the [ONE Record core code lists ontology v0.0.3](https://onerecord.iata.org/ns/coreCodeLists).

Furthermore, it utilises the [ONE Record API specificaiton v2.0.0](https://iata-cargo.github.io/ONE-Record/).

### Related Good Practices

This Good practice is closely related to the [ShipmentTracking](https://github.com/digital-cargo/good-practice-shipment-tracking) and the [shipment record](https://github.com/digital-cargo/good-practice-shipment-record). 

### Piece-centricity and physics-orientation

Today in air cargo, tracking information is typically provided at the shipment level, but the ONE Record data model follows the principle of piece-centricity as a core design principle.
Another design principle of ONE Record is its aim to reflect the actual physical world, its objects and activities. 
For example, in ONE Record, it is not a legal object or a paper document such as the Air Waybill (AWB) that marks the progress of a shipment and reaches a milestone. 
Instead, it is the actual [Piece](https://onerecord.iata.org/ns/cargo#Piece), the wrapping [Shipment](https://onerecord.iata.org/ns/cargo#Shipment), or a [TransportMovement](https://onerecord.iata.org/ns/cargo#TransportMovement) activity that reaches a milestone in the journey. 
For example, when every piece in a shipment has been loaded and the aircraft departs, we consider the entire shipment as having departed.

## Data Provisioning

### Data Model

**Class Diagam**

This good practice incorporates data classes of the [ONE Record cargo ontology](https://onerecord.iata.org/ns/cargo) 
and the [ONE Record core code lists ontology](https://onerecord.iata.org/ns/coreCodeLists).
For clarity, class inheritance and unused data properties are excluded, and only required properties and relationships are visualized in the following.

The following class diagram shows the LogisticsObject data classes used and their relationships to the LogisticsEvent data class in the context of ShipmentTracking.

```mermaid
sequenceDiagram
    participant Trucker TMS
    participant Trucker ONE Record Server
    participant GHA ONE Record Server
    participant GHA TMS
    autonumber
    Trucker TMS->>+Trucker ONE Record Server: Creat location "Export acceptance"
    GHA ONE Record Server->>+Trucker ONE Record Server: SUB on location "Export acceptance"
    note over Trucker ONE Record Server, GHA ONE Record Server: specific part per TruckPreAdvice below
    Trucker TMS->>+Trucker ONE Record Server: Creates a transportMovement with destination location "Export acceptance"
    Trucker ONE Record Server->>+GHA ONE Record Server: PUB notification for creation of transportMovement with destination location "Export acceptance"
    activate Trucker ONE Record Server
        GHA ONE Record Server->>+ Trucker ONE Record Server: GET transportMovement
        Trucker ONE Record Server->>+ GHA ONE Record Server: Provides transportMovement
        note over GHA ONE Record Server: includes LOs: loading, pieces, shipment, waybill, transportMeans, transportMeansOperator, externalReference
    deactivate Trucker ONE Record Server
    activate GHA TMS
        GHA ONE Record Server->>+GHA TMS: POSTS QDO-Request
        GHA TMS->>+GHA ONE Record Server: Creates transportMovement 1 for location 1 (incl. QDO-Code)
        GHA TMS->>+GHA ONE Record Server: Creates transportMovement n for location n (incl. QDO-Code)
    deactivate GHA TMS
    GHA ONE Record Server->>+ Trucker ONE Record Server: PATCH transportMovement 1 into pieces
    note over GHA ONE Record Server: via loading
    GHA ONE Record Server->>+ Trucker ONE Record Server: PATCH transportMovement 2 into pieces
    note over GHA ONE Record Server: via loading
    Trucker ONE Record Server ->>+Trucker TMS: Provide assigned arrivalLocation to driver (= ramp)
```

### Data Mapping

As there is no standardized solution in place, a mapping with existing standards is not done. 

### Implementation Guidelines

This section outlines mandatory and best practice guidelines for the TruckPreAdvice use case in accordance with the ONE Record standard. 
For every data class and property, compliance requires adherence to certain guidelines marked as MUST, while it is RECOMMENDED to follow others for best practices. 
Additionally, to facilitate comprehension, practical data examples are included to demonstrate the implementation of these guidelines.

**TransportMovement**

- For ShipmentTracking, every TransportMovement MUST have a [transportIdentifier](https://onerecord.iata.org/ns/cargo#transportIdentifier) property with the following structure:
`{carrier code in capital letters as two 2-digit code}{flight number 3-digit to 5-digit}{optional suffix}/{departure date as DDMMMyyyy}` or as regular expression:
`([A-Z]{2}|[A-Z\d]{2})\d{3-5}[A-Z]?\/\d{2}[A-Z]{3}\d{4}`. Examples: LH100S/16OCT2023, S72510/02NOV2023
- [arrivalLocation](https://onerecord.iata.org/ns/cargo#arrivalLocation) property MUST be a link to a Location data object
- [departureLocation](https://onerecord.iata.org/ns/cargo#departureLocation) property MUST be a link to a Location data object

```json
{
    "@context": {
        "@vocab": "https://onerecord.iata.org/ns/cargo#"
    },
    "@id": "https://1r.example.com/logistics-objects/bfcae0d4-9a29-4e60-880d-213aac434776",
    "@type": "TransportMovement",
    "actions": {
        "@id": "https://1r.example.com/logistics-objects/5a4ade17-fe91-4d0c-bb79-8685a99d5634"
    },
    "arrivalLocation": {
        "@id": "https://1r.example.com/logistics-objects/JFK"
    },
    "departureLocation": {
        "@id": "https://1r.example.com/logistics-objects/FRA"
    },
    "transportIdentifier": "LH400/16OCT2023"
}
```
([transport-movement-LH400.json](./assets/transport-movement-LH400.json))

**Location**

- A Location data object is a special LogisticsObject because it has a long lifespan and is linked comparatively often. Therefore, a location object SHOULD only be created once and then only referenced.
- It is possible that the same or a similar location is referenced by different organizations with different @id, e.g. because they are hosted on different servers. For example, a TransportMovement (on the ONE Record server of a carrier) refers to an FRA location, while a waybill (on the ONE Record server of a forwarder) also refers to an FRA location. In this case, both locations can have different @id. However, it is RECOMMENDED to refer to the same location (represented by the same @id) wherever possible.
- If only one data holder shares the data (variant 1 and variant 2), the @id of the Location object is the same.
- Since a Location object is typically stable yet frequently referenced master data, it is RECOMMENDED to choose an easily recognizable `@id`.
For instance, use `https://1r.example.com/logistics-objects/FRA` to represent Frankfurt Airport.
- For ShipmentTracking, besides the `@id` only the [locationCode](https://onerecord.iata.org/ns/cargo#locationCode) property MUST be set.
- Location data objects can be created ad-hoc during the data provisioning of shipment tracking data, e.g. when a new Location is referenced in a TransportMovement or Waybill object, or they can be created in advance, e.g. once during the initial setup of the ONE Record server and afterwards when a new location is added to the logistics network.

```json
{
    "@context": {
        "@vocab": "https://onerecord.iata.org/ns/cargo#"
    },
    "@type": "Location",
    "@id": "https://1r.example.com/logistics-objects/FRA",
    "locationCode": {
        "@type": "CodeListElement",
        "code": "FRA",
        "codeListName": "IATA airport codes"
    }
}
```
([location-FRA.json](./assets/location-FRA.json))

**Waybill**

- It is RECOMMENDED to use a defined schema for generation of @id of Waybill, e.g. using UUID v5 method in combination with a constant namespace UUID for all logistics objects of type Waybill and the waybill number as name.
See [UUID Version-5 Generator](https://www.uuidtools.com/v5), e.g. uuid5(namespace=6d5e79fa-3c9e-4e44-b4f0-b44cc5920f01, name=020-12345675-1) = 0615e450-ad51-552b-b512-45ae433ba3dd

```json
{
    "@context": {
        "@vocab": "https://onerecord.iata.org/ns/cargo#"
    },
    "@id": "https://1r.example.com/logistics-objects/1a8ded38-1804-467c-a369-81a411416b7c",
    "@type": "Waybill",
    "arrivalLocation": {
        "@id": "https://1r.example.com/logistics-objects/JFK"
    },
    "departureLocation": {
        "@id": "https://1r.example.com/logistics-objects/FRA"
    },
    "shipment": {
        "@id": "https://1r.example.com/logistics-objects/8a76ed85-959e-45d5-8c42-5fd39c08efb1"
    },
    "waybillNumber": "12345675",
    "waybillPrefix": "020",
    "waybillType": {
        "@id": "https://onerecord.iata.org/ns/cargo#MASTER"
    }
}
```
([waybill.json](./assets/waybill.json))

**Shipment**

```json
{
    "@context": {
        "@vocab": "https://onerecord.iata.org/ns/cargo#"
    },
    "@id": "https://1r.example.com/logistics-objects/8a76ed85-959e-45d5-8c42-5fd39c08efb1",
    "@type": "Shipment",
    "pieces": [
        {
            "@id": "https://1r.example.com/logistics-objects/21ed25ef-4ef9-45ac-9088-b003d32ded95"
        }
    ],
    "totalGrossWeight": {
        "@type": "Value",
        "value": {
            "@type": "http://www.w3.org/2001/XMLSchema#double",
            "@value": "100"
        },
        "unit": {
            "@id": "https://onerecord.iata.org/ns/coreCodeLists#MeasurementUnitCode_KGM"
        }
    },
    "waybill": {
        "@id": "https://1r.example.com/logistics-objects/1a8ded38-1804-467c-a369-81a411416b7c"
    }
}
```
([shipment.json](./assets/shipment.json))

**Piece**

```json
{
    "@context": {
        "@vocab": "https://onerecord.iata.org/ns/cargo#"
    },
    "@id": "https://1r.example.com/logistics-objects/21ed25ef-4ef9-45ac-9088-b003d32ded95",
    "@type": "Piece",
    "ofShipment": {
        "@id": "https://1r.example.com/logistics-objects/8a76ed85-959e-45d5-8c42-5fd39c08efb1"
    },
    "skeletonIndicator": {
        "@type": "http://www.w3.org/2001/XMLSchema#boolean",
        "@value": "true"
    },
    "grossWeight": {
        "@type": "Value",
        "value": {
            "@type": "http://www.w3.org/2001/XMLSchema#double",
            "@value": "100"
        },
        "unit": {
            "@id": "https://onerecord.iata.org/ns/coreCodeLists#MeasurementUnitCode_KGM"
        }
    }
}
```
([piece.json](./assets/piece.json))

**Loading**

```json
{
    "@context": {
        "@vocab": "https://onerecord.iata.org/ns/cargo#"
    },
    "@id": "https://1r.example.com/logistics-objects/5a4ade17-fe91-4d0c-bb79-8685a99d5634",
    "@type": "Loading",
    "loadedPieces": [{
        "@id": "https://1r.example.com/logistics-objects/21ed25ef-4ef9-45ac-9088-b003d32ded95"
    }],
    "servedActivity": {
        "@id": "https://1r.example.com/logistics-objects/bfcae0d4-9a29-4e60-880d-213aac434776"
    }
}
```
([loading.json](./assets/loading.json))

- For ShipmentTracking, the Loading data object is required to establish a connection between Pieces and the TransportMovements


**LogisticsEvent**

- LogisticsEvents are created in the context of a LogisticsObject, which MUST be consider when generating its `@id` property. The `@id` of a LogisticsEvent MUST be generated by using the `@id` of the LogisticsObject and appending `/logistics-events/{logisticsEventId}`, for example, `https://1r.example.com/logistics-objects/8a76ed85-959e-45d5-8c42-5fd39c08efb1/logistics-events/23e4d5f6-959e-45d5-8c42-5fd39c08efb1`
- For ShipmentTracking, the [eventTimeType](https://onerecord.iata.org/ns/cargo#eventTimeType) property MUST be set to [ACTUAL](https://onerecord.iata.org/ns/cargo#ACTUAL) or [PLANNED](https://onerecord.iata.org/ns/cargo#PLANNED). However, an LogisticsEvent with eventCode BKD MUST be only of eventTimeType [ACTUAL](https://onerecord.iata.org/ns/cargo#ACTUAL).
- For the [eventCode](https://onerecord.iata.org/ns/cargo#eventCode) property, a NamedIndividual from the [ONE Record core code lists ontology](https://onerecord.iata.org/ns/coreCodeLists) MUST be used.
- The [partialEventIndicator](https://onerecord.iata.org/ns/cargo#partialEventIndicator) property MUST only be used when some - but not all - pieces of a shipment have reached the milestone. In this case, this property MUST be set to `true` to indicate a partially reached milestone.
- The [recordedAtLocation](https://onerecord.iata.org/ns/cargo#recordedAtLocation) property MUST be a link to a [Location](https://onerecord.iata.org/ns/cargo#Location) data object.

For shipment tracking, Status Event Code, Reason Code (for DIS) and Partial ID (shipment level only) in data element <eventCode>
- concepts to map planning and actual status information
  - info from BKD status / booking info to be used for planned milestones
  - info from other status codes to be used for actual milestones


The following shows an example for a completes departure (DEP) milestone, without the [partialEventIndicator](https://onerecord.iata.org/ns/cargo#partialEventIndicator) property:
```json
{
    "@context": {
        "@vocab": "https://onerecord.iata.org/ns/cargo#"
    },
    "@type": "LogisticsEvent",
    "@id": "https://1r.example.com/logistics-objects/8a76ed85-959e-45d5-8c42-5fd39c08efb1/logistics-events/23e4d5f6-959e-45d5-8c42-5fd39c08efb1",
    "eventTimeType": {
        "@id": "https://onerecord.iata.org/ns/cargo#ACTUAL"
    },
    "eventCode": {
        "@id": "https://onerecord.iata.org/ns/coreCodeLists#StatusCode_DEP"
    },    
    "eventDate": {
        "@type": "http://www.w3.org/2001/XMLSchema#dateTime",
        "@value": "2023-04-01T10:38:01.000Z"
    },    
    "recordedAtLocation": {        
        "@id": "https://1r.example.com/logistics-objects/FRA"        
    }
}
```
([logistics-event-DEP.json](./assets/logistics-event-DEP.json))

The following shows an example for a partial completed departure (DEP) milestone with the [partialEventIndicator](https://onerecord.iata.org/ns/cargo#partialEventIndicator) property set to `true`:

```json
{
    "@context": {
        "@vocab": "https://onerecord.iata.org/ns/cargo#"
    },
    "@type": "LogisticsEvent",
    "@id": "https://1r.example.com/logistics-objects/8a76ed85-959e-45d5-8c42-5fd39c08efb2/logistics-events/23e4d5f6-959e-45d5-8c42-5fd39c08efb2",
    "eventTimeType": {
        "@id": "https://onerecord.iata.org/ns/cargo#ACTUAL"
    },
    "eventCode": {
        "@id": "https://onerecord.iata.org/ns/coreCodeLists#StatusCode_DEP"
    },    
    "eventDate": {
        "@type": "http://www.w3.org/2001/XMLSchema#dateTime",
        "@value": "2023-11-02T10:38:01.000Z"
    },    
    "partialEventIndicator": {
        "@type": "http://www.w3.org/2001/XMLSchema#boolean",
        "@value": "true"
    },
    "recordedAtLocation": {        
        "@id": "https://1r.example.com/logistics-objects/FRA"        
    }
}
```
([logistics-event-DEP-partial.json](./assets/logistics-event-DEP-partial.json))

**Linking LogisticsObjects and LogisticsEvents**

A LogisticsEvent (shipment LogisticsEvent all have first departure reached
shipment LogisticsEvent all have last arrived reached
The LogisticsEvent haben eine Sonderstellung, obwohl es nach dem ONE Record datenmodell möglichst ist die LogisticsEvents über die property #events an ein LogisticsObject zu hängen, ist vorgesehen diese über eine eingen Endpoint abzufragen.

When all pieces of a shipment have departed, the shipment has departed. 

### Examples

This section demonstrates the previously described [implementation guidelines](#implementation-guidelines) with examples.



**Refer to this legend to interpret the shapes used in examples:**
> - **Blue rectangle with solid blue line:** LogisticsObject (e.g. Shipment, Piece, TransportMovement)
> - **Blue rectangle with dashed yellow line:** LogisticsObject with [skeletonIndicator](https://onerecord.iata.org/ns/cargo#skeletonIndicator)=true
> - **Green diamond with solid green line:** LogisticsEvent without [partialEventIndicator](https://onerecord.iata.org/ns/cargo#partialEventIndicator)
> - **Yellow diamond with dashed yellow line:** LogisticsEvent with [partialEventIndicator](https://onerecord.iata.org/ns/cargo#partialEventIndicator)=true
> - **(Plan)** indicates that the LogisticsEvent is a planned milestone
> - **(Act)** indicates that the LogisticsEvent is an actual milestone
![Data Mapping Examples Legend](./assets/data-mapping-examples-legend.png)


#### Example 1a: Shipment with one piece (only planned milestones)

![Example 1a: Shipment with one piece (only planned milestones)](./assets/data-mapping-example-1a.png)

Example 1a shows a shipment with only one piece that has reached the milestone Booked (BKD). 
In addition, the milestones Freight on Hand (FOH) and Received from Shipper (RCS) are planned for the shipment and the piece.

#### Example 1b: Shipment with one piece (planned and actual milestones)

Example 1a is an extension of Example 1a and shows a situation where the milestones Freight on Hand (FOH) and Received from Shipper (RCS) are planned for the shipment and the piece were planned; and the shipment and its piece has reached the milestones: Booked (BKD), Freight on Hand (FOH), and Received from Shipper (RCS).

In legacy Cargo-IMP this status would be captured by the following FSU messages:

```
FSU/14 020-12345675FRAJFK/T1K100 BKD/16OCT1317/FRA/ACME
FSU/14 020-12345675FRAJFK/T1K100 FOH/16OCT1317/FRA/LCAG
FSU/14 020-12345675FRAJFK/T1K100 RCS/16OCT1317/FRA/LCAG
```

![Example 1b: Shipment once piece (planned and actual milestones)](./assets/data-mapping-example-1b.png)

#### Example 2a: Shipment with one piece and flight specific status

![Example 2a: Shipment with one piece with flight specific status](./assets/data-mapping-example-2a.png)

The shipment has reached five miletones: BKD, FOH, RCS, MAN, DEP
The piece has reached four milestones: FOH, RCS, MAN, DEP
Even if not necessary to linked logistics events to the TransportMovement object, it is helpful for data consumption to compare the eventCode and eventTime of the MAN or DEP LogisticsEvent.
The same MAN and DEP LogisticsEvents are created for the transport movement.

In legacy Cargo-IMP this status would be captured by the following FSU messages:
```
FSU/14 020-12345675FRAJFK/T1K100 BKD/16OCT1317/FRA/ACME
FSU/14 020-12345675FRAJFK/T1K100 FOH/16OCT1317/FRA/LCAG
FSU/14 020-12345675FRAJFK/T1K100 RCS/16OCT1317/FRA/LCAG
FSU/14 020-12345675FRAJFK/T1K100 MAN/LH400/16OCT/FRAJFK
FSU/14 020-12345675FRAJFK/T1K100 DEP/LH400/16OCT/FRAJFK/T1K100
```

#### Example 2b: Two shipments with one piece each (planned on same flight)

![Example 2b: Two shipments with one piece each and flight specific status](./assets/data-mapping-example-2b.png)

#### Example 2c: Rescheduled shipment with one piece

![Example 2c: Rescheduled shipment with one piece](./assets/data-mapping-example-2c.png)


#### Example 3: Shipment with two pieces and different status each

![Example 3: Shipment with two pieces and different status each](./assets/data-mapping-example-3.png)

This example demonstrates the use of the [partialEventIndicator](https://onerecord.iata.org/ns/cargo#partialEventIndicator).
A shipment gets a LogsticsEvent only if all pieces of a shipment have reached a milestone. 
If this is not the case, a LogsticsEvent with the `partialEventIndicator = true` is added to the shipment data object. 
Later, when all pieces of a shipment have reached a milestone, an additional LogisticsEvent without the [partialEventIndicator](https://onerecord.iata.org/ns/cargo#partialEventIndicator) property is added to the shipment data object.
Pieces will always have LogisticsEvent data objecs without the [partialEventIndicator](https://onerecord.iata.org/ns/cargo#partialEventIndicator).


#### Example 4: Split Shipment with two pieces

![Example 4: Split Shipment with two pieces](./assets/data-mapping-example-4.png)

The shipment is reached the MAN milestone.
Only one piece reached the DEP miletone. Therefore the shipment only has a DEP milestone with the `partialEventIndicator = true`
When the second piece reached the DEP milestone, the shipment will get a second DEP milestone with the `partialEventIndicator = true`, and a third DEP milestone without the `partialEventIndicator`


#### Example 5a: Transit Shipment with one piece, completely manifested

![Example 5a: Transit Shipment with one piece, completely manifested](./assets/data-mapping-example-5a.png)

This example demonstrates the benefits of having LogisticsEvents also added to the TransportMovements
by matching MAN#1 of Piece #1 with MAN#1 of TransportMovement.


#### Example 5b: Transit Shipment with one piece, not completely manifested

![Example 5b: Transit Shipment with one piece, not completely manifested](./assets/data-mapping-example-5b.png)


#### Example 6: Split Transit Shipment with two pieces

![Example 6: Split Transit Shipment with two pieces](./assets/data-mapping-example-6.png)

#### Example 7: Planned Shipment with one piece and LAT and TOA

## Data Exchange

The ShipmenTracking use case is an easy starting point for a ONE Record transition for data providers and consumers. 
As it doesn't directly include the conclusion of a contract and can usually be considered as 
"one way communication", not all technical ONE Record features must be used.

### Endpoints

The ONE Record API provides a set of endpoints to exchange data. 
However, not all endpoints are required for the ShipmentTracking use case, e.g. the endpoints for audit trail, and change requests. 

In the following table we describe the ONE Record API endpoints required for ShipmentTracking:

| Resource / Endpoint                                                            | HTTP Action | Description                                                           |
| ------------------------------------------------------------------------------ | ----------- | --------------------------------------------------------------------- |
| /logistics-objects/{{logisticsObjectId}}                                       | GET         | Get LogisticsObject details                                           |
| /logistics-objects/{{logisticsObjectId}}/logistics-events                      | GET         | Get all LogisticsEvents of a LogisticsObject                          |
| /logistics-objects/{{logisticsObjectId}}/logistics-events/{{logisticsEventId}} | GET         | Get LogisticsEvent details                                            |
| /subscriptions                                                                 | GET         | Provide subscription information to publisher                         |
| /subscriptions                                                                 | POST        | Request a subscription for a LogisticsObject                          |
| /access-delegation                                                             | POST        | Request access delegation for a LogisticsObject and/or LogisticsEvent |
| /action-requests/{{actionRequestId}}                                           | GET         | Check status of subscription or access delegation request             |
| /action-requests/{{actionRequestId}}                                           | DELETE      | Revoke a pending subscription or access delegation request            |
| /notifications                                                                 | POST        | Receive shipment tracking updates                                     |

### Security

ShipmentTracking considers various types of security requirements.

- **Public access:** Also known as `Open Tracking API`, this offers the easiest to implement but least secured access. 
The data provider makes the data available to the public without verifying the identity and permissions. (no authentication required, no authorization required)  
- **Authenticated access** (authentication required): This level requires data consumers to prove their identity before accessing the tracking information. 
This adds a layer of security by ensuring that only recognized clients can interact with the API.
- **Authorized access** (authentication and authorization required): This requires that the requestor MUST present a valid identity and it is checked who is trying to access the API 
and whether the person has sufficient authorization to perform the request.

The final decision on which security requirements are required for a specific use case is made by the data provider.

Considering the following scenarios when selecting the level of security:
- non sensitive vs. sensitive information
  sensitive data e.g.:
  - Tracking info for Valuable or Vulnerable shipments;
  - content of (M)AWB contractual data, i.e. beyond flight routing, quantity details and shipment status
- wer darf welche Daten sehen, etc. FWD, GHA, andere / Identifizierung


- Because of the specificaiton ofthe standard, every request to a logistics-object needs to be authenticated by definition.
- The authoriation and access limitation is up to the implementer.

As for every public facing web API, it is RECOMMENDED to follow security best practices, including authentication, authorization, data encryption, and others, to ensure safe and secure data exchange.

For security reasons, it is RECOMMENDED to restrict access to logistics objects and logistics events to keep track of data access and data consumers.

For this use case, the authorization approach is left over to the implementing party. 
As of the nature of the "open" tracking API, authentication might not be required at all.

- Open: 
- Authentication required
- Authorization (incl. authentication) required

Example JWT Token (encoded):

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJhYTk4N2RjZS02YjUxLTExZWUtYjk2Mi0wMjQyYWMxMjAwMDIiLCJleHAiOjE2OTczNzA5OTYsImlzcyI6Imh0dHBzOi8vYXV0aC5leGFtcGxlLmNvbS9vYXV0aDIvZGVmYXVsdC92MS90b2tlbi90b2tlbiIsInN1YiI6IjEyMzQ1Njc4OTAiLCJsb2dpc3RpY3NfYWdlbnRfdXJpIjoiaHR0cDovLzFyLmV4YW1wbGUuY29tL2xvZ2lzdGljcy1vYmplY3RzL29yZ2FuaXphdGlvbi0xIn0.B7tYWhuVwscgHkmNOOGueNQ7D3uM0QXy6Al6OTuKZq4
```

Example JWT token (decoded payload):
```json
{
  "aud": "aa987dce-6b51-11ee-b962-0242ac120002",
  "exp": 1697370996,
  "iss": "https://auth.example.com/oauth2/default/v1/token/token",
  "sub": "1234567890",
  "logistics_agent_uri": "http://1r.example.com/logistics-objects/organization-1"
}
```

The ONE Record API can use the claim `logistics_agent_uri` in the JWT token to identify the data consumer and determine access rights to the requested resource.



The solution should principally "open" to maximize the user's benefits and minimize hurdles of implementation. 
This means that a basic layer of information should be available for data consumers without authentication. 
Some stakeholders might still require technical features like API keys for technical management, 
hurdles should be kept as low as possible for the user.

Although the base layer is as open as possible, additional, more sensitive information can be made available over the same API endpoints after an authentication. 
Thus, this use case is a good starting point for entering the ONE Record digital eco system.



### Required API implementation

The publisher's ONE Record API MUST implement the 
- GET LogisticsObject endpoint
- Subscribe endpoint

The subscriber's MUST implement a `/notifications` endpoint to receive Notifications. 

## Required functions

The following technical features are required on the data provider side:

- Implemented basic requests: GET, POST
- Generating and managing links for linked data
- Support publish and subscribe


On the data consumer side, even less functions are required for pure data consumption from the open tracking API:

- Making basic GET request
- Retrieving data from linked data sources

single ONE Record server / multiple ONE Record clients

> For each use case what is required: server, client, endpoints

## Request Shipment Tracking data

> For the sake of better comprehensibility, in the following examples it is assumed tht all data objects are 
> provided by a single data owner and hosted on a single ONE Record server, e. g. 1r.example.com
> This is the case in variant 1 and variant 2 where only one data holder shares the data. (cf. [data holder variants](#variants))
> In a real world environment, data objects are distributed across multiple ONE Record servers. These can be, 
> for example, carriers, ground handling agent (GHA), and other parties that also provide milestones and status updates
> along the supply chain. While the base URL component of the URIs may change, the API interactions remain the same.
> In some cases, additional API calls will be required by the data consumer to follow the linked data principle.

### LogisticsObject URI

Every Logistic Object as defined the [data mapping](#data-mapping) MUST have a globally unique id.
This good practice follows the defined structure of logistics object URIs which can be found in the [ONE Record API specification](https://iata-cargo.github.io/ONE-Record/concepts/#logistics-object-uri).

#### Waybill Specific LogisticsObject URI

For tokenized URIs, it is assumed that there is always a first contact between data provider and data consumer in a ONE Record world. 
This results in a specific problem for the given use case. For an "open" API, a previous contact with a subscription cannot be assumed. 
For the "first contact", the data consumer requires the unique tokenized ID for a shipment to request the data from the data owner. 
However, at this point the tokenized URI is not yet known or communicated. The data provider on the other side can not provide the tokenized URI 
because the data consumer is unknown due to the assumption of an "open API".
 
To solve this problem, for this specific use case, the URI for the GET request should contain the AWB number as the logisticsObjectId for the request. 
The following section describes a reference implementation.

### Use Case: Request shipment status for given waybill identifier

> **Note:**
> 
> "JSON-LD uses the same array representation as JSON, the collection is unordered by default. While order is preserved in regular JSON arrays, it is not in regular JSON-LD arrays unless specifically defined."
> 
> Source: [JSON-LD 1.1 specification](https://www.w3.org/TR/json-ld11/#terms-imported-from-other-specifications)

### data holder depending on entry point

1) Entrypoint is Waybill data object for which the data consumer knowns the LogisticsObjectURI, e.g. AWB number, e.g. 020-12345676. 
Data consumer can follow property #shipment in Waybill to get Shipment data and - if available - LogisticsEvents for a Shipment. 

This is special use case for air freight shipment tracking and is meant to be easier the transition.
Existing shipment tracking solutions allow to query for known shipment ids, e.g. the Air Waybill number.
We encourage everyone to setup a HTTP redirection that accepts the following path structure
`{scheme}://{host}[:port]/[basePath]/logistics-objects/{logisticsObjectId}`
as described in the [ONE Record API specification](https://iata-cargo.github.io/ONE-Record/concepts/#logistics-object-uri).

However, with the deviation that the `logisticsObjectId` URI component should follow the following:
`awb-{AWB prefix}-{AWB number}`


| Component  | Explanation                                                | Example  | Example explanation          |
| ---------- | ---------------------------------------------------------- | -------- | ---------------------------- |
| AWB prefix | provides the AWB prefix as part of the uniqueID of the AWB | 020      | Example of LH Cargo's prefix |
| AWB number | provides the AWB number as part of the uniqueID of the AWB | 12345675 | random example               |

This allows to query a ONE Record API as follows:

Request:
```http
GET /logistics-objects/awb-020-12345675 HTTP/1.1
Host: 1r.example.com
Accept: application/ld+json
```

Response:

```http
HTTP/1.1 307 Temporary Redirect
Location: https://1r.example.com/logistics-object/1a8ded38-1804-467c-a369-81a411416b7c
```


The following response serves as an example.

It is a strongly simplified data set with a shipment with one piece only on one transport movement and two events:


The response contains the actual location of the requested object using the Location http header. 
The client then has retrieved the unique tokenized ID of the Waybill and can get them by requesting the actual URI.

Request:

```http
GET /logistics-objects/1a8ded38-1804-467c-a369-81a411416b7c HTTP/1.1
Host: 1r.example.com
Accept: application/ld+json
```

Response:

```bash
HTTP/1.1 200 OK
Content-Type: application/ld+json
Location: https://1r.example.com/logistics-objects/1a8ded38-1804-467c-a369-81a411416b7c
Type: https://onerecord.iata.org/ns/cargo#Waybill
Revision: 1
Latest-Revision: 1

{
   "@context": {
      "@vocab": "https://onerecord.iata.org/ns/cargo#"
   },
   "@id": "https://1r.example.com/logistics-objects/1a8ded38-1804-467c-a369-81a411416b7c",
   "@type": "Waybill",
   "waybillType": "https://onerecord.iata.org/ns/cargo#MASTER",
   "waybillNumber": "12345675",
   "waybillPrefix": "020",
   "shipment": {
      "@type": "Shipment",
      "@id": "https://1r.example.com/logistics-objects/8a76ed85-959e-45d5-8c42-5fd39c08efb1"
   }
}
```
_(This is a linked data representation of the Waybill)_

A ONE Record client can also request an embedded version of the Waybill using the `embedded` query parameter, , e.g.

```http
GET logistics-objects/1a8ded38-1804-467c-a369-81a411416b7c?embedded=true HTTP/1.1
Host: 1r.example.com
Accept: application/ld+json
```

Two important remarks on this:

1. Only data hosted on the same ONE Record server can be embedded, as otherwise the ownership control of another party would be violated.
2. Even if data is embedded, a link for every logistics object must be created additionally, to enable essential ONE Record features like audit trail, pub/sub, access control, etc. for these objects.

### Receive Notifications after Subscribe

#### Scenario 1 - Receive Complete Shipment Status after Notification

Sequence diagram
Prerequisite: Subscribed on shipment by publisher
A -> Notificaions -> B
A <- Get LogisticsEvents <- B

By receiving the non-partial LogisticsEvent for a milestone, 
the data consumer knows that all pieces of that shipment have reached that milestone.

#### Scenario 2 - Partial Shipment Status after Notification

Sequence diagram
Prerequisite: Subscribed on shipment by publisher
A -> Notificaions -> B
A <- Get LogisticsObject <- B
A <- Get LogisticsObject Piece A <- B
A <- Get LogisticsEvents <- B

By receiving a partial LogisticsEvent for a milestone,
the data consumer knwos that not the complete shipment, i.e. all pieces of that shipment, have reached that milestone.
Thus, if the data consumer wants to know which piece has reached or not yet reached the milestone, additional 
requests have to be made.


### Shipment data

The data field *Waybill#shipment* contains a link to a shipment. 
A shipment in ONE Record is the totality of physical entities under one contract. 
The *Shipment#totalGrossWeight* is a typical data field belonging in this object. 

```
### Piece linked to the shipment

Here, the piece has volume, dimensions and special handling codes (GEN, SPX and EAP).


## Special Case: Multi-Carrier tracking platform in a heterogenous environment

The mechanism as described above reflects the requirements of a single carrier or forwarder platform providing tracking data for themselves or other stakeholders that are not using ONE Record. Beyond that, there is a scenario where a tracking platform might want to offer a unified tracking mechanism in a heterogenous ONE Record / non ONE Record environment. All is covered by the mechanism as described above, except for the case the platform gets a call for an AWB number for a carrier that is providing data in ONE Record. 

A typical request in this case could look like this:

```http
GET /logistics-objects/awb-020-1234575 HTTP/1.1
Host: 1r.example.com
Accept: application/ld+json
```

Here, instead of providing the tracking data, the platform would need to re-direct the request to the airline´s ONE Record server. According to the HTTP standard, this could be done by answering with an HTTP/1.1 302 re-direct:

```http
HTTP/1.1 307 Temporary Redirect
Location: https://1r.carrier.com/logistics-objects/awb-020-1234575
```

This mechanism enables e.g. a platform to provide a unified ONE Record tracking API in a heterogenous environment, where single stakeholders are providing data in ONE Record format, and others don't.
  
## Shipment Tracking Subscribe

ShipmentTracking information can be shared with partners in two ways, either by pro-actively subscribing a known business partner or by request of a partner or 3rd party.

### Subscribe to Shipment Tracking Updates

In addition to a known business partner other parties might be interested in the shipment status, e.g. broker, consignee, ground handling agent, etc. 
These parties can actively subscribe by sending a subscription request for a specific shipment to its data owner / publisher.
After successful approval by the data owner, the publisher's ONE Record server begins sharing information with these parties whenever the shipping status information is updated.

The general mechanism of Subscription is described in the [ONE Record API specification](https://iata-cargo.github.io/ONE-Record/subscriptions/#subscribe-to-logistics-objects).

## Get subscribed to Shipment Tracking Updates 

**Subscription initied by party providing data:** For traditional FSU messaging transfer usually the relevant business partner, i.e. forwarding agent who has issued the AWB, is notified by the carrier proactively. This party usually also performs the booking and provides shipment data beforehand. In this context e.g. a completed booking and space allocation trigger submitting FSU messages to the business partner. For this purpose the carrier might maintain the messaging address (e.g. PIMA or SITA Address) in the internal customer database or any equivalent system.
A similar process is supported by One Record, i.e. notification process to a known business partner can be triggered by a dedicated shipment status, e.g. Booking completed - BKD. When e.g. BookingData and/or ShipmentRecord related data is shared via 1R beforehand this might be used as trigger.
Instead of a messaging address the OneRecord Server URI associated with that business partner is used to pass information to the right party. Similar to traditional messaging the carrier might have to maintain a list of URIs and related business partners for this purpose. 


## Guidelines for implementation

### Error Handling and ChangeRequest process

#### Considerations for Error Handling

- MIP Codes 
- HTTP Status Codes
  - 401
  - 403
  - 404
  - 500

#### Considerations for change and update process 
- MIP Codes; C indicator; 
- Not relevant?

If applicable, ONE Record ChangeRequest process has to be applied when updating logistics objects for a certain shipment.
Relevance of 1R ChangeRequest process depends on the applied data exchange scenario. Please refer to details above.
For any logistics object related updates where equivalent data elements exist in traditional messaging specifications 
it is recommended of using the appropriate MIP "Error" Code along with ChangeIndicator "C" as specified in IATA Message 
Improvement Programme as reference.

When data is updated, shared or requested to be shared errors might occur, e.g. requested shipment id is unknown, server 
not available, logistics object update restricted due to different owner, etc. In general, the standard processes as specified in IATA 1R documentation apply.
For any logistics object related errors where equivalent data elements exist in traditional messaging specifications it is recommended of using the appropriate 
MIP Error Code as specified in IATA Message Improvement Programme as reference.

The 1R ChangeRequest process and Error Handling procss is described in the [ONE REcord API specification](https://iata-cargo.github.io/ONE-Record/) 
chapters [Subscriptions](https://iata-cargo.github.io/ONE-Record/subscriptions/), [Notifications](https://iata-cargo.github.io/ONE-Record/notifications/), and [Action Requests](https://iata-cargo.github.io/ONE-Record/action-requests/).

 



# Migration from Legacy Data Exchange
As of now ShipmentTracking related data has been exchanged mostly via Cargo-IMP FSU and FSA messages. 
These messages provide status information for dedicated events of the airport to airport process on (M)AWB level. 
Same applied to equivalent Cargo-XML messages.
In contrast to that the 1R data model is based on piece level. Moreover, via 1R logistics event related information 
can be provided for any logistics objects available in the 1R data model. This involves differences to both, 
methodologies of data exchange and structure of data.
This has to be considered when migrating data exchange to 1R and/or transferring data between 1R and traditional 
data interchange methods.
The attached mapping instructions shall help to understand these differences, explain how to use the 1R data model 
to exchange ShipmentTracking related data, as well as provide guidelines of converting data from and to 1R. 

Compared to existing exchanges, e.g. via CargoIMP/CargoXML, some milestones in a ONE Record environment are assigned to other objects - partly more fine-grained. 

The table [ONE-Record-CargoIMP-Mapping.xslx](/assets/ONE-Record-CargoIMP-Mapping.xslx) illustrates the relationship of data classes and their data properties in ONE Record with the legacy standard Cargo-IMP and CargoXML through a mapping.




## Glossary
see [digita-cargo/glossary](https://github.com/digital-cargo/glossary)

## References

- ...
- ...
- ...
  
## Acknowledgements

The initial version of this document is the outcome of the 
"Joint ONE Record piloting and transition working group // technical part" at IATA. 
It was orchestrated by Arnaud Lambert of IATA as secretary and [Philipp Billion](https://github.com/DrPhilippBillion) of Lufthansa Cargo as chairman.

Special thanks to [Niclas Scheiber](https://github.com/NiclasScheiber), Frankfurt University of Applied Sciences for preparing version 3.0.0 of the 
ONE Record core ontology in coordination with the IATA ONE Record data model focus group.

## Community

### Contribute

See [CONTRIBUTING](CONTRIBUTING.md) for more details on how to contribute on this good practice.

### Issues
Issues related to this good practice are tracked on GitHub

- [View open issues](https://github.com/digital-cargo/good-practice-shipment-tracking/issues)
- [Create a new issue](https://github.com/digital-cargo/good-practice-shipment-tracking/issues/new)

### Maintainers

> Each good practice MUST have at least one maintainer who is responsible for ongoing development and quality assurance. 
> Every maintainer MUST have commit access to the good practice repository.

- [Daniel A. Döppner](https://github.com/ddoeppner), Lufthansa Industry Solutions 
- [Ingo Zeschky](https://github.com/ChrisKranich), Lufthansa Cargo
- [Philipp Billion](https://github.com/DrPhilippBillion), Lufthansa Cargo

_(sorted alphabetically)_

### Contributors

> Every good practice is the result of the work of the community, and therefore the contribution of each individual should be recognized and appreciated. 
> Below is a list of all the people who have actively contributed to this good practice.

- Ajay Manoharan, Qatar Airways
- Arnaud Lambert, IATA
- Bilel Chakroun, Air France-KLM 
- Josh Priebe, Air Canada
- Keith Lam, GLS HKG 
- Mark Belliss, British Telecom 
- Martin Fowler, MDF Solutions
- [Martin Skopp](https://github.com/mskopp), Riege Software
- Mary Stradling, DHL
- Matthias Hurst, Colog AG
- Pramod Rao, Nexshore Technologies
- [Ying Lu](https://github.com/luyinglu), Lufthansa Cargo

_(sorted alphabetically)_


