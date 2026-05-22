# Technical Requirements Document

## Program: ZOM_AUTO_ASSIGN_JOB_ATTR_ROLES

### 1. Overview

| Field | Value |
|-------|-------|
| **Program ID** | ZOM_AUTO_ASSIGN_JOB_ATTR_ROLES |
| **Title** | Assign Role to User |
| **Request Number** | EHP8.23 OM TSLC 6424-6142 -E-95191 |
| **Transport Number** | DV1K947811 |
| **Message Class** | ZPOM_004 |
| **Report Type** | Executable (REPORT) |
| **Output Format** | Line-Size 132, Line-Count 65 |

### 2. Purpose

This program automates the assignment and removal of SAP authorization roles to/from users based on their organizational and positional attributes in the HR/OM (Organizational Management) structure. The program ensures that users holding specific job attributes (occupation codes, organizational units, or explicit user assignments) are granted the corresponding SAP roles, and users who no longer qualify have those roles revoked.

---

### 3. Functional Requirements

#### 3.1 Role Eligibility Determination

The program must determine user eligibility for each role defined in custom table **ZOM_JOB_ROLE** based on three object types:

| Object Type (OTYPE) | Description | Eligibility Logic |
|----------------------|-------------|-------------------|
| **C** (Occupation Code) | Role assigned by job occupation | Users holding positions linked to the specified occupation code are eligible. |
| **O** (Organizational Unit) | Role assigned by org unit | Users holding positions within the specified organizational unit are eligible. |
| **US** (User) | Role assigned directly to user | The specified user ID is directly eligible. |

#### 3.2 Role Assignment

- The program must **add** a role to any user who is eligible per ZOM_JOB_ROLE but does not currently have the role assigned in table **AGR_USERS**.
- Role assignment must use class **ZCL_USER_ROLE_ASSIGN**, method **ASSIGN_ROLE**.
- The assigned role validity period must be from the current system date (`SY-DATUM`) through `12/31/9999`.

#### 3.3 Role Removal

- The program must **remove** a role from any user who currently holds the role (per AGR_USERS) but is no longer eligible per ZOM_JOB_ROLE.
- Role removal must use class **ZCL_USER_ROLE_ASSIGN**, method **DELETE_ROLE**.

#### 3.4 Detail Assignment Handling

When a user is on a **detail assignment** (HR relationship type `Z09`):

- The program must evaluate eligibility based on the **detail position**, not the home position.
- If the detail position's occupation code is found in ZOM_JOB_ROLE for the role, the user **retains** the role.
- If the detail position's occupation code is **not** found, the program must check the detail position's **organizational unit** against ZOM_JOB_ROLE.
- If neither the occupation code nor the org unit qualifies, the role must be **removed**.

#### 3.5 Test Mode

- The program must support a **Test Mode** checkbox parameter (`P_TEST`), defaulting to checked (`'X'`).
- When Test Mode is active, the program must perform all eligibility checks and generate the report but must **not** execute any role assignments or removals.
- When Test Mode is unchecked, the program must execute actual role changes.

#### 3.6 Reporting

The program must generate an ALV-style list report with the following columns:

| Column | Position | Description |
|--------|----------|-------------|
| User ID (UNAME) | Col 2 | SAP User Name |
| Personnel Number (PERNR) | Col 16 | Employee personnel number from PA0105 |
| Status Text | Col 32 | Action result (e.g., "Role Added To User", "Role Deleted From User", "Failed To Update Role To User") |
| Role Name | Col 75 | SAP role (AGR_NAME) |

The report must include a page header displaying:
- Program name, title, date, and time
- System ID and client
- Page number

---

### 4. Data Model / Database Tables

#### 4.1 Custom Tables

| Table | Description | Key Fields Used |
|-------|-------------|-----------------|
| **ZOM_JOB_ROLE** | Maps job attributes to SAP roles | AGR_NAME, OTYPE, USROROBJID |

#### 4.2 SAP Standard Tables

| Table | Description | Purpose |
|-------|-------------|---------|
| **AGR_USERS** | Role-to-user assignments | Retrieve current role assignments for users |
| **HRP1000** | HR Object infotype (descriptions) | Retrieve occupation code descriptions (OTYPE = 'C', 'S') |
| **HRP1001** | HR Relationship infotype | Traverse OM hierarchy (position - occupation, position - org unit, person - position) |
| **PA0105** | Personnel Communication infotype | Map personnel number to SAP User ID (SUBTY/USRTY = '0001') |

#### 4.3 Relationship Types Used

| Relationship (RELAT) | Direction (RSIGN) | From - To | Purpose |
|----------------------|-------------------|-----------|---------|
| **007** | A (top-down) | Occupation - Position | Get positions for an occupation code |
| **003** | B (bottom-up) | Org Unit - Position | Get positions under an org unit |
| **003** | A (top-down) | Position - Org Unit | Get org unit for a position |
| **008** | B (bottom-up) | Person - Position | Get holder of a position |
| **Z09** | B (bottom-up) | Person - Position (Detail) | Get detail assignment holder |

