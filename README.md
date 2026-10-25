# SmartPass — Campus Permission & Attendance (Tomcat 9)

SmartPass is a Java Servlet/JSP/JDBC/MySQL lab application for student permission requests, branch-level HOD approval, automatic class-teacher routing, and attendance. It uses `javax.servlet` and is intended for Apache Tomcat 9 in Eclipse.

## Campus model

Supported programmes:

- B.Tech branches: `CSE`, `AIML`, `AIDS`, `IT`, `ECE`, `MECH`, `CIVIL`, `AEROSPACE`.
- B.Tech years: 1–4.
- Each configured B.Tech class can have 2–8 sections, starting at A. For example, count 2 creates A–B, count 5 creates A–E, and count 8 creates A–H.
- MBA is provisionally set to years 1–2 with 2 sections per year. Confirm and edit this if your campus MBA structure differs.

### IMPORTANT: configure your actual section counts

Open `src/main/java/edu/smartpass/util/CampusConfig.java`. The starter configuration currently uses two sections per branch/year because the actual section count for every class was not provided. Change each number to your real campus count (any number from 2 to 8) before seeding the accounts. For example:

```java
putBranch(counts, "CSE",       8, 8, 6, 4); // EXAMPLE ONLY; replace with your actual counts
putBranch(counts, "AIML",      4, 4, 3, 2); // EXAMPLE ONLY; replace with your actual counts
```

The four values are Years 1, 2, 3 and 4, in that order. The numbers above are examples only, not claims about your college. Repeat this for each branch. MBA uses the two `counts.put(key("MBA", year), count)` entries. Counts must be 2–8.

After editing the configuration, run the SQL script (it adds the `campus_classes` table without dropping your existing tables), redeploy the project, and visit `/setup` again. The setup endpoint will create one demo HOD per programme, one class teacher for every configured class, and update the class-to-teacher mapping. Classes not included in the current configuration are deactivated for demo routing.

## Demo credentials

All accounts use demo passwords for local lab use only.

### HODs — password `Hod@12345`

| Programme | Username |
|---|---|
| CSE | `hod_cse` |
| AIML | `hod_aiml` |
| AIDS | `hod_aids` |
| IT | `hod_it` |
| ECE | `hod_ece` |
| MECH | `hod_mech` |
| CIVIL | `hod_civil` |
| AEROSPACE | `hod_aerospace` |
| MBA | `hod_mba` |

### Class teachers — password `Teacher@12345`

Username format: `teacher_<programme-lowercase>_y<year>_<section-lowercase>`.

Examples:

- CSE Year 2 Section B: `teacher_cse_y2_b`
- AIML Year 4 Section H: `teacher_aiml_y4_h` (only if that class is configured)
- ECE Year 1 Section A: `teacher_ece_y1_a`
- MBA Year 1 Section A: `teacher_mba_y1_a` (only if configured)

A teacher account is active only when its class is included in `CampusConfig.java`.

### Admin

- Username: `admin`
- Password: `Admin@12345`

The previous single demo usernames `hod` and `teacher_cse3a` are deactivated by setup. Use the programme/class-specific usernames above.

## Database setup

1. Open MySQL Workbench and connect to your local MySQL Server.
2. Open `database/smartpass.sql` from this project and execute the full script. It uses `CREATE TABLE IF NOT EXISTS` and does not drop existing data. It creates the new `campus_classes` table needed for explicit class-teacher assignments.
3. Open `src/main/java/edu/smartpass/util/DB.java`.
4. Update the `PASS` default to your MySQL root password, or set `SMARTPASS_DB_PASSWORD`. Optional environment variables: `SMARTPASS_DB_URL`, `SMARTPASS_DB_USER`.
5. The default URL is `jdbc:mysql://localhost:3306/smartpass?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC`.

## Import and run in Eclipse

1. Extract the ZIP.
2. Eclipse → **File → Import → Maven → Existing Maven Projects**.
3. Select the extracted `SmartPass` folder.
4. Right-click the project → **Maven → Update Project**.
5. Use JDK 8+ and Apache Tomcat 9.
6. Right-click the project → **Run As → Run on Server**.
7. Use the port Eclipse assigned to Tomcat. The user's current local setup uses port `8081`, but your port may differ.

## Seed demo accounts and class mappings

Open the setup URL using your actual Tomcat port. Example:

`http://localhost:8081/SmartPass/setup`

Expected message: `SmartPass demo staff accounts and class assignments are ready.`

Then open:

`http://localhost:8081/SmartPass/login.jsp`

The `/setup` endpoint is intentionally for local development. Remove or disable `SetupServlet` before any public/production deployment.

## End-to-end test

1. Configure actual section counts in `CampusConfig.java`.
2. Run the SQL script to ensure the schema includes `campus_classes`.
3. Deploy/restart SmartPass and visit `/setup`.
4. Register a student in a configured class, e.g. CSE / Year 2 / Section B.
5. Student login uses their roll number or email and their chosen password.
6. Submit a permission request.
7. Sign in as `hod_cse` / `Hod@12345`; only CSE requests should appear.
8. Approve the request. SmartPass looks up the exact active class assignment in `campus_classes` and assigns the request to that teacher.
9. Sign in as `teacher_cse_y2_b` / `Teacher@12345`; the request should appear in “Requests forwarded by HOD”. Click **Verify permission**.
10. The class teacher can mark attendance only for students in their exact assigned branch/year/section.
11. Sign back in as the student to review the request status and attendance.

## Routing and validation rules

- Each HOD is tied to one programme/branch and sees requests only for that programme.
- HOD approval routes via `campus_classes`, not a loose `LIMIT 1` search over staff records.
- A request cannot be approved if no active class-teacher assignment exists; a useful flash message explains the missing branch/year/section.
- Teacher verification checks both `assigned_teacher_id` and `FORWARDED_TO_TEACHER` status.
- Teacher attendance is limited to the teacher's assigned branch/year/section.
- Student registration's section dropdown loads only active sections for the selected programme/year from `/class-options`; the server validates the selected class again before insert.
- Passwords are stored as BCrypt hashes.

## Current lab limitations

This is a teaching/demo starter, not a production campus identity system. The roster counts must be edited to match your real campus. The MBA structure is provisional until confirmed. All demo staff passwords are shared within each role and must be replaced with unique staff credentials before real use. There is no staff-management UI yet; staff accounts are provisioned by `SeedDemo.java`. Supporting-document uploads, notification inbox UI, printable permission letters, CSRF protection, and complete filtered history remain future enhancements. Confirm official attendance rules with your institution before real use.
