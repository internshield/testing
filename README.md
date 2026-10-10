```
==========================================================================
                                                                          
        EXAMINATION  MANAGEMENT  PORTAL   ( E M P )                       
        Flask  .  Jinja2  .  SQLite  .  Bootstrap                         
                                                                          
        Admin  |  Examiner  |  Student   -   one portal, three roles      
                                                                          
==========================================================================
```

A small web portal to run viva / practical / project-demo examinations
from start to end: the admin sets up the exam, examiners create slots,
students book a slot, examiners give marks, admin publishes the result.

Everything runs locally on your own computer. No internet is needed
except for the Bootstrap stylesheet that the pages load from a CDN.


```
--------------------------------------------------------------------------
   [ 1 ]   CREATOR
--------------------------------------------------------------------------
```

| | |
|---|---|
| **Name**  | Zaira Rahman |
| **Email** | zairarahman24@gmail.com |
| **Project folder / zip** | `24F2008737` |


```
--------------------------------------------------------------------------
   [ 2 ]   LOGIN ID AND PASSWORD
--------------------------------------------------------------------------
```

**Admin (already created, nobody can register as admin)**

| Field    | Value        |
|----------|--------------|
| Username | `Admin`      |
| Password | `24f2008737` |

**Student**: click *"Create a student or examiner account"* on the login
page, fill the form, and log in straight away.

**Examiner**: register the same way, but the account stays *pending*.
The admin has to press **Approve** (Admin > Examiners) before the first
login works. The admin can also add examiners and students directly.

> The admin username and password are written in `config.py`
> (`ADMIN_USERNAME`, `ADMIN_PASSWORD`). They are used only the very
> first time, when the database is created.


```
--------------------------------------------------------------------------
   [ 3 ]   HOW TO RUN
--------------------------------------------------------------------------
```

**Easy way (Windows)**

1. Unzip the folder `24F2008737`.
2. Double click **`run.bat`**.
3. It installs Flask, opens `http://127.0.0.1:5000` and starts the app.

**Manual way**

```
pip install -r requirements.txt
python app.py
```

Then open `http://127.0.0.1:5000` in the browser.

`flask --app app run` works too, because the tables are created when
`app.py` is imported (not only when it is run directly).

Requirements: Python 3.9 or newer and Flask 3.

**Start again from zero:** stop the app, delete `Database/emp.db`, start
the app. A new empty database and the admin account are made again.


```
--------------------------------------------------------------------------
   [ 4 ]   TREE OF THE FILES
--------------------------------------------------------------------------
```

```
24F2008737/
|
|-- app.py                     starts the app, joins all parts, runs init_db()
|-- config.py                  admin login, secret key, exam types and statuses
|-- helpers.py                 login_required, in_window, take_seat, give_seat_back
|-- api_routes.py              small JSON api  (/api/exams, /api/slots, /api/bookings)
|-- run.bat                    one click start for Windows
|-- requirements.txt           Flask
|-- README.md                  this file
|
|-- Login and Registration/
|   |-- login.html             login page
|   |-- Registration.html      student / examiner sign up
|   `-- auth_routes.py         login, register, logout
|
|-- Admin/
|   |-- Admin_dashboard.html   all admin pages (one page per sidebar link)
|   `-- admin_routes.py        courses, exams, rubrics, people, slots, bookings
|
|-- Examiner/
|   |-- Examiner_dashboard.html   dashboard, my slots, booked students, evaluate
|   `-- examiner_routes.py        create / edit slots, give marks
|
|-- Student/
|   |-- Student_dashboard.html    dashboard, exams + booking, history, results, profile
|   `-- student_routes.py         book, cancel, profile, scorecard download
|
|-- Images/                    workflow-management.png, workflow-sketch.png (used in this README)
|
|-- Database/
|   |-- database.py            tables (SCHEMA), init_db(), query() and execute()
|   `-- emp.db                 made automatically on the first start
|
`-- Style/
    `-- Style.css              one stylesheet for every page
```

Every HTML file is complete on its own (own `<!DOCTYPE html>`, head,
top bar and sidebar). There is no shared layout file. Each role folder
keeps its page and its python file side by side.


```
--------------------------------------------------------------------------
   [ 5 ]   WHAT EACH ROLE CAN DO
