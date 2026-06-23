# ABAP Program: `ZSEC_USER_PROFILE_CHECK` — Step-by-Step Explanation

This is an SAP ABAP report that **synchronizes user security profiles in table T77UA** (HR Authorization Profiles) based on role assignments in custom table `ZPA_PROFILE_ROLE`.

---

## 1. Setup & Declarations

- Declares work tables for **T77UA** (HR user profiles) and **AGR_USERS** (role assignments)
- Defines internal tables for:
  - `lt` — users who *should* have profiles (from role assignments)
  - `lt77` — users who *currently* have profiles (from T77UA)
  - `lt77_upd` — records to **add**
  - `lt77_del` — records to **delete**
  - `lt_rep` — report output log

---

## 2. Selection Screen

Two blocks are presented to the user:

| Parameter | Purpose |
|---|---|
| `P_TEST` | Checkbox (default ON) — **Test/Simulation mode**, no actual DB changes |
| `P_UNAME` | Limit processing to a **single user ID** (for testing) |
| `S_EXLUPD` | List of users to **exclude from updates** |
| `S_EXLDEL` | List of users to **exclude from deletes** |

---

## 3. Confirmation Popup (Interactive Mode Only)

- If running **online** (not in background batch), a confirmation popup appears:
  > *"THIS WILL DELETE OR UPDATE USERS SECURITY PROFILE — DO YOU WANT TO CONTINUE?"*
- If the user answers **No**, the program exits immediately.

---

## 4. Data Collection — Who *Should* Have Profiles?

```abap
SELECT a~uname, b~profl1, a~from_dat
FROM agr_users AS a
INNER JOIN zpa_profile_role AS b ON a~agr_name = b~agr_name
WHERE a~to_dat >= syst-datum AND a~from_dat <= syst-datum
```

- Reads **active role assignments** from `AGR_USERS`
- Joins with custom mapping table `ZPA_PROFILE_ROLE` to find which **HR profiles** (`PROFL1`, `PROFL2`) a role maps to
- Does this **twice** — once for `PROFL1`, once for `PROFL2` (APPENDING)
- Result: `lt` = complete list of **user + profile combinations that should exist**
- Sorts and removes duplicates

---

## 5. Data Collection — Who *Currently* Has Profiles?

```abap
SELECT uname, profl FROM t77ua
FOR ALL ENTRIES IN lt_profl
WHERE profl = lt_profl-profl
  AND begda <= syst-datum AND endda >= syst-datum
```

- First collects all **distinct profile names** used in `ZPA_PROFILE_ROLE`
- Then reads **currently active** entries in `T77UA` matching those profiles
- Result: `lt77` = what currently exists in the system

---

## 6. Comparison — Determine Changes Needed

**Records to ADD (`lt77_upd`):**
```
Loop through lt (should exist)
  → If NOT found in lt77 (doesn't exist yet)
    → Add to lt77_upd
```

**Records to DELETE (`lt77_del`):**
```
Loop through lt77 (currently exists)
  → If NOT found in lt (shouldn't exist anymore)
    → Add to lt77_del
```

---

## 7. Optional Filter by Single User

```abap
IF p_uname IS NOT INITIAL.
  DELETE lt77_del WHERE uname <> p_uname.
  DELETE lt77_upd WHERE uname <> p_uname.
ENDIF.
```

- If a specific user was entered, **restricts all changes to that user only**

---

## 8. DELETE Processing (`FORM delete_prf`)

For each record in `lt77_del`:

1. Check if user is in the **exclusion list** (`S_EXLDEL`) → skip if excluded
2. If **not test mode**, call `ZCL_USER_ROLE_ASSIGN->DELETE_T77UA` to remove the profile
3. If delete succeeds, also **cleans up T77UU** (related user table) and issues `COMMIT WORK`
4. Logs result to report table (`lt_rep`)

---

## 9. UPDATE/INSERT Processing (`FORM update_prf`)

For each record in `lt77_upd`:

1. Check if user is in the **exclusion list** (`S_EXLUPD`) → skip if excluded
2. If **not test mode**, call `ZCL_USER_ROLE_ASSIGN->UPDATE_T77UA` with:
   - Begin date = **today**
   - End date = **9999/12/31** (open-ended)
3. On success, issues `COMMIT WORK`; on failure, captures the error message
4. Logs result to report table (`lt_rep`)

---

## 10. Report Output (`FORM report_out`)

Prints a formatted list:

| Column | Content |
|---|---|
| Col 2 | User ID |
| Col 16 | Profile name |
| Col 50 | Action result message |

Possible messages include:
- `Profile Updated`
- `Profile Deleted`
- `Profile not updated - user excluded`
- `Profile not deleted - user excluded`
- `Error - DELETE failed`
- Error details from BAPI messages

---

## Overall Flow Summary

```
Role Assignments (AGR_USERS)
    + Profile Mapping (ZPA_PROFILE_ROLE)
           ↓
    "Who SHOULD have profiles" (lt)
           ↓
    Compare with T77UA (current state)
           ↓
    ┌──────────────┐    ┌──────────────┐
    │  ADD missing  │    │ DELETE extra  │
    │  profiles     │    │  profiles     │
    └──────────────┘    └──────────────┘
           ↓
    Report output of all actions taken
```

> **Key safety features:** Test mode checkbox, single-user filter, exclusion lists, and online confirmation popup all prevent accidental mass changes.
