# ESPRIT PIDEV — Exam Management Desktop App (Qt / C++)

A Qt Widgets desktop application for managing national exams: the schools that sit them, the exams themselves, the exam centres, and the teachers who supervise and correct. Built as the C++ integration project (PIDEV) at ESPRIT.

The app connects to an Oracle database over ODBC, and adds a few extras on top of the CRUD screens: statistics charts, PDF export, an OpenStreetMap view, email notifications, a small rule-based assistant, and an Arduino link that pulls pending exams over serial.

## Features

| Module | What it does |
|---|---|
| **Établissements** (schools) | Add / edit / delete, input validation, search, sort, statistics by address, PDF export |
| **Examens** (exams) | Add / edit / delete with duplicate-name check, search, sort, attached PDF, stats (by level, session, year, average copies), formatted PDF report |
| **Centres d'examen** (exam centres) | Add / edit / delete, tree view, bar chart by region, PDF export and a "verification contract" PDF, email to the centre manager, map with markers |
| **Enseignants** (teachers) | Add / edit / delete, search, sort, role statistics, PDF export, role/assignment suggestions |
| **Assistant** | Separate chat window with keyword-based answers and teacher suggestions (`AIManager`) |
| **Arduino** | Sends the list of pending exams to an Arduino Uno over serial and marks exams as done when the board reports them |

The navigation also includes **Ressources** and **Candidats** pages; their logic is not part of this repository.

## Tech stack

- **C++17**, **Qt 6.7** (built with Qt Creator, MinGW 64-bit)
- Qt modules: Widgets, SQL, Charts, PrintSupport, Network, SerialPort, QuickWidgets, Location, Positioning
- **Oracle Database** via the `QODBC` driver
- **QML** + OpenStreetMap plugin for the map
- **Arduino Uno** (optional) over USB serial

## Project structure

```
.
├── integration.pro        # qmake project file
├── main.cpp               # entry point: DB connection, main window, chatbot window
├── mainwindow.{h,cpp,ui}  # all screens and their slots
├── connection.{h,cpp}     # ODBC database connection
├── etablissement.{h,cpp}  # school entity + SQL
├── exam.{h,cpp}           # exam entity + SQL (search, sort)
├── centre.{h,cpp}         # exam-centre entity + SQL
├── enseignant.{h,cpp}     # teacher entity + SQL
├── aimanager.{h,cpp}      # rule-based role / teacher suggestions
├── chatbotwindow.{h,cpp}  # assistant window
├── smtpclient.{h,cpp}     # minimal SMTP-over-SSL client
├── arduino.{h,cpp}        # serial link to the Arduino
├── map.qml, qml.qrc       # OpenStreetMap view
└── build/                 # Qt Creator build output (not needed to build)
```

## Getting started

### Prerequisites

- Qt 6.7+ with the **Charts**, **Serial Port**, **Positioning** and **Location** add-ons (install them from the Qt Maintenance Tool)
- Qt Creator (recommended) with the MinGW 64-bit kit
- Oracle Database (e.g. Oracle XE) and the Oracle ODBC driver
- Optional: an Arduino Uno

### 1. Set up the database

1. Create an Oracle user/schema (the code expects `projetC`).
2. Create the tables used by the app: `ETABLISSEMENT`, `EXAM`, `CENTRE_EXAMEN`, `ENSEIGNANT`, `ASSIGNMENTS`.
3. On Windows, open **ODBC Data Sources (64-bit)** and add a DSN named `ProjetC` pointing to that database.

The connection settings live in `connection.cpp`:

```cpp
db.setDatabaseName("ProjetC"); // ODBC DSN
db.setUserName("projetC");
db.setPassword("projetC");
```

### 2. Build and run

1. Open `integration.pro` in Qt Creator.
2. Select the Desktop Qt 6.7.x MinGW 64-bit kit.
3. Build and run (Ctrl+R).

On start-up the app shows whether the database connection succeeded, then opens the login screen.

### 3. Email (optional)

Centre notifications use `SmtpClient` against Gmail (`smtp.gmail.com:465`). You need a Gmail address and an [app password](https://support.google.com/accounts/answer/185833). Keep these out of the source code — read them from environment variables or a local config file that is git-ignored.

## Arduino protocol

The board talks to the app line by line over serial (Arduino Uno, auto-detected by vendor/product ID):

| Direction | Message | Meaning |
|---|---|---|
| Arduino → App | `REQUEST_EXAMS` | Ask for pending exams |
| App → Arduino | `ID,NAME,YYYY-MM-DD` | One pending exam (`etat IS NULL`) per line |
| App → Arduino | `END_TRANSMISSION` / `NO_EXAMS_FOUND` | End of list |
| Arduino → App | `REMOVE:<id>` | Mark that exam as completed (`etat = 1`) |
| App → Arduino | `SUCCESS:<id>` / `ERROR:...` | Result; the list is then re-sent |

## Team

ESPRIT — PIDEV C++ project.

| Member | Module |
|---|---|
| Hamza Najjar | Établissements |
| Ranya Ouesleti | Examens |
| Yomen Souissi | Centres d'examen |
| Nada Ajmi | Enseignants |
