# Technical Documentation for Team 2DE02

Technical documentation of **ShopRobot**, the Integrative Project of the 3rd semester of LEI (2026/27).

## Team

| Number | Name | Sprint 1 Role |
| :--- | :--- | :--- |
| 1231752 | João Pedro Silva Ferreira | Scrum Master / DEV / QA |
| 1221508 | Tiago André Sá Marquez | DEV / QA |
| 1212081 | Francisco de Araújo Latourrette Marques | DEV / QA |
| 1231536 | Gonçalo de Menezes Pacheco Morais de Abreu | DEV / QA |
| 1231109 | Tiago José Oliveira Borges | DEV / QA |

## Glossary

[Glossary](./Glossaries/glossary/readme.md)

NOTE: The glossary should be in synchronization with the domain model at all times!

## Domain Model

The domain model is the shared understanding of the ShopRobot business and is updated whenever that understanding changes.

![Domain Model](./domain-model/V2-domain-model.svg)

Source: [V2-domain-model.puml](./domain-model/V2-domain-model.puml)

It covers five areas of the business:

- **Shopping Location** — retailers, shopping locations, levels, areas (from the GeoJSON layout) and elevators.
- **Stores and Products** — merchants, the stores that joined the solution and the products they sell through it.
- **Ordering** — customers, shopping carts, orders, order lines and delivery spots.
- **Robot Fleet** — robots, robot stations, control terminals and scheduled station movements.
- **Planning** — sub-orders assigned to robots and the routes (sequences of stops) that fulfil them.

Conventions followed in the diagram:

- only business concepts are included;
- every entity has an identity attribute from the business domain;
- attributes are listed with their name and a conceptual data type (`Text`, `Number`, `Integer`, `Boolean`, `DateTime`) or an enumeration;
- "complex" attributes are modelled as classes (value objects);
- associations have a business-meaningful name, a direction and multiplicities;
- enumerations are used for closed sets of values.

## ESINF Report

[ESINF Report — Sprint 1](./ESINF/report.md): data structures, time complexity and empirical study of the ESINF user stories (USEI01–USEI05). Each developer completes the section of their user story.

## Assigned Work

Please keep this table updated at all times!

### 🏃 Sprint 1 Work Summary

**DEV**

| Team Member | LAPR3 | BDDAD | ESINF | ARQCP | FSIAP |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **João Ferreira** | Task-CI, USLP02 | USBD03 | USEI02 | — | — |
| **Tiago Marquez** | USLP01 | USBD04 | USEI01 | — | — |
| **Francisco Marques** | USLP03, USLP05 | USBD05 | USEI04 | — | — |
| **Gonçalo Abreu** | USLP04 | USBD01 | USEI05 | — | — |
| **Tiago Borges** | USLP06 | USBD02 | USEI03 | — | — |

**QA**

| Team Member | LAPR3 | BDDAD | ESINF | ARQCP | FSIAP |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **João Ferreira** | USLP01 | — | USEI03, USEI04 | — | — |
| **Tiago Marquez** | Task-CI, USLP04 | USBD02, USBD05 | — | — | — |
| **Francisco Marques** | — | USBD01, USBD03 | USEI05 | — | — |
| **Gonçalo Abreu** | USLP02, USLP06 | — | USEI01 | — | — |
| **Tiago Borges** | USLP03, USLP05 | USBD04 | USEI02 | — | — |

### 🏃 Sprint 2 Work Summary

### 🏃 Sprint 3 Work Summary
