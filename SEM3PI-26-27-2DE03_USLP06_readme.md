# [USLP06] Import stores and levels from GeoJSON file

* **Priority:** TBD (to be defined in Sprint Planning)
* **Story Points:** TBD (to be defined in Sprint Planning)
* **Module / Component:** `Back office app` (`app-three`) / `GeoJSON import model`
* **Status:** Ready for dev (documentation finished, next is TDD tests development)

---

## 👤 User Story Description

**As a** Retailer Administrator <br>
**I want to** use the back-office application to import
the stores and levels of a shopping location from a GeoJSON file <br> 
**So that** create the stores available at a shopping location

---

## 🚦 Definition of Ready (DoR)

* [x] **Clear User Story:** Story follows the standard *As a / I want to / So that* format.
* [ ] **Acceptance Criteria Defined:** Taken directly from the project requirements document (see below).
* [ ] **Dependencies Identified:** All the upstream dependencies in "Found Out Dependencies" below are fulfilled beforehand.
* [ ] **Inputs/Outputs Mapped:** All Input/Output are defined and documented in the "Input & Output Data" section below.
* [x] **Estimation Complete:** Story has been refined and estimated (story points assigned) by the dev team (8 points estimated).
* [x] **Small & Actionable:** Story is scoped down enough to be completed within a single sprint.

---

## 💬 Customer Specifications & Clarifications
**From the assignment document:** 
* **Selection of the Shopping Location:** A list of all existing Shopping Locations must be presented to the user so one can be selected.
* **Input of the GeoJSON file:** The path to the GeoJSON file to be used (corresponding to the NorteShopping Shopping Location) is located in the system, as a properties file with a default filename, configurable  without code changes. <br>**When you intend to try the feature of this US, you must have a geosjon file in your storage, access the "geoJSON_filePath_toImport" text file and add/replace in it with the path to YOUR GeoJSON in YOUR storage. That text file is present inside the app-three internal files.**
* **GeoJSON format specification `at Requirements doc/4.1 Shopping Location`**  The GeoJSON file contains the geographical layout of the shopping center, in which each area (e.g., stores, corridors, parking) is represented by the coordinates of a polygon’s vertices and associated with a specific floor.
* **Data presentation `at Requirements doc/LAPR3/USLP06`** The Java console application must display an explicit list of the imported floors and stores, and report any errors in a clear, understandable manner if the file is corrupted or malformed.
* **Persistence through the database `at Requirements doc/4.1 Shopping location` `at Requirements doc/LAPR3/USLP06`** All valid imported information that was then selected by the user must be persisted in the relational database via the data access layer.
* **Out of Scope:** Importing autonomous navigation routes for robots (to be addressed in dedicated User Stories)

**From client meetings:**
* The use of a GeoJSON-specific library to aid development is acceptable and preferred.
* For now, for the scope of this US, Stores fields must consist of, at least, the name and level
* A Store without a level shall be skipped and a warning must be logged
* In the output, a list of all imported Levels must be present, and below each one the imported Stores that belong to it, listed with their names.

**From Moodle forums:** 
* Only Features which have a property entry "shop" are considered shops and to be imported
* Every other Feature that represents anything else other than Levels or Shops is not to be imported at this moment, staying out of scope of this US. The same applies to any other property entry in the Store features outside of "id", "name", "shop_type" and "level"
* Some of the Features contain decimal numbers (like 0.5) as values of the "level" properties entry. Those are to also be considered Levels to be imported.
* The PoC for this project (this meaning, the following applies for the entire project, not only the scope of this US) only envisions one Shopping Location, which is NorteShopping. This means there isn't a need for the user, in this US, select which Shopping Location will be the import destination.
* If a new import includes Levels or Stores already existing in the database, the system does not add them, adding only the new ones, making this an effective update to the database.
---

## Extra Research - GeoJSON file
For this User Story there was some extra research done on the GeoJSON file and format, due to the parsing process needed to identify all Levels and Stores present on it. <br>
These were the most relevant findings for the scope of this US, based on the given GeoJSON file example and the RFC 7946 published at https://www.rfc-editor.org/info/rfc7946/:

> GeoJSON is a format for encoding a variety of geographic data
structures using JavaScript Object Notation (JSON) [RFC7159]. <br>
> A GeoJSON object may represent: 
> * a region of space (a `Geometry`), 
> * a spatially bounded entity (a `Feature`), 
> * or a list of Features (a `FeatureCollection`).  <br>
> 
> GeoJSON supports the following `Geometry` types: <br>
> * `Point` 
> * `LineString` 
> * `Polygon` 
> * `MultiPoint` 
> * `MultiLineString` 
> * `MultiPolygon` 
> * `GeometryCollection` <br> <br>
>
> `Features` in GeoJSON contain a member called `"type"` with the value "Feature", a member called `"geometry"`  with the value being a `Geometry` object, and a member called `"properties"` with the value being an Object containing additional properties that can take the form of String, numbers, boolean, null or other Objects. <br>
> A `FeatureCollection`  contains a list of `Features`. This is also the type of GeoJSON file we'll be working with on our developed solution. <br> <br>

For this specific US the focus is not on the `Geometry` objects or the coordinate system (whose parsing and inclusion in the database shall be addressed in the next Sprint 2, as new US on Robot navigation enter the scope), but the `"properties"` member of each `Feature`. That is where each Level is identified and each Store is identified and caracterized by its diverse attributes. <br> 
 
---

## Extra Research - Jackson Library and its `Jackson Databind` component
For this User Story there was a need to outsource the parsing process of the GeoJSON file, as permitted by the client (see Customer Specifications & Clarifications section). <br>
That being said, a choice was made to utilize the Jackson Java library to attend to this necessity.<br>
Note that **this is not a definitive choice of implementation**, as the Jackson library focus more on the properties and attributes aspects of GeoJSON features, but not so much on 
their geographical aspects (as coordinates, etc), which will most-likely be the focus on future sprints and implementations. <br>
There might be a need to switch to another GeoJSON and JSON parsing library, such as *GeoTools*, to accommodate to future sprints requirements. But for now Jackson will serve just fine. <br>

> Jackson is a Java library used to process JSON (JavaScript Object Notation) data, allowing applications to read, parse, and manipulate JSON without having to implement a parser from scratch. <br> For this feature, the Jackson Databind component is particularly useful, as it provides the ObjectMapper class and the JsonNode tree model, enabling the program to navigate the hierarchical structure of a GeoJSON file. <br> <br>
> The implementation uses ObjectMapper.readTree() to parse the GeoJSON file into an in-memory tree of JsonNode objects, representing JSON objects, arrays, and individual values. <br>The program can then navigate this tree to access the features array, iterate through its features, and inspect each feature's properties object. <br>Features containing a valid shop property are selected, and the required data is extracted to instantiate the corresponding Store and Level objects, which are subsequently added to the repository.

This approach delegates the JSON parsing to Jackson while keeping the feature-selection logic, domain-object creation, and repository management within the application's own implementation.

---

## ✅ Acceptance Criteria

* [ ] **AC1 - GeoJSON file import:** The app must import the data from the provided GeoJSON file. 
* [ ] **AC2 - List:** A list of the imported levels and stores must be presented to the user. 
* [ ] **AC3 - Database:** The app must insert the imported data into the database
* [ ] **AC4 - Error display:** Importing errors should be clearly displayed.

---

## 🔒 Non-Functional Requirements & Security

* **Architecture and persistence:** The back-office application must be developed in Java (with a text-based UI), accessing the relational database via JDBC and employing the DAO (Data Access Object) pattern in conjunction with PL/SQL APIs.
* **Data integrity:** Validation of business rules and relational integrity constraints when adding new floors and stores 
* **Error handling:** Capture of parsing or database connection errors, with explanatory logging.  

---

## 🔗 Found Out Dependencies

### Upstream Dependencies (Required before starting)
* **[USLP05] Back-office application skeleton:** Needs a base for the app to build on top of.
* **[USBD04] & [USBD05]: Relational model and instantiation** Needs the relational model instantiated in an Oracle SGBD, for data storing purposes.  

### Downstream Dependencies (Impacted by this story)
* **[USLP04] List available stores:** Needs the stores and respective levels already imported from the GeoJSON file and stored in the database 

## 📥 Input & Output Data

### Inputs
| Field Name             | Data Type          | Required | Source          | Example / Constraints                                                                                                        |
|:-----------------------|:-------------------|:---------|:----------------|:-----------------------------------------------------------------------------------------------------------------------------|
| `geoJsonFielPath`      | `String`           | Yes      | Properties file | `"norte_shopping_layout.geojson"` / Constraint: must be an existing path and have a valid file path format                   |

