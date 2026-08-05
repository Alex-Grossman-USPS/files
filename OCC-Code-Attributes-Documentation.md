# OCC Code Attributes Documentation

## Overview
This document provides detailed documentation for the `jobHiringAttributesDataResponse` data structure based on the XML schema definition.

**Namespace:** `http://usps.gov/OccCd/DataRequest`

---

## Data Structure: jobHiringAttributesDataResponse

### Elements

#### 1. flsa
- **Data Type:** `string`
- **Cardinality:** Optional (0..1)
- **Description:** FLSA Designation Code (displayed on job offer)

---

#### 2. jobAttribute
- **Data Type:** Complex Type
- **Cardinality:** Optional, Unbounded (0..*)
- **Description:** Job Attribute Codes and Text

**Child Elements:**
- `code` (string) - Required
- `text` (string) - Required

---

#### 3. salary
- **Data Type:** Complex Type
- **Cardinality:** Optional (0..1)
- **Description:** Salary information

**Child Elements:**
- `min` (string) - Minimum salary
- `max` (string) - Maximum salary
- `wageType` (string) - Type of wage

---

#### 4. sortOrder
- **Data Type:** Complex Type
- **Cardinality:** Optional (0..1)
- **Description:** Vef Pref Sort Code and Text

**Child Elements:**
- `code` (string) - Sort order code
- `text` (string) - Sort order description

---

#### 5. picture
- **Data Type:** Complex Type
- **Cardinality:** Optional (0..1)
- **Description:** Picture Number Code & Text

**Child Elements:**
- `code` (string) - Picture number code
- `text` (string) - Picture description

---

#### 6. video
- **Data Type:** Complex Type
- **Cardinality:** Optional (0..1)
- **Description:** Video Number Code & Text

**Child Elements:**
- `code` (string) - Video number code
- `text` (string) - Video description

---

#### 7. gatingQuestion
- **Data Type:** Complex Type
- **Cardinality:** Optional, Unbounded (0..*)
- **Description:** Gating Question Codes & Text

**Child Elements:**
- `code` (string) - Question code
- `text` (string) - Question text

---

#### 8. jobOverview
- **Data Type:** Complex Type
- **Cardinality:** Optional (0..1)
- **Description:** Job Overview Code & Text

**Child Elements:**
- `code` (string) - Overview code
- `text` (string) - Overview text

---

#### 9. matchingQuestions
- **Data Type:** Complex Type
- **Cardinality:** Optional, Unbounded (0..*)
- **Description:** Matching Question Codes & Text

**Child Elements:**
- `groupId` (string) - Group identifier
- `code` (string) - Question code
- `text` (string) - Question text

---

#### 10. exam
- **Data Type:** Complex Type
- **Cardinality:** Optional, Unbounded (0..*)
- **Description:** Assessment/Examination information

**Child Elements:**

| Element | Data Type | Description |
|---------|-----------|-------------|
| `vendor` | string | Assessment Vendor |
| `packageId` | string | Vendor Package/Exam Id |
| `number` | string | Exam Number |
| `result` | string | Result Number |
| `indicator` | string | Internal or External indicator |
| `testDays` | string | Number of days in the test cycle |
| `score` | string | Passing Score |
| `expHours` | string | Number of Hours for Assessment Request Expiration |
| `passExpDays` | string | Number of days to calculate Expiration Date of Exam for Pass |
| `failExpDays` | string | Number of days to calculate Expiration Date of Exam for Fail |
| `passRetestDays` | string | Number of days to calculate when an applicant can retest to achieve score |
| `failRetestDays` | string | Number of days to calculate when an applicant can retest to achieve a better score |

---

#### 11. description
- **Data Type:** `string`
- **Cardinality:** Optional (0..1)
- **Description:** Job Description

---

#### 12. qualifications
- **Data Type:** `string`
- **Cardinality:** Optional (0..1)
- **Description:** Job Qualifications

---

#### 13. benefit
- **Data Type:** `string`
- **Cardinality:** Optional (0..1)
- **Description:** Job Benefit

---

#### 14. background
- **Data Type:** `string`
- **Cardinality:** Optional (0..1)
- **Description:** Job Background

---

#### 15. overview
- **Data Type:** `string`
- **Cardinality:** Optional (0..1)
- **Description:** Job Overview

---

#### 16. finePrint
- **Data Type:** `string`
- **Cardinality:** Optional (0..1)
- **Description:** Job Fine Print

---

#### 17. keywords
- **Data Type:** `string`
- **Cardinality:** Optional (0..1)
- **Description:** Job Keywords

---

#### 18. error
- **Data Type:** Complex Type
- **Cardinality:** Optional (0..1)
- **Description:** Error information

**Child Elements:**
- `flag` (string) - Error flag
- `code` (string) - Error code
- `message` (string) - Error message

---

## Summary Table

| Element | Type | Required | Multiple | Description |
|---------|------|----------|----------|-------------|
| flsa | string | No | No | FLSA Designation Code |
| jobAttribute | complex | No | Yes | Job Attribute Codes and Text |
| salary | complex | No | No | Salary information |
| sortOrder | complex | No | No | Vef Pref Sort Code and Text |
| picture | complex | No | No | Picture Number Code & Text |
| video | complex | No | No | Video Number Code & Text |
| gatingQuestion | complex | No | Yes | Gating Question Codes & Text |
| jobOverview | complex | No | No | Job Overview Code & Text |
| matchingQuestions | complex | No | Yes | Matching Question Codes & Text |
| exam | complex | No | Yes | Assessment/Examination information |
| description | string | No | No | Job Description |
| qualifications | string | No | No | Job Qualifications |
| benefit | string | No | No | Job Benefit |
| background | string | No | No | Job Background |
| overview | string | No | No | Job Overview |
| finePrint | string | No | No | Job Fine Print |
| keywords | string | No | No | Job Keywords |
| error | complex | No | No | Error information |

---

## Notes

- All elements are optional (minOccurs="0")
- Elements marked as "Multiple: Yes" can appear unlimited times (maxOccurs="unbounded")
- Complex types contain nested child elements with their own structure
- String types have no explicit length constraints defined in the schema

---

**Schema Version:** Based on XML Schema Definition for `http://usps.gov/OccCd/DataRequest`
**Last Updated:** 2026-08-05