---

### 5. Processing Logic

#### 5.1 High-Level Flow

1. Read all distinct roles from ZOM_JOB_ROLE
2. For each role, execute UPDATE_BY_ROLE:
   - a. Get current holders from AGR_USERS
   - b. Get eligible users by Occupation (OTYPE='C')
   - c. Get eligible users by Org Unit (OTYPE='O')
   - d. Get eligible users by User (OTYPE='US')
   - e. Resolve positions to person numbers
   - f. Handle detail (Z09) assignments
   - g. Build ADD list
   - h. Build DELETE list
3. ADD_ROLE: Assign roles via ZCL_USER_ROLE_ASSIGN
4. REMOVE_ROLE: Delete roles via ZCL_USER_ROLE_ASSIGN
5. PRINT_REPORT: Output results

#### 5.2 Detail Assignment Logic (Decision Tree)

- User has role in AGR_USERS?
  - **NO** - Check if user is eligible - YES - Add to ADD list / NO - Skip
  - **YES** - Check if user is eligible?
    - **YES** - Keep (no action)
    - **NO** - Is user on detail (Z09)?
      - **NO** - Add to DELETE list
      - **YES** - Is detail position's occupation in ZOM_JOB_ROLE?
        - **YES** - Keep (no action)
        - **NO** - Is detail position's org unit in ZOM_JOB_ROLE?
          - **YES** - Keep (no action)
          - **NO** - Add to DELETE list

---

### 6. Selection Screen

**Block B1** - Main Parameters:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| P_TEST | Checkbox | X (checked) | Test mode - no actual role changes when active |

---

### 7. Custom Class Dependency

#### ZCL_USER_ROLE_ASSIGN

| Method | Parameters | Description |
|--------|------------|-------------|
| **ASSIGN_ROLE** | IV_USER (SYST-UNAME), IV_ROLE (AGR_NAME), IV_BEGDA (DATUM), IV_ENDDA (DATUM) - ET_ERRORS (BAPIRET2_T) | Assigns a role to a user for the given validity period |
| **DELETE_ROLE** | IV_USER (SYST-UNAME), IV_ROLE (AGR_NAME), IV_BEGDA (DATUM), IV_ENDDA (DATUM) - ET_ERRORS (BAPIRET2_T) | Removes a role from a user |

**Exceptions for ASSIGN_ROLE:** NOBEGDA (1), NOENDDA (2), OTHERS (3)

---

### 8. Error Handling

| Scenario | Behavior |
|----------|----------|
| ASSIGN_ROLE returns SY-SUBRC not equal 0 | Report: "Failed To Update Role To User" |
| ASSIGN_ROLE returns error in ET_ERRORS | Report: "Failed To Update Role To User" |
| DELETE_ROLE returns error in ET_ERRORS | Report: "Failed To Delete Role From User" |
| No errors | Report: "Role Added To User" or "Role Deleted From User" |

---

### 9. Authorization and Security Considerations

- The program modifies SAP user role assignments; execution must be restricted to authorized security administrators.
- The program should be scheduled as a background job with appropriate authorization profiles.
- All changes are logged in the report output for audit purposes.

---

### 10. Performance Considerations

| Area | Implementation |
|------|----------------|
| Database reads | FOR ALL ENTRIES used for bulk position-to-person resolution |
| Duplicate elimination | SORT + DELETE ADJACENT DUPLICATES applied to user ID lists |
| Lookup optimization | BINARY SEARCH used for user list lookups |
| Hashed table | lt_pernr uses HASHED TABLE with unique key for O(1) lookups |

---

### 11. Assumptions and Constraints

1. Plan version **'01'** (active plan) is used for all OM queries.
2. Only **active** infotype records (ISTAT = '1') with valid date ranges (BEGDA <= today <= ENDDA) are considered.
3. Personnel-to-user mapping uses PA0105 subtype **'0001'** (SAP User Name).
4. Role validity end date is always set to **12/31/9999**.
5. The program processes **all** roles in ZOM_JOB_ROLE in a single execution.
6. Detail assignments (relationship Z09) take precedence over home assignments (relationship 008) when both exist for the same position.

---

### 12. Testing Criteria

| Test Case | Expected Result |
|-----------|-----------------|
| User holds qualifying position by occupation code | Role assigned |
| User holds qualifying position by org unit | Role assigned |
| User explicitly listed in ZOM_JOB_ROLE (OTYPE = 'US') | Role assigned |
| User no longer holds qualifying position | Role removed |
| User on detail to qualifying position | Role retained |
| User on detail to non-qualifying position and org unit | Role removed |
| Test Mode enabled | Report generated, no role changes executed |
| Test Mode disabled | Role changes executed and reported |
| Role assignment fails | "Failed To Update Role To User" reported |
| Role deletion fails | "Failed To Delete Role From User" reported |