### Outputs
| Field Name       | Data Type      | Destination      | Example / Description                                                                                                                                                                                                 |
|:-----------------|:---------------|:-----------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `importedLevels` | `List<Level>`  | Console UI       | List of identified Levels (ex: `"Level 0"`, <br>`"Level 1"`, `"Level 2"`, etc)                                                                                                                                        |
| `importedStores` | `List<Store>`  | Consule UI       | List of imported Stores (ex: `"FNAC"`, `"Worten"`, `"Zara"`, etc ) <br> This list is to be presented divided amongst the different levels of the previous Level list, each Level item has its Stores listed below it. |
| `statusMessage`  | `String`       | Console UI Alert | `"Import successfully concluded: 2 Levels and 12 Stores registered."`                                                                                                                                                 |
| `errorLog`       | `List<String>` | Console UI Alert | `"Error in line 34: 'shop' missing in the polygon."`                                                                                                                                                                  |

---

## Edge Cases

* **Invalid GeoJSON file path:** The app fails to find the GeoJSON file through the path provided in the properties file, aborting the process and displaying an error message.
* **Empty GeoJSON file:** The GeoJSON file exists and is found, but is empty. The parser picks up on this and aborts the process, displaying the message *"Error: the GeoJSON file is empty."*
* **Invalid syntax on GeoJSON file:** The Controller detects the JSON format error and aborts the process, displaying the message *"Error: wrongly formatted syntax in the GeoJSON file"* without modifying the database.
* **Attempted duplication:** If a Lev**e**l or Store already exists in the database for the same shopping center, the system prevents duplication and reports the ignored records.
* **Database connection failure during import:** The relational transaction is rolled back to ensure data consistency, and the persistence error is reported.



---

## 🧪 Functional Tests / Validation

| Scenario ID | Test Case / Steps                                                                                                               | Expected Result                                                                                                                                                                 |
|:------------|:--------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **TC-01**   | Import a valid GeoJSON file containing 2 Levels and 10 Stores into an existing shopping center                                  | The Levels and Stores are recorded in the database, and the summary list is successfully displayed in the console.                                                              |
| **TC-02**   | Provide an invalid path to the GeoJSON file                                                                                     | A clear error message is displayed, indicating the failure to find the GeoJSON file, and aborting the process (no changes made to the database)                                 |
| **TC-03**   | Provide an empty GeoJSON file                                                                                                   | A clear error message is displayed, indicating the GeoJSON file as empty and aborting the process (no changes made to the database)                                             |
| **TC-04**   | Provide a path to a file with an invalid JSON format                                                                            | A clear error message is displayed, indicating the parsing failure, with no changes made to the database.                                                                       |
| **TC-05**   | Import a GeoJSON file containing Levels or Stores that already exist in the database for this Shopping Location                 | A clear warning message is displayed, indicating the duplicate entity and it not being added to the database, every other valid Level and Store is still added to the database. |
| **TC-06**   | Simulate a failed connection to the database                                                                                    | A clear error message is displayed, indicating the failure to establish a connection with the database.                                                                         |

---

# 🧪 QA & Test Documentation

See Report at [QA & Test Documentation: USLP01 Feature Title](./qa-readme.md)

---

## 🧩 Related Modules & Classes

* **Module:** `JavaApps/app-three` (The Back-office application), using `shared-library` for DAO code 
* **Frontend classes:**
  * Component: `ImportGeoJSONUI`
* **Backend classes:**
  * Controller: `ImportGeoJSONController`
  * Service/Parser: `GeoJSONParserService` (uses `Jackson Databind` from Jackson external library)
  * Repository/DAO: `ShoppingLocationDAO`, `LevelDao` `StoreDao`
  * Domain entities: `ShoppingLocation`, `Level`, `Store` 
* **External library:** Jackson Java Library (specifically its `Jackson Databind` component) for GeoJSON general parsing


---

## 🔄 Sequence Diagram

![Sequence Diagram](sd.svg)

## 🏁 Definition of Done (DoD)

* [ ] Acceptance criteria verified by QA / Product Owner.
* [ ] Unit tests written and passing (minimum 80% coverage on new service logic).
* [ ] Data imported and duly validated in the relational database
* [ ] Exceptions and parsing errors properly handled and displayed
* [ ] Code submitted to the GitHub repository and validated through a continuous integration (CI) process