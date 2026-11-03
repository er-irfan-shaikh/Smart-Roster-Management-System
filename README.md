# Smart-Roster-Management-System

# DataStraw Smart Roster Management System

## 1. Overview & Live Solution
This application solves the multi-brand D2C operations scheduling, coverage validation, and mid-month conflict resolution problem for **32 employees** distributed across **20 D2C brands**, **4 processes (CS, ACC, NDR, RET)**, and **4 operational pods**.

### Tech Stack
- **Frontend & UI:** HTML5, Tailwind CSS, Vanilla JavaScript (ES6+ modular architecture)
- **Data Store:** Relational in-memory database (`pods`, `brands`, `brand_process_rules`, `employees`, `assignments`, `roster`) persisted via `localStorage`
- **Analytics & BI Integration:** 1-Click CSV Export structured for Excel PivotTables and Power BI dashboards

---

## 2. Tell Us How You Think (Section 8 Answers)

### 1. How did you model employees, brands, processes, pods, and their relationships?
I modeled the domain using a normalized relational schema:
- **Pods (`pods`):** 4 operational pods (Alpha, Beta, Gamma, Delta), each overseen by a Pod Lead.
- **Brands (`brands` & `brand_process_rules`):** 20 D2C brands split into **80% Single-Resource (16 brands)** and **20% Multi-Resource (4 brands)**. Each brand defines process-level operating calendars (`365_DAYS`, `NO_PUBLIC_HOLIDAYS`, `MON_SAT_NO_HOLIDAYS`) and daily minimum staffing (`minStaffWeekday`, `minStaffSunday`).
- **Employees & Assignments (`employees` & `employee_assignments`):** 32 employees linked via a many-to-many assignment array specifying `{ brandId, processes: ['CS','ACC','NDR','RET'] }`, preferred weekly off day, planned leaves, and comp-offs.

### 2. How does your system generate a roster and determine whether it satisfies the required constraints?
The system uses a deterministic **Two-Pass Constraint Solver**:
- **Pass 1 (Employee Calendar Pass):** Iterates through all 30 days of November 2026, locking in hard non-availability (`LEAVE`, `COMP_OFF`), client non-operating days (`HOLIDAY`, Sunday `WEEK_OFF` for Mon-Sat brands), and preferred weekly offs.
- **Pass 2 (Coverage Validation & Pod Float Allocation):** Evaluates every active `(brand, process, day)` tuple against `minStaff`. If a gap is detected, it automatically assigns an available cross-trained Pod Float specialist or Pod Lead from the same Pod as `BACKUP`, or flags an explicit conflict.

### 3. How did you handle the difference between single-resource brands and brands with multiple employees?
- **Single-Resource Brands (80%):** 1 dedicated owner covers all active processes (`CS`, `ACC`, `NDR`). When that owner takes a weekly off or planned leave on a required working day (such as on a `365_DAYS` brand), the engine routes a Pod Float specialist or Pod Lead to cover the brand for that day.
- **Multi-Resource Brands (20%):** 2–3 specialists handle distinct or overlapping processes (e.g., `CS` requiring `minStaff: 2` on weekdays and `1` on Sundays, while `NDR` runs `MON_SAT_NO_HOLIDAYS`). Weekly offs are staggered across specialists so coverage is maintained internally before pulling a Pod Floater.

### 4. What happens when an employee managing all processes for a brand is unavailable on a required working day?
The validator immediately flags a **Critical Coverage Gap** across `CS, ACC, NDR` for that brand on that date, identifies the root cause (e.g., *"Rohini Verma is on LEAVE"*), and provides a **1-Click Assign Backup** button recommending an available Pod Floater/Lead in the same Pod without regenerating the entire monthly schedule.

### 5. How does your system handle conflicting requirements or insufficient staffing?
The engine strictly separates **Mandatory Constraints** (client operating calendar, minimum process staffing, approved leaves) from **Soft Preferences** (preferred weekly off day, target monthly offs). Mandatory coverage overrides soft weekly-off preferences (logging an amber Soft Warning), and if Pod float capacity is exhausted, the unmet requirement is highlighted in red in both the Roster Matrix and Conflict Center.

### 6. Which parts of the process did you automate, and which did you leave under human control? Why?
Initial 30-day schedule generation, operating calendar compliance, real-time gap detection, and backup candidate ranking are automated. Mid-month emergency changes, client rule upgrades, and final conflict overrides remain human-in-the-loop (with 1-click resolution buttons) because operations managers hold qualitative context on client priority and employee wellbeing.

### 7. If the company doubled its team and brand count, what would you change about your solution?
I would move the persistent data layer from `localStorage` to **PostgreSQL** behind a Python FastAPI service, upgrade the two-pass heuristic solver to **Google OR-Tools (CP-SAT)** for global multi-pod optimization, and introduce Pod-level approval workflows.

### 8. If you had one additional week to work on this, what would you improve first?
I would integrate historical daily ticket/order volume per brand to dynamically adjust `minStaff` during D2C flash sales, and add automated Slack/email notifications whenever a published shift is modified.