--------------------------------------------------------------------------
```

**Admin**
- Dashboard with counts and simple bar charts (bookings, examiner workload, exam progress)
- Courses: add, edit, delete, search
- Examinations: create, edit, delete, set status, assign / remove examiners, publish results
- Rubrics: marking criteria for each exam (their total can never go above the exam's maximum marks)
- Examiners and Students: add directly, approve pending examiners, activate / deactivate
- All Slots: see every slot, change the examiner of a slot
- Bookings: see every booking, move a student to another open slot of the same exam

**Examiner**
- Dashboard: assigned exams, upcoming slots, pending evaluations
- My Slots: create, edit, remove slots (date, time, capacity, venue)
- Booked Students: see who booked, evaluate or re-evaluate with the rubric marks and remarks

**Student**
- Dashboard: open exams, free slots, upcoming exams, latest results
- Examinations and Booking: search by name / course / type and book a slot
- Booking History: see every booking and cancel an active one
- Results: published marks with remarks, download the scorecard as a text file
- My Profile: update details and change password

All data tables have search where it makes sense and show
**Total records** under the table. Any number of records can be entered;
long tables scroll inside their card with the heading row staying visible.


```
--------------------------------------------------------------------------
   [ 6 ]   HOW AN EXAM TRAVELS  (STATUS FLOW)
--------------------------------------------------------------------------
```

```
Draft  ->  Slot Creation  ->  Booking Open  ->  Closed  ->  Completed
```

1. **Admin** adds a course, creates the exam (with the four dates), assigns an examiner and adds rubrics.
2. **Admin** sets the exam to *Slot Creation*. Examiners can now add slots (only while today is inside the slot creation dates).
3. **Admin** sets the exam to *Booking Open*. Students can book (only while today is inside the booking dates).
4. After the slot date, the **examiner** gives marks for each student.
5. **Admin** presses *Publish results*. Students see the marks and the exam becomes *Completed*.

Slot status: `Available` -> `Full` (no seat left) -> `Completed`.
A slot becomes *Completed* only when nobody is waiting for evaluation
**and** no seat is still free, so an empty seat is never locked.

Booking status: `Booked` -> `Completed` (after evaluation) or `Cancelled`.

**Workflow (management view)**

![Management workflow](Images/workflow-management.png)

**Workflow (hand drawn on canvas)**

![Hand drawn workflow](Images/workflow-sketch.png)


```
--------------------------------------------------------------------------
   [ 7 ]   IMPORTANT RULES BUILT IN
--------------------------------------------------------------------------
```

- A student can hold only **one live booking per exam**. Cancel first to choose another slot.
- A slot is booked one seat at a time, so two students can never take the last seat together.
- An examiner cannot make two overlapping slots on the same day, and cannot make a slot in the past.
- A slot can be edited or removed only while no student has booked it and the exam is still in *Draft* / *Slot Creation*.
- An examiner can evaluate only students who booked **their own** slots.
- Marks must be between 0 and the maximum of each rubric criterion. Once results are published the marks are locked.
- A course that already has exams, or an exam that already has slots, cannot be deleted (set it Inactive / Closed instead).
- Pages are protected by role. A student opening an admin link is sent back to his own dashboard.
- Passwords are stored as hashes, never as plain text.


```
--------------------------------------------------------------------------
   [ 8 ]   DATABASE  (SQLite, Database/emp.db)
--------------------------------------------------------------------------
```

| Table              | Holds |
|--------------------|-------|
| `users`            | admin, examiners, students (role + status) |
| `courses`          | course code, name, description, status |
| `exams`            | exam details, marks, the four dates, status |
| `assignments`      | which examiner is assigned to which exam |
| `rubrics`          | marking criteria of an exam |
| `slots`            | date, time, capacity, seats available, venue |
| `bookings`         | which student booked which slot |
| `evaluations`      | total marks, remarks, published flag |
| `evaluation_marks` | marks given for each rubric criterion |

The file and all tables are created by `init_db()` in
`Database/database.py`. Nothing has to be created by hand.


```
--------------------------------------------------------------------------
   [ 9 ]   API ENDPOINTS  (JSON, login required)
--------------------------------------------------------------------------
```

| URL             | Returns |
|-----------------|---------|
| `/api/exams`    | exams (admin sees drafts too, others do not) |
| `/api/slots`    | slots, optional `?exam_id=1` |
| `/api/bookings` | admin: all, examiner: own slots, student: own bookings |

Without a login they answer `401`.


```
--------------------------------------------------------------------------
   [ 10 ]   THINGS TO REMEMBER
--------------------------------------------------------------------------
```

- Route names carry the role as a prefix because each role is a Flask
  blueprint: `auth.login`, `admin.show_page`, `examiner.save_slot`,
  `student.book_slot`. Use the same style when adding a new route.
- The python files sit inside the role folders, so `app.py` adds those
  folders to the import path at the top. Keep that part when editing.
- To add a new admin page: add its name to `ADMIN_PAGES` in
  `admin_routes.py`, fill the data in `show_page`, and add a
  `{% if page == '...' %}` block plus a sidebar link in
  `Admin_dashboard.html`.
- Change the `SECRET_KEY` in `config.py` before using it anywhere other
  than your own computer, and `debug=True` in `app.py` is only for local use.


```
--------------------------------------------------------------------------
   Examination Management Portal  ~  By 24F2008737
   Creator : Zaira Rahman  |  zairarahman24@gmail.com
--------------------------------------------------------------------------
```
