# Good Practice: TruckPreAdvice

[](https://digital-cargo.org)
[](https://creativecommons.org/licenses/by/4.0/)

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

### Stakeholders

In this use case, four parties are involved: The trucking company, the forwarder, the GHA and the carrier. As the trucker usually acts as an agent for the forwarder, and the GHA as an agent for the carrier, we will focus on the roles of trucker and GHA in the following documentation.

### Business Process

The business process for this use case is quite simple. A trucker provides the truck and driver information plus the shipments on a truck, a GHA retrieves this information and shares dock assignments. Also, the terms "trucker" and "trucking company" are synonym here, as the information can be provided by the trucking company (e.g. directy from the TMS, ), or by the trucker (e.g. an app using ONE Record for exchanging data). This process is very general and can cover export e.g. acceptance at GHA, import pickups at GHA, incoming Road Feed Service, Forwarder´s pick up at Shipper. Principally, it can be used wherever the information of a transport using a TransportMeans (e.g. truck, train, AGV) is to be shared.

To examplify the use case, the following setting was assumed:

A Trucker/Trucking Company wants to preAdvice the truck related data for export acceptance at the GHA. In return, the trucker expects to get a ramp assignment and a QR code for each drop off for accelerated identification.

The following business data objects are shared by the trucker:

General Details

- Quick Drop-Off Airport
- Type of vehicle
- Plate
- Company name
- STA and/or ETA
- Piece(s) loaded

Driver details

- First name
- Last name
- Birth date
- Mobile Phone number
- Email address
- Type of ID
- ID Number
- Place of ID Issueing

Remarks: The Quick Drop-Off Airport could be removed as the drop off station is clear by the origin on the AWB.

In return, the GHA will provide

- QDO-Code (Cleartext and Barcode)
- QDO Groups with each
  - Dock (Name and Geolocation (optional))
  - Pieces to be droped of there
  - Time slot (optional)

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

## Data Sharing

### Sequence

The following sequence diagram shows in which sequence data object need to be created to fulfill the process.

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
    GHA ONE Record Server->>+ Trucker ONE Record Server: PATCH transportMovement n into pieces
    note over GHA ONE Record Server: via loading
    Trucker ONE Record Server ->>+Trucker TMS: Provide assigned arrivalLocation to driver (= ramp)
```

### Data Model

**Class Diagam**

This good practice incorporates data classes of the [ONE Record cargo ontology](https://onerecord.iata.org/ns/cargo) 
and the [ONE Record core code lists ontology](https://onerecord.iata.org/ns/coreCodeLists).
For clarity, class inheritance and unused data properties are excluded, and only required properties and relationships are visualized in the following.

The following class diagram shows the LogisticsObject data classes used and their relationships in the context of TruckPreAdvice.



### Data Mapping

As there is no standardized solution in place, a mapping with existing standards is not done.

### Implementation Guidelines

This section outlines mandatory and best practice guidelines for the TruckPreAdvice use case in accordance with the ONE Record standard. 
For every data class and property, compliance requires adherence to certain guidelines marked as MUST, while it is RECOMMENDED to follow others for best practices. 
Additionally, to facilitate comprehension, practical data examples are included to demonstrate the implementation of these guidelines.

**Location**

- As a starting point, there must be a specific location as a destination for truck drives to the carrier for the purpose of cargo drop off / pick up.
- As a convention for this, we would suggest to use a carrier specific wording, e.g. "FRA LH Trucking Gate". This object can either be created by the carrier, or the trucker. In our example, the trucker this object. In the other case, subscriptions and patch requests must be adjusted accordingly.
- It is also possible to use a more general location (e.g. a three letter code "FRA", provided by the Airport Operator). As the subscription on this location will inform all GHAs for all trucks, the access needs to be steered with Access control lists. This is not recommended as it might lead to very high data traffic on this one location.
- It is important, that the same location is only created ONCE, not by several parties.

**TransportMovement**

- To indicate an incoming truck with shipments to the carrier/GHA, the forwarder has to create a TransportMovement.
- The [arrivalLocation](https://onerecord.iata.org/ns/cargo#arrivalLocation) must link to the carrier´s / GHA´s gate.
- Carrier/GHA have to subscribe on that location, if the location is provided by the trucker, so they get notified whenever a truck is planned for this destination.
- The [transportIdentifier](https://onerecord.iata.org/ns/cargo#transportIdentifier) can be created according to the requirements and conventions of the trucking company and does not have to follow use case specific rules.
- To share the scheduled and/or estimated time of arrival at the carrier / GHA, at leas one MovementTimes object must be embedded in the transportMovement.
- The direction for the estimated arrival must be "INBOUND". As this is an essential part of the use case, the use of this data field is mandatory for this use case.
- The MovementTimeType must be set according to the nature of the MovementTime, "ESTIMATED" or "SCHEDULED"
- The MovementMilestone ?????
- The MovementTimestamp provides the arrival timestamp for this MovementTimeType.
- One or more movementTimes can be embedded, updated movementTimes are simply appended to the previous movementTimes. (CHECK?????)

([transport-movement-LH400.json](./assets/transport-movement-LH400.json))

**Location**

- A Location data object is a special LogisticsObject because it has a long lifespan and is linked comparatively often. Therefore, a location object SHOULD only be created once and then only referenced.
- It is possible that the same or a similar location is referenced by different organizations with different @id, e.g. because they are hosted on different servers. For example, a TransportMovement (on the ONE Record server of a carrier) refers to an FRA location, while a waybill (on the ONE Record server of a forwarder) also refers to an FRA location. In this case, both locations can have different @id. However, it is RECOMMENDED to refer to the same location (represented by the same @id) wherever possible.

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

**transportMeans**

- The transportMeans is directly linked into the TransportMovement as described above.
- For this use case, the transportMeans contain the information on the physical truck and the trucking company.
- The type of vehicle is defined in the vehicleType data field
- The plate is shared via the vehicleRegistration
- The trucking company is shared via the transportOrganization.
- The TransportOperator is the person driving the truck and is linked into this object

**person**

- The Person contains information on the driver.
- First and last name are mandatory for this use case.
- The field "document" is of type externalReference contains links to the ID document of the driver.

- The date of birth should be shared as well as a Mobile phone number and emailaddress in the embedded contactDetails.

**externalReference**

- The externalReference contains the id information of the driver.
- The Type of ID is share in the documentType. This can be e.g. "Driver Licence", "Passport" or "ID-Card"
- The ID Number is to be shared in the documentIdentifier
- Place of ID Issueing is to be shared in the createdAtLocation (LINK??)

**Loading**

- to link pieces and transportMovement, a loading action must be created. TransportMovement and LoadingAction are linked via "onTransportMeans" in the Loading and "LoadingActions" in the TransportMovement
  
- For the delivery process, it is recommended to use the loading object for the previous loading of the truck, indicating the loadingType "LOADING"; Carrier / GHA then share the unloading of the truck in a new LoadingAction.
  
- For a pickup, it is recommended to also use the loadingType "LOADING" and link the pieces to be loaded, but add an embedded actionTimeType "PLANNED" and an  actionStartTime with the estimated pickup-time
  
- This can be a skeleton, or provide information beyond linking a transportMovmenet with the Pieces can be shared
  
- As a minimum requirement, it must link the TransportMovement in the "servedActivity" and the loaded pieces in the "loadedPieces" data field
  

**Piece**

As ONE Record is piece-centric, pieces are the primary objects that are used for loading and unloading. This also corresponds to the physical world, where pieces, not shipments are loaded and / or unloaded

- There are no specific requirements to the piece for this use case, beyond the linking of the loading action in the involvedInActions.
  

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

**Shipment**

As the sharing of the AWB-Numbers of the Shipments on the truck is mandatory, a Shipment-LO must be created to link the pieces to the Waybill-Object. 

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

**Waybill**

- The purpose of the Waybill-LO(s) here is to share the AWB Number(s) of the shipments on the truck.
 
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


### Examples

This section demonstrates the previously described [implementation guidelines](#implementation-guidelines) with examples.


#### Example 1: TruckPreAdvice for Export Acceptance

...

#### Example 2: TruckPreAdvice for Import Shipment Pickup
...

## Data Sharing

In terms of data exchange, there are no specific requirements for this use case going beyong what e.g. sharing the shipment record requires.

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

- [Philipp Billion](https://github.com/DrPhilippBillion), Lufthansa Cargo

_(sorted alphabetically)_

### Contributors

> Every good practice is the result of the work of the community, and therefore the contribution of each individual should be recognized and appreciated. 
> Below is a list of all the people who have actively contributed to this good practice.

- [Oliver Ditz](https://github.com/oditz), Fraunhofer IML
- [Oliver Meschkov](https://github.com/Meschkov), CHI
- [Daniel A. Döppner](https://github.com/ddoeppner), Lufthansa Cargo
- [Ingo Zeschky](https://github.com/ChrisKranich), Lufthansa Cargo
- [Philipp Billion](https://github.com/DrPhilippBillion), Lufthansa Cargo

_(sorted alphabetically)_
