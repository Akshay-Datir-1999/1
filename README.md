# ERP-SYSTEM 
## Introduction

This repository contains a detailed functional and UI testing report for the **GTVL ERP System**.  
The objective of this assignment is to thoroughly test the system and document all valid bugs identified
across the following modules:
- Management Portal  
- Promodizer Portal  
- Store Performance Analysis  
- Supervisor Portal  

Each bug in this report includes:
- Detailed description  
- Expected vs Actual results  
- Evidence (screenshots/videos)  
- Structured Bug ID format  
- Organized, module-wise presentation  

Use the sections below to quickly navigate through the complete bug summary.

---

<a id="bug-index"></a>

## Bug Index Table (Module-wise – Click to Expand)

<details>
<summary><strong>Module 1 — Management Portal Bugs</strong></summary>

| Bug ID | Title |
|--------|-------|
| [P1-F01](#bug-id-p1-f01) | Arrow icons inside dashboard boxes are not clickable |
| [P1-F02](#bug-id-p1-f02) | Edit, View, and Delete buttons do not work for several SKUs |
| [P1-F03](#bug-id-p1-f03) | System accepts duplicate SKU Codes without validation |
| [P1-F04](#bug-id-p1-f04) | Export button in Store Management does not perform any action |
| [P1-F05](#bug-id-p1-f05) | Postal Code field accepts alphabets and special characters |
| [P1-F06](#bug-id-p1-f06) | Mobile Number field does not allow deleting or editing digits |
| [P1-F07](#bug-id-p1-f07) | Edit, View, and Delete buttons do not work for any store |
| [P1-F08](#bug-id-p1-f08) | Supervisor First Name and Last Name fields accept numbers and duplicates |
| [P1-F09](#bug-id-p1-f09) | Promodizer Edit form allows changes but does not save updates |
| [P1-F10](#bug-id-p1-f10) | Promodizer First Name and Last Name fields accept numeric input |
| [P1-F11](#bug-id-p1-f11) | SKU remains visible after deletion until page refresh |
| [P1-F12](#bug-id-p1-f12) | No success message after deleting SKU |
| [P1-F13](#bug-id-p1-f13) | Status label inconsistent (“active” vs “Active”) |
| [P1-F14](#bug-id-p1-f14) | Import button uses wrong icon (Export icon shown) |
| [P1-F15](#bug-id-p1-f15) | Radio button shows extra square box when selected |
| [P1-F16](#bug-id-p1-f16) | Chatbot icon overlaps action buttons |
| [P1-F17](#bug-id-p1-f17) | GTVL icon color mismatch across portals |

</details>

<details>
<summary><strong>Module 2 — Promodizer Portal Bugs</strong></summary>

| Bug ID | Title |
|--------|-------|
| [P2-F01](#bug-id-p2-f01) | Home page icons are not properly aligned across Promodizer sections |
| [P2-F02](#bug-id-p2-f02) | Recorded sales do not appear in the Sales History section |
| [P2-F03](#bug-id-p2-f03) | Feedback form does not submit and shows an error message |
| [P2-F04](#bug-id-p2-f04) | Sidebar menu does not highlight the selected option |
| [P2-F05](#bug-id-p2-f05) | Calendar dates are misaligned under incorrect weekday columns |
| [P2-F06](#bug-id-p2-f06) | My Profile page alignment is inconsistent compared to other sections |
| [P2-F07](#bug-id-p2-f07) | Barcode field allows longer pasted input than typed input |
| [P2-F08](#bug-id-p2-f08) | Attendance History displays attendance only on Sundays |
| [P2-F09](#bug-id-p2-f09) | Dashboard icons not following theme (inconsistent colors) |
| [P2-F10](#bug-id-p2-f10) | Dropdown misaligned in mobile view |

</details>

<details>
<summary><strong>Module 3 — Store Performance Analysis Bugs</strong></summary>

| Bug ID | Title |
|--------|-------|
| [P3-F01](#bug-id-p3-f01) | SKU Performance table and filter panel are misaligned |
| [P3-F02](#bug-id-p3-f02) | Min and Max quantity filter fields accept negative values |
| [P3-F03](#bug-id-p3-f03) | Active filter does not reset and combined selection shows incorrect results |
| [P3-F04](#bug-id-p3-f04) | Custom date range option does not display Start and End date fields |
| [P3-F05](#bug-id-p3-f05) | Table alignment issues cause some records to be partially hidden |
| [P3-F06](#bug-id-p3-f06) | Filter fields accept negative values in the Store Performance section |
| [P3-F07](#bug-id-p3-f07) | Filter tag close (×) button does not remove the applied filter |
| [P3-F08](#bug-id-p3-f08) | Clicking any field in the Staff Performance table redirects to “Staff Not Found” page |
| [P3-F09](#bug-id-p3-f09) | Clear All button position is inconsistent in Transaction History module |
| [P3-F10](#bug-id-p3-f10) | Transaction History table has inconsistent header styling and missing sorting arrows |
| [P3-F11](#bug-id-p3-f11) | Store filter dropdown displays duplicate store names |
| [P3-F12](#bug-id-p3-f12) | Sorting icons in Store Performance table are inconsistent in size and alignment |
| [P3-F13](#bug-id-p3-f13) | Date filter accepts invalid date ranges (From Date > To Date) | 
| [P3-F14](#bug-id-p3-f14) | Sorting icon not visible until hover and appears in inconsistent shape |
| [P3-F15](#bug-id-p3-f15) | Store Performance chart not visible on desktop view |
| [P3-F16](#bug-id-p3-f16) | Transaction History table fluctuates during sorting |
| [P3-F17](#bug-id-p3-f17) | SKU Performance sorting icon behavior not standard |
| [P3-F18](#bug-id-p3-f18) | Supervisor Assignment Stores missing labels & checkboxes |


</details>

<details>
<summary><strong>Module 4 — Supervisor Portal Bugs</strong></summary>

| Bug ID | Title |
|--------|-------|
| [P4-F01](#bug-id-p4-f01) | Dashboard does not update attendance status after marking attendance |
| [P4-F02](#bug-id-p4-f02) | Clock Out Time option is missing in the Attendance section |
| [P4-F03](#bug-id-p4-f03) | Mobile Number field accepts alphabets and special characters |
| [P4-F04](#bug-id-p4-f04) | Sales History selection auto-redirects to record details page and stuck on loading |
| [P4-F05](#bug-id-p4-f05) | Cancel button does not navigate back when form fields are empty |
| [P4-F06](#bug-id-p4-f06) | Two Cancel buttons are displayed and both perform the same action |
| [P4-F07](#bug-id-p4-f07) | "Add Store" button in My Team redirects to Add Promodizer page |
| [P4-F08](#bug-id-p4-f08) | Dropdown menus are not responsive on mobile and tablet view |
| [P4-F09](#bug-id-p4-f09) | "View Team" quick action loads with delay while other quick actions open instantly |
| [P4-F10](#bug-id-p4-f10) | Clear Filters button design is inconsistent compared to other modules |
| [P4-F11](#bug-id-p4-f11) | Success popup missing close (×) button after submitting sales |
| [P4-F12](#bug-id-p4-f12) | Pagination missing in Sales History despite showing more records available |
| [P4-F13](#bug-id-p4-f13) | Clear Filters button does not reset the date fields |


</details>

---

## Module Navigation (Jump to Any Section)


- **[Module 1: Management Portal Bugs](#module1-gtvl-management-portal-bugs)**
- **[Module 2: Promodizer Portal Bugs](#module2-gtvl-promodizer-portal-bugs)**
- **[Module 3: Store Performance Analysis Bugs](#module3-gtvl-store-performance-analysis-bugs)**
- **[Module 4: Supervisor Portal Bugs](#module4-gtvl-supervisor-portal-bugs)**
---
<h1 align="center">DAY-1</h1>

## Create PR

---
<h1 align="center">DAY-2</h1>

---

## Module1: gtvl-Management-Portal-bugs

---


## Bug ID: P1-F01

**Title:** Arrow icons inside dashboard boxes are not clickable

**Module:** Management Portal → Dashboard

**Description:**
- Each dashboard box opens correctly when the user clicks anywhere on the tile.  
- However, the arrow icon inside these boxes does not respond to clicks.  
- The icon appears interactive but performs no navigation or action.  
- This results in inconsistent behavior between the tile and the icon.  
- Users may incorrectly assume the arrow icon is broken or disabled.

**Expected Result:** Clicking the arrow icon should navigate to the appropriate module page.

**Actual Result:** Clicking the arrow icon does nothing.

**Evidence:**
- [P1-F01 Screenshot](evidence/screenshots/P1-F01_arrow_not_working_all_boxes.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P1-F02

**Title:** Edit, View, and Delete buttons do not work for several SKUs

**Module:** Management Portal → SKU Management → SKU List

**Description:**
- In the SKU list, some records respond correctly to Edit, View, and Delete actions.  
- However, many SKUs have non-responsive action buttons that perform no action.  
- The buttons appear visually active but do not trigger any navigation or functionality.  
- This inconsistency makes SKU management unreliable and confusing for users.  
- It prevents administrators from efficiently modifying or reviewing SKU details.

**Expected Result:** All SKUs should respond correctly to Edit, View, and Delete button actions.

**Actual Result:** Several SKUs do nothing when Edit, View, or Delete buttons are clicked.

**Evidence:**
- [P1-F02 Screenshot](evidence/screenshots/P1-F02_sku_buttons_not_working.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P1-F03

**Title:** System accepts duplicate SKU Codes without validation

**Module:** Management Portal → SKU Management → Add SKU

**Description:**
- When adding a new SKU, the form allows entering a SKU Code that already exists.  
- No validation message appears to warn the user about duplication.  
- The system saves multiple SKUs with the same code, causing data conflict.  
- Duplicate product identifiers affect product tracking and reporting accuracy.  
- This breaks the uniqueness requirement expected for SKU Codes.

**Expected Result:** The system should block duplicate SKU Codes and show a validation message.

**Actual Result:** Duplicate SKU Codes are accepted without any warning.

**Evidence:**
- [P1-F03 Screenshot](evidence/screenshots/P1-F03_duplicate_sku_code_allowed.png)
- [Return to Bug Index](#bug-index)


---

## Bug ID: P1-F04

**Title:** Export button in Store Management does not perform any action

**Module:** Management Portal → Store Management

**Description:**
- The Export button appears clickable but does not trigger any functionality.  
- No file is downloaded when the button is pressed.  
- The system gives no confirmation or error message, leaving the user confused.  
- This prevents users from exporting store data for offline use or reporting.  
- The feature is present visually but completely non-functional.

**Expected Result:** Clicking the Export button should download a CSV or Excel file containing store data.

**Actual Result:** No file is downloaded, and no action occurs on clicking Export.

**Evidence:**
- [P1-F04 Screenshot](evidence/screenshots/P1-F04_store_export_not_working.png)
- [Return to Bug Index](#bug-index)


---

## Bug ID: P1-F05

**Title:** Postal Code field accepts alphabets and special characters

**Module:** Management Portal → Store Management → Add Store

**Description:**
- The Postal Code input field allows users to enter alphabets and symbols.  
- There is no validation to restrict input to numeric characters only.  
- This results in invalid postal codes being saved in the system.  
- Incorrect data affects address accuracy and system reliability.  
- Proper input rules are missing for this mandatory field.

**Expected Result:** Postal Code should accept only numeric values.

**Actual Result:** Field accepts alphabets and special characters.

**Evidence:**
- [P1-F05 Screenshot](evidence/screenshots/P1-F05_store_postalcode_invalid_input.png)
- [Return to Bug Index](#bug-index)


---

## Bug ID: P1-F06

**Title:** Mobile Number field does not allow deleting or editing entered digits

**Module:** Management Portal → Store Management → Add Store

**Description:**
- After entering digits in the Mobile Number field, the input becomes locked.  
- Users cannot delete or modify the number using Backspace or Delete keys.  
- This forces users to refresh the entire page to correct mistakes.  
- The behavior breaks the basic input editing experience on forms.  
- It severely impacts usability when entering contact information.

**Expected Result:** Users should be able to freely delete or edit digits in the Mobile Number field.

**Actual Result:** Entered digits cannot be erased or modified.

**Evidence:**
- [P1-F06 Screenshot](evidence/screenshots/P1-F06_mobile_number_not_erasable.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P1-F07

**Title:** Edit, View, and Delete buttons do not work for any store

**Module:** Management Portal → Store Management → Store List

**Description:**
- The action buttons for each store (Edit, View, Delete) appear clickable but perform no actions.  
- Clicking any of these buttons fails to trigger navigation or open related pages.  
- This prevents users from managing or updating existing store records.  
- The entire store management workflow becomes unusable.  
- The UI suggests functionality that does not exist or is not wired correctly.

**Expected Result:** Edit, View, and Delete buttons should open the appropriate pages or actions.

**Actual Result:** All action buttons are unresponsive and do nothing.

**Evidence:**
- [P1-F07 Screenshot](evidence/screenshots/P1-F07_store_buttons_not_working.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P1-F08

**Title:** Supervisor First Name and Last Name fields accept numbers and duplicate entries

**Module:** Management Portal → Supervisor Management → Add New Supervisor

**Description:**
- The First Name and Last Name fields allow numeric values instead of restricting input to alphabets.  
- Users can enter invalid data such as digits or special characters in the name fields.  
- The system also accepts duplicate supervisor names without any validation.  
- This leads to inconsistent and unreliable supervisor records.  
- Proper data validation for names and uniqueness is completely missing.

**Expected Result:** Name fields should accept only alphabets and reject duplicate supervisor names.

**Actual Result:** Fields accept numbers, and duplicate names are saved without warning.

**Evidence:**
- [P1-F08 Screenshot](evidence/screenshots/P1-F08_supervisor_name_validation_issue.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P1-F09

**Title:** Promodizer Edit form allows changes but does not save updates

**Module:** Management Portal → Promodizer Management → Edit Promodizer

**Description:**
- When updating Promodizer details, the form accepts input changes normally.  
- However, clicking the Save button does not store the updated information.  
- After returning to the Promodizer list, the data remains unchanged.  
- No success message, validation, or error message is displayed.  
- This breaks the core functionality of editing Promodizer records.

**Expected Result:** Updated Promodizer details should be saved and reflected instantly.

**Actual Result:** Changes are not saved; the list continues to show old data.

**Evidence:**
- [P1-F09 Screenshot](evidence/screenshots/P1-F09_promodizer_update_not_saving.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P1-F10

**Title:** Promodizer First Name and Last Name fields incorrectly accept numeric input

**Module:** Management Portal → Promodizer Management → Add New Promodizer

**Description:**
- The First Name and Last Name fields allow users to enter numbers instead of restricting to alphabets.  
- No validation message appears when invalid characters are typed.  
- This allows submission of Promodizer records with invalid names.  
- It compromises data quality and consistency in the management system.  
- Mandatory fields lack proper validation rules for personal information.

**Expected Result:** The name fields should accept alphabets only and restrict numeric input.

**Actual Result:** Numeric values are accepted in both name fields without warning.

**Evidence:**
- [P1-F10 Screenshot](evidence/screenshots/P1-F10_promodizer_name_accepts_numbers.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P1-F11

**Title:** SKU remains visible after deletion until page refresh

**Module:** Management Portal → SKU Management

**Description:**
- After deleting an SKU, the record continues to appear in the table.
- The user must manually refresh the page to see updated changes.
- This creates confusion about whether the deletion succeeded.
- May cause repeated unnecessary delete attempts.

**Expected Result:** Deleted SKU should disappear immediately from the list.

**Actual Result:** SKU stays visible until manual page refresh.

**Evidence:**
- [P1-F11 Screenshot](evidence/screenshots/P1-F11_sku_visible_after_delete.gif)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P1-F12

**Title:** No success message after deleting SKU

**Module:** Management Portal → SKU Management

**Description:**
- When the user deletes an SKU, no confirmation popup or toast message is shown.
- The system provides no feedback about whether the deletion succeeded.
- This forces users to manually verify by refreshing the page.
- Creates uncertainty and poor user experience.

**Expected Result:** A “SKU deleted successfully” message should appear.

**Actual Result:** No success notification is displayed.

**Evidence:**
- [P1-F12 Screenshot](evidence/screenshots/P1-F12_no_delete_success_message.gif)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P1-F15

**Title:** Status label inconsistent (“active” vs “Active”)

**Module:** Management Portal → SKU List

**Description:**
- The status label appears in different formats across modules.
- Some places show lowercase “active”, others show “Active”.
- Inconsistent UI text formatting reduces professionalism.
- A unified style is required for coherence.

**Expected Result:** Status labels should follow a consistent capitalization format.

**Actual Result:** Status text varies between pages.

**Evidence:**
- [P1-F13 Screenshot](evidence/screenshots/P1-F13_status_inconsistent.gif)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P1-F14

**Title:** Import button uses wrong icon (Export icon shown)

**Module:** Management Portal → SKU Management

**Description:**
- The Import button incorrectly uses an export-style arrow icon.
- This causes confusion about the action’s actual purpose.
- Button icon mapping is incorrect and inconsistent with function.
- Creates misunderstanding among users.

**Expected Result:** Import button should show a correct import icon.

**Actual Result:** Export icon is displayed for Import.

**Evidence:**
- [P1-F14 Screenshot](evidence/screenshots/P1-F14_import_wrong_icon.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P1-F15

**Title:** Radio button shows extra square box when selected

**Module:** Management Portal → Add/Edit Forms

**Description:**
- When clicking the Active/Inactive radio option, an extra square box appears around the selected element.
- This looks like a CSS overflow or outline issue.
- The UI becomes visually inconsistent and distracting.
- Not aligned with the design system used across other modules.

**Expected Result:** Only the radio circle should highlight during selection.

**Actual Result:** Extra square highlight box appears.

**Evidence:**
- [P1-F15 Screenshot](evidence/screenshots/P1-F15_radio_square_issue.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P1-F16

**Title:** Chatbot icon overlaps action buttons

**Module:** Management Portal → Global UI Component

**Description:**
- The floating chatbot widget overlaps Edit/Delete buttons and pagination components.
- Users cannot click underlying elements unless they reposition the chatbot.
- This severely impacts usability and task completion flow.
- The bot should avoid covering critical interactive areas.

**Expected Result:** Chatbot icon should not obstruct functional elements.

**Actual Result:** Icon overlaps key UI controls.

**Evidence:**
- [P1-F16 Screenshot](evidence/screenshots/P1-F16_chatbot_overlapping.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P1-F17

**Title:** GTVL icon color mismatch across portals

**Module:** Management Portal / Promodizer Portal / Supervisor Portal

**Description:**
- The GTVL branding icon shows different color tones in each portal.
- This breaks visual consistency of branding.
- Users experience an inconsistent visual theme.
- Branding elements should remain uniform across all modules.

**Expected Result:** Same icon color across all portals.

**Actual Result:** Icon color varies between modules.

**Evidence:**
- [P1-F17 Screenshot](evidence/screenshots/P1-F17_gtvl_icon_mismatch.gif)
- [Return to Bug Index](#bug-index)

---

<h1 align="center">DAY-3</h1>

---

## Module2: GTVL-Promodizer-Portal-bugs 

---

## Bug ID: P2-F01

**Title:** Home page icons are not properly aligned across Promodizer sections

**Module:** Promodizer Portal → Home / Record Sales / Sales History / Attendance

**Description:**
- The icons displayed in the Promodizer Portal sections are not consistently aligned.  
- Different pages show misaligned icons with uneven spacing and positioning.  
- The layout appears visually unbalanced, reducing UI clarity and professionalism.  
- Users may get confused due to inconsistent design patterns across sections.  
- This affects the visual quality and standardization of the portal’s interface.

**Expected Result:** All icons across Promodizer sections should be uniformly aligned.

**Actual Result:** Icons appear misaligned and inconsistent on multiple pages.

**Evidence:**
- [P2-F01 Screenshot](evidence/screenshots/P2-F01_promodizer_icon_alignment_issue.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P2-F02

**Title:** Recorded sales do not appear in the Sales History section

**Module:** Promodizer Portal → Record Sales / Sales History

**Description:**
- When a sale is recorded by scanning a barcode and submitting the quantity, the entry is not saved.  
- The Sales History section fails to display recently recorded sales.  
- No confirmation message appears after submitting a sale, leaving users unsure whether it succeeded.  
- This breaks a critical workflow for tracking Promodizer performance.  
- The absence of sales records makes sales monitoring impossible.

**Expected Result:** Submitted sales should immediately appear in the Sales History list.

**Actual Result:** No recorded sales appear in the Sales History section.

**Evidence:**
- [P2-F02 Screenshot](evidence/screenshots/P2-F02_sales_not_visible_in_history.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P2-F03

**Title:** Feedback form does not submit and shows an error message

**Module:** Promodizer Portal → Feedback Form

**Description:**
- When the user fills out the feedback form and submits it, the system fails to process the request.  
- An error message “Failed to submit feedback. Please try again.” appears every time.  
- No data is stored or transmitted despite correct input from the user.  
- This prevents Promodizers from reporting issues or providing feedback.  
- The form appears functional but is completely non-operational.

**Expected Result:** Feedback should be successfully submitted and a confirmation message should appear.

**Actual Result:** The system displays an error and no feedback is submitted.

**Evidence:**
- [P2-F03 Screenshot](evidence/screenshots/P2-F03_feedback_form_not_submitting.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P2-F04

**Title:** Sidebar menu does not highlight the selected option

**Module:** Promodizer Portal → Sidebar Navigation

**Description:**
- The sidebar menu highlights only the Dashboard option, regardless of which section the user opens.  
- When navigating to other sections such as Sales History, Attendance, or My Stores, the highlight does not change.  
- This causes confusion as users cannot identify which section they are currently viewing.  
- The inactive highlight logic breaks expected UI navigation behavior.  
- The user experience becomes inconsistent and misleading.

**Expected Result:** The currently active menu item should be highlighted based on the page opened.

**Actual Result:** Only the Dashboard option remains highlighted at all times.

**Evidence:**
- [P2-F04 Video](evidence/screenshots/P2-F04_sidebar_highlight_issue.gif)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P2-F05

**Title:** Calendar dates are misaligned under incorrect weekday columns

**Module:** Promodizer Portal → Attendance → Calendar

**Description:**
- The calendar layout displays several dates under incorrect weekday labels such as Sun, Mon, Tue, etc.  
- Some dates are shifted, overlapping, or placed outside their proper cells.  
- Attendance markers (dots) also appear in wrong positions, making the calendar difficult to understand.  
- This breaks the structure of the attendance calendar and reduces readability.  
- Users cannot correctly track attendance due to visual misalignment.

**Expected Result:** All dates and markers should appear under their correct weekday columns with proper alignment.

**Actual Result:** Dates appear misaligned and out of position across the calendar grid.

**Evidence:**
- [P2-F05 Screenshot](evidence/screenshots/P2-F05_calendar_date_alignment_issue.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P2-F06

**Title:** My Profile page alignment is inconsistent compared to other sections

**Module:** Promodizer Portal → My Profile

**Description:**
- While most sections in the Promodizer Portal are centered or evenly spaced, the My Profile page is misaligned.  
- The content appears shifted toward the left side instead of staying centered like other modules.  
- This inconsistent layout creates a visually unbalanced appearance.  
- Users may get confused due to the sudden change in structure and spacing.  
- The UI lacks uniformity across different sections of the portal.

**Expected Result:** My Profile layout should be centered and aligned consistently with other pages.

**Actual Result:** Content on the My Profile page appears left-aligned and visually uneven.

**Evidence:**
- [P2-F06 Video](evidence/screenshots/P2-F06_profile_alignment_issue.gif)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P2-F07

**Title:** Barcode field allows longer pasted input than typed input  

**Module:** Promodizer Portal → Record Sales  

**Description:**  
- Typing is restricted to 7 characters in the barcode field.
- But pasting allows 8+ characters, bypassing validation.
- This inconsistency allows invalid barcodes to be submitted.

**Expected Result:** Field should enforce the same character limit for typing and pasting.

**Actual Result:** Pasting bypasses the character limit.

**Evidence:**  
- [P2-F07 Screenshot](evidence/screenshots/P2-F07_barcode_paste_limit_issue.gif)  
- [Return to Bug Index](#bug-index)

---

## Bug ID: P2-F08

**Title:** Attendance History displays attendance only on Sundays  

**Module:** Promodizer Portal → Attendance  

**Description:**  
- Attendance entries appear only on Sundays.
- All other days remain blank even after attendance is marked.
- Incorrect date rendering affects attendance accuracy.

**Expected Result:** Attendance records should display on correct dates.

**Actual Result:** Attendance shows only on Sundays; other days empty.

**Evidence:**  
- [P2-F08 Screenshot](evidence/screenshots/P2-F08_attendance_only_sundays.png)  
- [Return to Bug Index](#bug-index)

---

## Bug ID: P2-F09

**Title:** Dashboard icons not following theme (inconsistent colors)

**Module:** Promodizer Portal → Dashboard

**Description:**
- The Dashboard tiles use inconsistent icon colors.
- Some icons appear green, while others turn grey on hover.
- The theme does not match the system’s UI style used in other portals.
- Creates a lack of visual cohesion and inconsistent brand feel.

**Expected Result:** Dashboard icons should follow a consistent style and color theme.

**Actual Result:** Icons display mismatched colors and hover styles.

**Evidence:**
- [P2-F09 Screenshot](evidence/screenshots/P2-F09_dashboard_icon_inconsistent.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P2-F10

**Title:** Dropdown misaligned in mobile view

**Module:** Promodizer Portal → All Dropdowns

**Description:**
- In mobile layout, dropdown menus appear outside the visible container.
- The dropdown list extends off-screen, making items hard or impossible to select.
- UI scaling issues cause fields to overlap other elements.
- This severely impacts usability on smaller screens.

**Expected Result:** Dropdowns should resize and align correctly on mobile devices.

**Actual Result:** Dropdown lists appear cut off and misaligned in mobile view.

**Evidence:**
- [P2-F10 Screenshot](evidence/screenshots/P2-F10_dropdown_mobile_misaligned.png)
- [Return to Bug Index](#bug-index)

---

## Module3: gtvl-Store-Performance-Analysis-bugs

---

## Bug ID: P3-F01

**Title:** SKU Performance table and filter panel are misaligned

**Module:** Store Performance Analysis → SKU Performance

**Description:**
- The filter panel on the left and the SKU table on the right are not aligned properly.  
- The “Max” quantity filter field is partially hidden beneath another UI container.  
- Column headers like “SKU CODE” do not align with the actual data rows.  
- This misalignment makes the interface look broken and difficult to read.  
- Users may miss important information due to layout issues.

**Expected Result:** The filter panel and table should be properly aligned with all fields visible.

**Actual Result:** UI elements overlap and the Max field is partially hidden.

**Evidence:**
- [P3-F01 Screenshot](evidence/screenshots/P3-F01_sku_performance_alignment_issue.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P3-F02

**Title:** Min and Max quantity filter fields accept negative values

**Module:** Store Performance Analysis → SKU Performance → Filters

**Description:**
- The Min and Max input fields in the Total Quantity Sold filter allow negative numbers.  
- The system does not validate or restrict the input to positive values only.  
- This results in invalid filter conditions that affect analysis accuracy.  
- Users may unintentionally enter incorrect ranges that produce misleading results.  
- Proper validation for numeric fields is missing entirely.

**Expected Result:** The filter should accept only non-negative values.

**Actual Result:** Negative values can be entered freely without any validation.

**Evidence:**
- [P3-F02 Screenshot](evidence/screenshots/P3-F02_negative_values_in_quantity_filter.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P3-F03

**Title:** Active filter does not reset and combined selection shows incorrect results

**Module:** Store Performance Analysis → SKU Performance → Filters

**Description:**
- When clicking “Clear All,” the Active status checkbox remains selected instead of resetting.  
- Selecting both Active and Inactive filters returns only Inactive results instead of both.  
- The filter logic behaves inconsistently and does not reflect actual selected conditions.  
- Users are misled by incorrect output, affecting decision-making during analysis.  
- This breaks expected filter functionality and result accuracy.

**Expected Result:** Clearing filters should reset all options, and selecting both statuses should show all SKUs.

**Actual Result:** Active remains selected after clearing, and combined selection shows only Inactive results.

**Evidence:**
- [P3-F03 Video](evidence/screenshots/P3-F03_status_filter_issue.gif)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P3-F04

**Title:** Custom date range option does not display Start and End date fields

**Module:** Store Performance Analysis → SKU Performance → Date Range Filter

**Description:**
- When the user selects the Custom Range option from the date filter, no date inputs appear.  
- The interface fails to show Start Date and End Date selection fields.  
- Users cannot choose a manual date range, making the filter option unusable.  
- This reduces flexibility for targeted data analysis within specific periods.  
- The UI wrongly suggests availability of a feature that is non-functional.

**Expected Result:** Selecting Custom Range should display Start Date and End Date fields.

**Actual Result:** No date input fields appear after selecting Custom Range.

**Evidence:**
- [P3-F04 Screenshot](evidence/screenshots/P3-F04_custom_date_range_not_visible.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P3-F05

**Title:** Table alignment issues cause some records to be partially hidden

**Module:** Store Performance Analysis → Performance Table

**Description:**
- Several rows and columns in the performance table are misaligned.  
- Some records appear cut off or partially hidden within the table layout.  
- Scrolling does not correct the visibility issue, leaving data unreadable.  
- This disrupts the user’s ability to analyze or compare store performance accurately.  
- The table structure lacks proper formatting and spacing.

**Expected Result:** All table rows and columns should be properly aligned and fully visible.

**Actual Result:** Record entries appear misaligned or cut off in the table.

**Evidence:**
- [P3-F05 Screenshot](evidence/screenshots/P3-F05_table_alignment_issue.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P3-F06

**Title:** Filter fields accept negative values in the Store Performance section

**Module:** Store Performance Analysis → Filters

**Description:**
- The numeric filter fields allow users to enter negative numbers without restriction.  
- This results in invalid filter ranges that distort the output of performance data.  
- There is no validation to prevent incorrect or logically impossible inputs.  
- Users may unintentionally apply wrong filters, affecting data interpretation.  
- The absence of validation reduces the reliability of analytical results.

**Expected Result:** Filter fields should accept only valid positive numeric values.

**Actual Result:** Negative values are accepted without any validation or error message.

**Evidence:**
- [P3-F06 Screenshot](evidence/screenshots/P3-F06_negative_filter_values.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P3-F07

**Title:** Filter tag close (×) button does not remove the applied filter

**Module:** Store Performance Analysis → Staff Performance → Filters

**Description:**
- After applying a date range filter, a filter tag appears at the top of the table.  
- When clicking the close (×) button on the tag, the filter does not get removed.  
- The tag remains visible, and the filter stays active despite the user action.  
- This prevents users from clearing specific filters without resetting everything.  
- The behavior breaks expected filter interaction functionality.

**Expected Result:** Clicking the (×) button should remove the applied date filter.

**Actual Result:** Clicking the (×) does nothing; the filter tag remains active.

**Evidence:**
- [P3-F07 Screenshot](evidence/screenshots/P3-F07_filter_tag_cross_not_working.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P3-F08

**Title:** Clicking any field in the Staff Performance table redirects to “Staff Not Found” page

**Module:** Store Performance Analysis → Staff Performance Table

**Description:**
- Every field inside the Staff Performance table behaves like a clickable link.  
- Clicking any row or column redirects the user to a page showing “Staff Not Found.”  
- No valid staff details are loaded on the redirected page.  
- This breaks the staff review functionality and confuses users.  
- Users are unable to view or validate performance details for individual staff members.

**Expected Result:** Clicking a staff row should open correct staff details or remain non-interactive if not intended.

**Actual Result:** Clicking any table field redirects to an incorrect “Staff Not Found” page.

**Evidence:**
- [P3-F08 Screenshot 1](evidence/screenshots/P3-F08_staff_performance_table.png)  
- [P3-F08 Screenshot 2](evidence/screenshots/P3-F08_staff_not_found.png)
- [Return to Bug Index](#bug-index)

---

<h1 align="center">DAY-4</h1>

---

## Bug ID: P3-F09

**Title:** Clear All button position is inconsistent in Transaction History module

**Module:** Sales Analytics Dashboard → Transaction History

**Description:**
- In other analytics modules, the Clear All button appears below the filters near Apply Filters.
- In Transaction History, the Clear All button is placed at the top of the filter panel.
- This creates UI inconsistency across sales analytics sections.
- Users may get confused due to different filter layouts in similar modules.

**Expected Result:** Clear All button should appear in a consistent position across all analytics modules.

**Actual Result:** Transaction History places Clear All at the top, while other modules place it at the bottom.

**Evidence:**
- [P3-F09 Screenshot 1](evidence/screenshots/P3-F09_clear_all_position_top.png)
- [P3-F09 Screenshot 2](evidence/screenshots/P3-F09_clear_all_position_bottom.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P3-F10

**Title:** Transaction History table has inconsistent header styling and missing sorting arrows

**Module:** Sales Analytics Dashboard → Transaction History → Table

**Description:**
- Most table headers use normal styling and include sorting arrows for ascending/descending order.
- However, the "Role" and "Submission ID" columns appear in bold, unlike other headers.
- These two columns also do not show sorting arrows like the rest.
- This creates inconsistency in table design and sorting functionality.

**Expected Result:** All table headers should follow the same style and include sorting arrows where supported.

**Actual Result:** "Role" and "Submission ID" headers are bold and lack sorting arrows, unlike other columns.

**Evidence:**
- [P3-F10 Screenshot](evidence/screenshots/P3-F10_inconsistent_table_headers.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P3-F11

**Title:** Store filter dropdown displays duplicate store names

**Module:** Store Performance Analysis → SKU Performance / Transaction History → Filters

**Description:**
- The Store dropdown in the filter section displays the same store name multiple times.
- Duplicate entries appear even though each store should be listed only once.
- This causes confusion when selecting a store for filtering analytics data.
- The issue indicates incorrect mapping or repeated data population in the dropdown.
- Users may unintentionally choose the wrong store due to duplicate options.

**Expected Result:** Store dropdown should list each store only once without duplicates.

**Actual Result:** Store dropdown shows the same store name multiple times.

**Evidence:**
- [P3-F09 Screenshot](evidence/screenshots/P3-F11_duplicate_store_dropdown.png)
- [Return to Bug Index](#bug-index)

---


## Bug ID: P3-F12

**Title:** Sorting icons in Store Performance table are inconsistent in size and alignment

**Module:** Sales Analytics Dashboard → Store Performance → Table Header

**Description:**
- The sorting icons (↑↓) displayed in the table headers such as **City**, **Total Quantity Sold**, **Total Transactions**, and **SKUs Sold** are not uniform in size.
- Some sorting icons appear smaller, while others appear larger or misaligned compared to the rest.
- This creates a visually inconsistent table header and affects UI clarity.
- The inconsistent icon size makes the sorting controls look unbalanced and reduces readability.

**Expected Result:** All sorting icons should be displayed in a uniform size, style, and alignment across all table columns.

**Actual Result:** Sorting icons differ in size and alignment across multiple header columns, causing uneven UI presentation.

**Evidence:**
- [P3-F12 Screenshot](evidence/screenshots/P3-F12_sorting_icon_inconsistent.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P3-F13

**Title:** Date filter accepts invalid date ranges (From Date > To Date)  

**Module:** Sales Analytics Dashboard → Date Filters  

**Description:**  
- User can select a From Date later than the To Date.
- System does not warn or block invalid ranges.
- This results in incorrect filtering and empty data.

**Expected Result:** System should validate dates and block invalid ranges.

**Actual Result:** Invalid date ranges are accepted without error.

**Evidence:**  
- [P3-F13 Screenshot](evidence/screenshots/P3-F13_invalid_date_range.png)  
- [Return to Bug Index](#bug-index)

---

## Bug ID: P3-F14

**Title:** Sorting icon not visible until hover and appears in inconsistent shape  

**Module:** Sales Analytics Dashboard → SKU Performance → Table Header  

**Description:**  
- Sorting icons (↑↓) for columns in the SKU Performance table are **not visible by default**.  
- The icons appear **only when hovering** over the column header.  
- When they appear on hover, the icons have a **different shape and style** compared to other tables in the system.  
- This creates UI inconsistency and makes it unclear to the user that sorting is available.  
- Users may assume sorting is not supported because the icon is hidden initially.

**Expected Result:**  
Sorting icons should be consistently visible across all sortable columns and maintain a uniform style.

**Actual Result:**  
Sorting icons remain hidden until hover and appear in a different shape/style when displayed.

**Evidence:**  
- [P3-F14 Screenshot](evidence/screenshots/P3-F14_sorting_icon_hover_issue.gif)  
- [Return to Bug Index](#bug-index)

---

## Bug ID: P3-F15

**Title:** Store Performance chart not visible on desktop view

**Module:** Sales Analytics Dashboard → Store Performance → Chart View

**Description:**
- The Store Performance chart does not load or render on desktop screens.
- The chart area remains blank even after clicking multiple times.
- The same chart loads correctly on mobile view, but not on desktop.
- This breaks the analytics workflow and prevents desktop users from visual data analysis.

**Expected Result:** The Store Performance chart should load properly on desktop and mobile devices.

**Actual Result:** The chart appears only on mobile view and remains invisible on desktop.

**Evidence:**
- [P3-F15 Screenshot](evidence/screenshots/P3-F15_chart_not_visible_desktop.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P3-F16

**Title:** Transaction History table fluctuates during sorting

**Module:** Sales Analytics Dashboard → Transaction History → Table

**Description:**
- When clicking the sorting icons in any column, the entire table layout jumps or fluctuates.
- The headers and rows shift position temporarily.
- This creates a distracting and unstable UI experience.
- Sorting should smoothly reorder rows without shaking the table layout.

**Expected Result:** Table should remain stable and only reorder rows during sorting.

**Actual Result:** Table layout visibly fluctuates each time sorting is applied.

**Evidence:**
- [P3-F16 Screenshot](evidence/screenshots/P3-F16_table_fluctuates_on_sorting.gif)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P3-F17

**Title:** SKU Performance sorting icon behavior not standard (visible only on click)

**Module:** Sales Analytics Dashboard → SKU Performance → Table

**Description:**
- Sorting icons are not visible by default in the table headers.
- Icons appear only after clicking the header instead of being always visible.
- This behavior is inconsistent with other tables in the system.
- Users may assume sorting is not available due to hidden icons.

**Expected Result:** Sorting icons should be consistently visible across all sortable columns.

**Actual Result:** Icons remain hidden until clicked and use a different style compared to other modules.

**Evidence:**
- [P3-F17 Screenshot](evidence/screenshots/P3-F17_sorting_icon_visible_on_click.gif)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P3-F18

**Title:** Supervisor Assignment Stores missing labels & checkboxes

**Module:** Sales Analytics Dashboard → Store Performance → Assignment Stores

**Description:**
- The Assignment Stores section contains no checkbox UI elements.
- The search bar also does not include a title or label.
- Other filter sections contain consistent labels and checkbox styling.
- This inconsistency creates confusion and makes the section appear incomplete.

**Expected Result:** Assignment Stores should include labeled checkboxes matching project UI standards.

**Actual Result:** Label text is missing and checkboxes are not displayed.

**Evidence:**
- [P3-F18 Screenshot](evidence/screenshots/P3-F18_assignment_store_missing_labels.png)
- [Return to Bug Index](#bug-index)

---

## Module4: GTVL-Supervisor-Portal-bugs

---

## Bug ID: P4-F01

**Title:** Dashboard does not update attendance status after marking attendance

**Module:** Supervisor Portal → Attendance / Dashboard

**Description:**
- When a promodizer's attendance is marked as Present in the Attendance section, the update is saved correctly.
- However, the Supervisor Dashboard still shows “No attendance record for today.”
- The dashboard does not refresh or reflect the latest attendance status.
- This creates inconsistency between the Attendance module and Dashboard view.

**Expected Result:** Dashboard should immediately update and display today’s attendance record after marking attendance.

**Actual Result:** Dashboard continues showing “No attendance record for today” even after attendance is marked.

**Evidence:**
- [P4-F01 Screenshot](evidence/screenshots/P4-F01_dashboard_attendance_not_updated.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P4-F02

**Title:** Clock Out Time option is missing in the Attendance section

**Module:** Supervisor Portal → Attendance

**Description:**
- The Attendance section displays two columns: “Clock In Time” and “Clock Out Time.”
- While Clock In Time is available and visible, there is no input or button for Clock Out Time.
- The system only allows marking attendance but does not provide a way to record clock out.
- This makes attendance incomplete and prevents tracking full working hours.

**Expected Result:** Attendance section should provide a functional option for “Clock Out Time” alongside Clock In.

**Actual Result:** Clock Out Time column is shown in the table header but no option exists to mark or enter it.

**Evidence:**
- [P4-F02 Screenshot](evidence/screenshots/P4-F02_clockout_missing.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P4-F03

**Title:** Mobile Number field accepts alphabets and special characters

**Module:** Supervisor Portal → Add Promodizer → Personal Details

**Description:**
- The Mobile Number input field allows users to enter alphabets, symbols, and other non-numeric characters.
- There is no validation to restrict the field to digits only.
- Invalid data can be saved, leading to incorrect or unusable contact information.
- This breaks basic form validation rules expected for phone number fields.

**Expected Result:** Mobile Number field should accept only numeric digits with proper validation.

**Actual Result:** Field accepts alphabets, symbols, and invalid characters without any error.

**Evidence:**
- [P4-F03 Screenshot](evidence/screenshots/P4-F03_invalid_mobile_input.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P4-F04

**Title:** Sales History selection auto-redirects to record details page and stuck on loading

**Module:** Supervisor Portal → Sales History

**Description:**
- When selecting any sale entry from the Sales History list, the system automatically redirects to the Sale Record Details page.
- The redirected page shows a loading/buffering state and does not display any sale details.
- No data is visible, and the page remains stuck without loading the selected record.
- This interrupts supervision workflow and prevents reviewing submitted sales.

**Expected Result:** Selecting a sale entry should open the Sale Record Details page and display full sale information.

**Actual Result:** System redirects to details page but stays stuck on loading and shows no data.

**Evidence:**
- [P4-F04 Video](evidence/screenshots/P4-F04_sales_history_redirect_issue.gif)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P4-F05

**Title:** Cancel button does not navigate back when form fields are empty

**Module:** Supervisor Portal → Add New Supervisor

**Description:**
- When the user fills some information and clicks the Cancel button, a confirmation popup appears correctly.
- If all fields are cleared or left blank and the user clicks Cancel, no popup appears.
- The system does not navigate back or close the form when fields are empty.
- This results in inconsistent Cancel button behavior and prevents exiting the form without manually navigating.

**Expected Result:** Cancel button should work consistently and navigate back regardless of whether fields are filled or empty.

**Actual Result:** Cancel button works only when fields contain values; does nothing when fields are empty.

**Evidence:**
- [P4-F05 Video](evidence/screenshots/P4-F05_cancel_button_not_working_empty_form.gif)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P4-F06

**Title:** Two Cancel buttons are displayed and both perform the same action

**Module:** Supervisor Portal → Add Promodizer

**Description:**
- The Add Promodizer form displays two separate Cancel buttons on the same page.
- Both Cancel buttons trigger the same action and perform identical behavior.
- This creates UI redundancy and may confuse users about which button to use.
- It also indicates improper layout structure or duplicated elements in the form design.

**Expected Result:** Only one Cancel button should be shown in the form for a clear and consistent user experience.

**Actual Result:** Two Cancel buttons appear, both performing the same function.

**Evidence:**
- [P4-F06 Screenshot](evidence/screenshots/P4-F06_duplicate_cancel_buttons.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P4-F07

**Title:** "Add Store" button in My Team redirects to Add Promodizer page

**Module:** Supervisor Portal → My Team

**Description:**
- In the My Team section, the UI displays an “Add Store” button meant for assigning or adding a new store.
- When the user clicks this button, it incorrectly redirects to the Add Promodizer form instead of the Add Store page.
- This causes confusion and prevents supervisors from performing store-related actions.
- The button is mapped to the wrong page or incorrect routing is implemented.

**Expected Result:** Clicking “Add Store” should open the Add Store page or the correct store assignment form.

**Actual Result:** Button redirects to the Add Promodizer window instead of the Add Store screen.

**Evidence:**
- [P4-F07 Video](evidence/screenshots/P4-F07_addstore_redirects_to_addpromodizer.gif)
- [Return to Bug Index](#bug-index)


---

## Bug ID: P4-F08

**Title:** Dropdown menus are not responsive on mobile and tablet view

**Module:** Supervisor Portal → All Dropdown Components (Sales History, Record Sales, My Team, etc.)

**Description:**
- When viewing the Supervisor Portal on mobile or tablet device resolutions, dropdown menus do not resize or align properly.
- Dropdown lists appear cut off, shifted, or partially hidden outside the viewport.
- Users cannot select options properly due to broken alignment and unresponsive dropdown behavior.
- This affects multiple dropdown components across modules such as Sales History, Record Sales, Attendance, and My Team.

**Expected Result:** Dropdown menus should be fully responsive and display correctly on mobile and tablet screen sizes.

**Actual Result:** Dropdowns appear misaligned, partially hidden, and become difficult or impossible to use on mobile/tablet.

**Evidence:**
- [P4-F08 Screenshot](evidence/screenshots/P4-F08_dropdown_not_responsive_mobile.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P4-F09

**Title:** "View Team" quick action loads with delay while other quick actions open instantly

**Module:** Supervisor Portal → Dashboard → Quick Actions

**Description:**
- Quick action buttons such as **Record Sales**, **Mark Attendance**, and **Sales History** open instantly and display information immediately.
- However, clicking the **View Team** button results in a noticeable delay before any content becomes visible.
- The page remains blank or stuck for a few seconds before loading team details.
- This creates inconsistency in quick action performance and negatively affects user experience.

**Expected Result:** "View Team" quick action should open instantly like other quick actions.

**Actual Result:** "View Team" takes several seconds to load and shows no content initially.

**Evidence:**
- [P4-F09 Screenshot](evidence/screenshots/P4-F09_view_team_slow_loading.gif)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P4-F10

**Title:** Clear Filters button design is inconsistent compared to other modules

**Module:** Supervisor Portal → Sales History → Filters

**Description:**
- The Clear Filters button in the Sales History section appears as plain text without any styled button background.
- In other modules of the ERP system, Clear Filters appears as a properly styled button with a filled or bordered design.
- This visual inconsistency makes the Sales History Clear Filters option less noticeable and harder to identify as a button.
- The inconsistent UI element reduces usability and breaks uniform design standards.

**Expected Result:** Clear Filters should be displayed as a styled button similar to other modules for consistent UI/UX.

**Actual Result:** Clear Filters appears as plain unstyled text, unlike the properly designed Clear Filters buttons elsewhere.

**Evidence:**
- [P4-F10 Screenshot](evidence/screenshots/P4-F10_clear_filter_inconsistent_style.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P4-F11

**Title:** Success popup missing close (×) button after submitting sales

**Module:** Supervisor Portal → Record Sales → Submission Success Popup

**Description:**
- After submitting a sales batch successfully, a confirmation popup appears showing the Submission ID, items recorded, store, and timestamp.
- However, the popup does not include a close (×) icon in the top-right corner.
- Users cannot dismiss or exit the popup quickly using a standard close button.
- This breaks expected modal UI behavior and reduces ease of navigation.
- Lack of a close icon may confuse users who expect a quick and accessible way to exit modal windows.

**Expected Result:** A close (×) button should appear on the popup to allow users to dismiss it instantly.

**Actual Result:** No close (×) icon is available; users must use the action buttons at the bottom of the popup.

**Evidence:**
- [P4-F11 Screenshot](evidence/screenshots/P4-F11_success_popup_no_close_button.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P4-F12

**Title:** Pagination missing in Sales History despite showing more records available

**Module:** Supervisor Portal → Sales History

**Description:**
- The Sales History page displays “Showing 6 of 9 records,” indicating additional entries exist.
- However, no pagination controls (Next, Previous, page numbers) appear on the page.
- Users cannot view the remaining records beyond the first 6.
- This prevents supervisors from accessing complete historical data.
- The UI incorrectly suggests full visibility while hiding additional pages.

**Expected Result:** Pagination controls should appear to allow navigation to all available records.

**Actual Result:** Only the first 6 records are visible, and no pagination controls are provided.

Evidence:
- [P4-F12 Screenshot](evidence/screenshots/P4-F12_missing_pagination_sales_history.png)
- [Return to Bug Index](#bug-index)

---

## Bug ID: P4-F13

**Title:** Clear Filters button does not reset the date fields  

**Module:** Supervisor Portal → Sales History → Filters  

**Description:**  
- When the user clicks the **Clear Filters** button, the **From Date** and **To Date** fields do not reset.  
- Other filter fields may clear, but the date fields remain unchanged.  
- This prevents users from fully resetting the filter criteria with one action.  
- The behavior causes confusion because the UI suggests all filters should be cleared.  
- Users must manually remove the dates, which breaks expected filter usability.

**Expected Result:** Clicking **Clear Filters** should clear all filter fields, including From Date and To Date.  

**Actual Result:** Date fields remain filled even after clicking Clear Filters.  

**Evidence:**  
- [P4-F13 Screenshot](evidence/screenshots/P4-F13_clear_filter_not_resetting_dates.png)  
- [Return to Bug Index](#bug-index)

---

