# School Data Importer — targeted fixes after v11

A compact Core PHP + SQLite school records importer and entry tool. The application remains a single `index.php` file; there is no `config.php` in this revision because configuration centralization was postponed.

## Files
- `index.php` — application, API handlers, inline configuration, imports/exports, manual entry, result entry, and photo cropping.
- `data/` — SQLite database directory. Keep the existing `school.sqlite` when upgrading.
- `uploads/students/` — student photos. Keep existing photos when upgrading.
- `data/` and `uploads/students/` are created automatically when needed; keep these folders and their contents when upgrading.

## Deploy / upgrade
1. Back up your existing `data/school.sqlite`, `uploads/`, and current `index.php` before replacing files.
2. Upload the updated `index.php` and retain your existing `data/` and `uploads/` directories. Do not delete the database or student photos.
3. Use PHP 8.1+ with `pdo_sqlite` enabled. PHP must be able to write to the database and student photo directories.
4. Open the application and verify Students, Enrollment, all three Result Entry modes, CSV/XLSX import, CSV export, edit/update, and photo crop.

## Configuration for future changes
For now, changeable values remain in the `$cfg=(object)[ ... ]` block near the top of `index.php`. This is intentionally not split into `config.php` in this revision. Settings include:
- Current academic session and allowed baseline sessions
- Classes and sections
- Exam names and default maximum marks
- Subject names and class-based subject groups
- School names for primary and secondary classes
- Student photo output dimensions, maximum upload size, and cropper width

Change settings carefully and keep them consistent with existing records. Changing the current session does not migrate historical enrollments or results.

## Deferred work — not implemented in this revision
The following items are deliberately postponed and are documented here for a later finalization pass:

### `config.php`
Move editable school/session/class/subject/exam/photo/path settings into a dedicated `config.php`, document each setting, and ensure no secrets or private values are exposed to the browser.

### Database and deployment security
- Review whether the SQLite database can be moved outside the public web root on the target hosting plan; otherwise verify that direct HTTP access is blocked.
- Review file and directory permissions for the database, photo uploads, and backups.
- Protect database backups and test a restore procedure.
- Review administrative access control, HTTPS, and server-side validation before public production use.

These are future tasks; this revision does not claim that the database security review is complete.

## Behavior and regression notes
- Import is a two-step flow: upload/edit/select rows, then map fields and import. Rows are selected by default; Select All and individual row checkboxes are available. Missing required mappings are blocked. Missing required values are highlighted in the preview; the user can correct them or explicitly confirm importing only valid rows while skipping invalid rows.
- The separate Manual Entry / Import tabs remain postponed. Existing layout is retained.
- Roll number is optional in Student + Enrollment and Enrollment entry; blank roll numbers are saved as `NULL`.
- The first bulk-entry row cannot be manually removed. Additional rows have Remove buttons; empty rows remove immediately and populated rows require confirmation. Fully empty rows are ignored on save; partially populated rows are validated.
- Students list shows permanent student details plus enrollment status for the globally selected session. Active students without an enrollment remain visible and have an Enroll action. Enroll opens the existing student/enrollment form with the student preselected; select the class and section, and assign a roll number only if available. Students imported without a class can be enrolled later without re-importing their permanent details.
- Result entry defaults to Class-wise · Single Subject when opened and supports Class-wise · Single Subject, Class-wise · All Subjects, and Student-wise · All Subjects. Blank marks remain unchanged; zero is valid; marks are checked against maximum marks. In Class-wise · All Subjects, “Same maximum marks for all subjects” is checked by default and applies one common maximum for that operation; unchecking it reveals individual subject maximums. Student-wise entry retains per-subject maximums.
- Class-based subjects remain: Classes 1–3 use EVS; Classes 4–8 use Science, S.St and Computer; Classes 9–10 use Information Technology.
- Existing database records and student photos must be retained when upgrading.

## Validation performed
- PHP syntax check (`php -l index.php`): passed.
- Inline JavaScript syntax check (`node --check`): passed.
- Full live PHP + SQLite browser/API integration testing could not be run in this build environment because its PHP CLI does not have the PDO SQLite extension enabled. Run the verification checklist above on the target hosting account before production use.

## Recent UI adjustments

- Result records are grouped into one row per student, exam, session, class, and section, with all saved subject marks summarized together. The database still stores one record per subject. **Edit marks** opens the student-wise result workflow; **Delete exam marks** deletes that student's marks for the selected exam and session after confirmation.
- After a successful import, the import panel resets to step 1 and clears the previous preview/mapping so the mapping fields do not remain visible after completion.
- The public URL `https://web.isatish.in/nps-result15/` could not be fetched by the analysis browser in this run, so this revision was reviewed from the supplied screenshot and application source rather than a live remote session.
