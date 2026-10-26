# QA & Test Execution Report: USEI01

## USEI01 — Data Import and System Initialization

* **User Story Reference:** [USEI01: Data Import and System Initialization](./readme.md)
* **Issue:** [#7 — USEI01 Data Import and System Initialization](https://github.com/Departamento-de-Engenharia-Informatica/SEM3PI-26-27-2DE02/issues/7)
* **Developer:** Tiago Marquez
* **QA:** Gonçalo Abreu (GmpABR)
* **Implementation Area:** ESINF / Planning area (CSV Import into memory)
* **Execution Date:** 2026-10-19
* **Environment:** Java 21, Maven 3.9, JUnit 5, JaCoCo; datasets in `data/medium` and `data/large`
* **QA Status:** 🟢 Passed (All unit and component tests passed; 0 defects)

---

## 🎯 QA Strategy & Scope

### In-Scope

* Verification of parsing and loading of entities (`Robot`, `RobotStation`, `Order`, `OrderItem`) from CSV files into memory (AC1).
* Correct use of Java Collection Framework (JCF) structures (preservation of input/arrival order with `LinkedHashMap`) (AC2).
* Proper error handling for malformed rows, invalid values, and duplicate IDs without terminating the import process (AC3).
* Execution of automated tests (`CsvDataImporterTest`, `ImportResultTest`).
* Static code review of compliance with non-functional requirements (no `PriorityQueue`, Javadoc present).

### Out-of-Scope

* Robot queue prioritization and station monitoring (covered by USEI02 and USEI03).
* Single-robot and multi-robot order assignment algorithms (USEI04, USEI05).
* Database persistence (out of scope for Sprint 1).

---

## 🛠️ Review Setup

* **Specification:** Project Specification v1, Section 7.2.1.
* **User Story Requirements:** [USEI01 readme](./readme.md).
* **Implementation under review:**
  * `CsvDataImporter` (`model/importer`)
  * `ImportResult` (`model/importer`)
  * `ImportError` (`model/importer`)
  * Domain entities: `Robot`, `RobotStation`, `Order`, `OrderItem`, `RobotStatus` (`model/domain`)
* **Automated tests:**
  * `CsvDataImporterTest`
  * `ImportResultTest`

---

## ✅ Acceptance Criteria

* **AC1 — Parsing & Loading:** Successful parsing and loading of all entities (Robots, Stations, Orders) into Java Collection Framework structures without data loss or corruption.
* **AC2 — Collection Choices:** Appropriate collection choices based on access patterns (preserving input order and efficient key lookup).
* **AC3 — Error Handling & Logging:** Informative error messages or logs for malformed lines or missing values during the import process.

---

## 📋 Execution Checklist & Results

| Step | Test / Verification | Expected Result | Actual Result | Status |
| :---: | :--- | :--- | :--- | :---: |
| **1.1** | `CsvDataImporterTest#importsCleanRobotStationAndOrderFiles` | Clean files imported completely into domain models without errors | 2 robots, 1 station, 1 order with 2 items imported; all attributes mapped accurately | 🟢 PASS |
| **1.2** | `CsvDataImporterTest#reportsMalformedRowsAndContinuesImporting` | Malformed rows skipped, specific `ImportError` generated, valid rows imported | 3 errors recorded with exact line/field; valid entities imported successfully | 🟢 PASS |
| **1.3** | `ImportResultTest#keepsTheOrdersInTheOrderTheyWereImported` | Order arrival/input sequence is maintained in `ImportResult` | Order IDs preserve exact insertion order | 🟢 PASS |
| **1.4** | `ImportResultTest#keepsTheRobotsAndStationsInTheOrderTheyWereImported` | Input sequence of robots and stations is preserved | Insertion order maintained | 🟢 PASS |
| **1.5** | Full Project Test Suite (`mvn test`) | All project unit and component tests execute cleanly | 141 tests run, 0 failures, 0 errors, 0 skipped | 🟢 PASS |
| **1.6** | Static Analysis / JCF Restrictions | Strict adherence to Sprint 1 rules: no heap or `PriorityQueue` used | Verified. Uses `LinkedHashMap` and immutable `List` | 🟢 PASS |

---

## 🔍 Detailed Output Highlights

### 1. Automated Tests Execution

**Command executed:**
```bash
mvn -B -Dtest=CsvDataImporterTest,ImportResultTest test
```

**Execution Output Summary:**
```text
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.example.shoprobot.model.importer.CsvDataImporterTest
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.337 s -- in com.example.shoprobot.model.importer.CsvDataImporterTest
[INFO] Running com.example.shoprobot.model.importer.ImportResultTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.021 s -- in com.example.shoprobot.model.importer.ImportResultTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

---

## 🐞 Issues Found

* **No blocking defects found.**
* Note: Previous concern regarding order preservation in `ImportResult` (addressed in `ImportResultTest`) was confirmed fixed; `ImportResult` preserves order correctly.

---

## 📊 Summary & QA Sign-Off

### Execution Results

* **Total test cases:** 5 (USEI01-specific), 141 (Global project suite)
* **Executed:** All
* **Passed:** 5 / 5 (141 / 141 global)
* **Failed:** 0
* **Blocked:** 0
* **Open QA issues:** 0

### Final QA Recommendation

> 🟢 **Approved:** The implementation of USEI01 fulfills all functional and non-functional acceptance criteria. Parsing is robust against errors, entity relationships and data types are correctly populated, and collections comply strictly with JCF requirements for Sprint 1.

* [x] **QA Sign-off Approved**
* **Signed By:** Gonçalo Abreu (GmpABR)
* **Date:** 2026-10-19
