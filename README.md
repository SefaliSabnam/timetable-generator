# Intelligent Timetable Generator 

Free, zero-install prototype. Single file (`index.html`), no backend, no database, no paid service.

## Run
Open `index.html` in any browser. Live demo (free): GitHub Pages -> Settings -> Pages -> Deploy from branch `main` / root.

## Tech stack
| Layer | Choice | Why |
|---|---|---|
| Language | HTML5, CSS3, vanilla JavaScript (ES6+) | No build step, no install, runs by double-clicking the file or opening it on any free static host. |
| UI | Hand-written CSS (flexbox/grid, CSS variables for light/dark theme) | Keeps the app to one file with no external CSS framework or network dependency. |
| Scheduling engine | Custom backtracking constraint-satisfaction algorithm, written in JavaScript | Timetabling is a classic CSP; a small dependency-free backtracker is enough at this scale and keeps everything client-side. |
| Data storage | None (in-memory only for this prototype) | Keeps the assignment free and self-contained. Swappable for a real database later (see Future scope). |
| Hosting | Static file — works from `file://`, GitHub Pages, Netlify, or any web server | No backend/server code needed, so hosting is free. |
| Export | Client-side CSV generation (`Blob` + `URL.createObjectURL`) | Opens directly in Excel/Google Sheets without any server round-trip. |

No paid API, database, or framework is required to run or evaluate this project.

## Project requirements

**Functional requirements**
- Capture week structure: working days, periods/day, break position, period timing.
- Capture rooms (name, type — classroom/lab, capacity).
- Capture divisions/sections (name, student count).
- Capture faculty (name, unavailable slots).
- Capture subject requirements per division: faculty, periods/week, session block size (e.g., 2-period labs), required room type.
- Generate a timetable that respects all hard constraints (see below).
- Detect and clearly explain impossible inputs before attempting to generate.
- Detect and clearly explain why generation failed, if it fails after valid input.
- View the generated timetable by division, by faculty, and by room.
- Export the timetable (CSV) and allow printing.

**Non-functional requirements**
- Zero cost to run, host, and demo (no paid services).
- No installation — runs in any modern browser.
- Usable by a non-technical staff member (forms, not raw data files).
- Deterministic verification — every generated timetable is independently re-checked before being shown as valid.
- Responsive layout (desktop and mobile) with light/dark theme support.

**Hard constraints (must never be violated)**
- No teacher, room, or division double-booked in the same period.
- Room type must match the subject's requirement (lab vs classroom).
- Room capacity must be at least the division size.
- Teacher's declared unavailable slots are respected.
- Multi-period sessions (e.g., labs) are consecutive periods and never span the break.

**Soft constraint**
- A subject is scheduled at most once per day for a division, when the weekly load allows it.

## Project process
1. **Requirement analysis** — read the problem statement, listed entities (teachers, rooms, divisions, subjects, periods) and identified hard vs soft constraints.
2. **Data modelling** — designed a simple schema (days, periods, break, rooms, divisions, faculty, requirements) that could represent any college's structure without code changes.
3. **UI design** — built a tabbed form (Week setup / Rooms / Divisions / Faculty / Subjects) so a non-technical user never has to see or edit raw data.
4. **Constraint validation (pre-check)** — implemented a fail-fast layer that catches impossible setups (overloaded teacher, undersized room, etc.) and explains each one in plain language before the solver runs.
5. **Solver design** — implemented a backtracking algorithm that orders the hardest-to-place sessions first and retries with randomised ordering if it gets stuck, to increase the chance of finding a valid schedule.
6. **Independent verification** — added a second, separate function that re-checks the solver's output from scratch, so the "0 conflicts" message is never just the solver's own claim.
7. **Failure reporting** — when no timetable can be found, the app reports the specific session that blocked progress and why, instead of a generic error.
8. **Views and export** — added division/faculty/room views, colour-coded subjects, real clock times, CSV export, and print support.
9. **Testing** — ran a working sample (32 sessions, 0 conflicts) and a deliberately impossible sample (undersized lab, overloaded teacher) through the app to confirm both the success and failure paths work correctly (see Edge cases tested).
10. **Documentation** — wrote this README (approach, assumptions, trade-offs) and the mandatory AI usage report.

## Approach
1. **Input**: entered through editable forms (week setup, rooms, divisions, faculty with unavailable slots, subjects with periods/week, block size and room type).
2. **Pre-check (fail fast, explain clearly)**: unknown division/faculty, blocks that cannot fit, no room big enough, teacher overload vs available slots, division overload vs weekly slots, room-type capacity. Each problem is shown in plain language.
3. **Solver**: backtracking search. Sessions ordered most-constrained first (lab blocks, few slots/rooms). Randomised restarts (10 attempts, 60k steps each).
4. **Independent verifier**: re-checks the finished timetable for teacher, room and division clashes, teacher unavailability and room capacity. The UI reports "0 violations" only if this passes.
5. **If solving fails after pre-checks pass**: shows the hardest session and why its slots were rejected (teacher busy / division busy / no free room / teacher unavailable / subject already that day).
6. **Views**: by division, faculty or room. Export the timetable as CSV (opens in Excel) or print it.

## Hard constraints
Teacher, room and division cannot double-book. Room type must match (lab vs classroom). Room capacity >= division size. Teacher unavailable slots respected. Lab blocks are consecutive periods and never span the break.

## Soft rule
A subject appears at most once per day for a division (when periods/week <= number of days).

## Assumptions
- Same period grid for all days. One teacher per requirement. One break, after period `breakAfter`.
- Divisions are independent groups (no electives/shared classes).

## Trade-offs and limitations
- Backtracking is complete only within the step limit; very large inputs may report "not found" even if a solution exists.
- No manual drag-and-drop editing, no saved data between sessions (re-enter details or keep the downloaded CSV). Next steps: manual swap with live conflict check, teacher max-hours/day, soft-preference scoring, database storage so setups are saved automatically.

## Edge cases tested
- "Load working example": generates and verifies with 0 violations.
- "Load impossible example": lab too small, teacher over 28 available slots, division over 30 weekly slots -> clear messages, no crash.
- Empty setup, unknown teacher/division reference, block size not dividing periods/week evenly.
