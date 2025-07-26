# SMS Gateway

A Flask-based SMS gateway application to send and receive SMS and USSD via multiple SIM800L modules, managed through an intuitive web interface and RESTful API.

---

## Table of Contents

1. [Features](#features)
2. [Prerequisites](#prerequisites)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Running the Application](#running-the-application)
6. [Usage](#usage)

   * [Web Interface](#web-interface)
   * [REST API](#rest-api)
7. [Database Seeding](#database-seeding)
8. [Scheduler](#scheduler)
9. [Services](#services)
10. [Directory Structure](#directory-structure)
11. [Contributing](#contributing)
12. [License](#license)

---

## Features

* **User & Role Management**: Admin, Manager, Viewer roles with CRUD operations
* **SMS Sending**: Single SMS and bulk campaigns via CSV import
* **Inbound SMS Inbox**: View, filter, and export received messages
* **USSD Sessions**: Send USSD codes and view session history
* **AT Command Console**: Enqueue and execute arbitrary AT commands on SIM modules
* **SIM Status Monitoring**: Real-time status and signal strength display
* **Dashboard & Reports**: Statistics on SMS volume, SIM health, scheduled tasks
* **REST API**: Secure endpoint for external alert systems (`/sms/api/send_single`)
* **Scheduler**: Background daemon to dispatch scheduled tasks (SMS & USSD)
* **Database Seeding**: Scripts for demo data and admin account setup

---

## Prerequisites

* Python 3.6+
* MySQL/MariaDB server
* Arduino Mega with SIM800L modules (via serial ports `/dev/ttyUSB*`)
* `pip` for Python package management

---

## Installation

1. **Clone the repository**

   ```bash
   git clone https://github.com/<username>/elmadkouryassine-sms_gateway.git
   cd elmadkouryassine-sms_gateway
   ```

2. **Install Python dependencies**

   ```bash
   python3 get-pip.py
   pip install flask flask_sqlalchemy pymysql werkzeug
   ```

3. **Create the database**

   ```sql
   CREATE DATABASE sms_gateway;
   ```

4. **Configure database URI**

   * Edit `config.py`:

     ```python
     SQLALCHEMY_DATABASE_URI = 'mysql://root:<password>@localhost/sms_gateway'
     ```

5. **Initialize tables**
   Run a Python shell:

   ```python
   from app import app
   from database.models import db

   with app.app_context():
       db.create_all()
   ```

---

## Configuration

* **Secret key**: In `app.py`, replace `app.secret_key` with a secure value.
* **SIM device mapping**: Ensure serial ports on Raspberry Pi match `SimPort.port_number - 1` to `/dev/ttyUSB*` in `run_scheduler.py` and `serial_handler.py`.

---

## Running the Application

1. **Start the Flask server**

   ```bash
   python app.py
   ```

   * Accessible at `http://<host>:5000`

2. **Start the scheduler**

   ```bash
   python run_scheduler.py
   ```

   * Runs in foreground; consider using `systemd` or `supervisor` for production

---

## Usage

### Web Interface

* **Login**: `http://<host>:5000/auth/login`
* **Dashboard**: Overview of SMS sent, active SIMs, pending tasks
* **Send SMS**: Single SMS and CSV-based campaigns
* **Inbox**: Filter by port, sender, date; export to CSV
* **USSD**: Send codes, view sessions
* **AT Commands**: Queue commands, view execution results
* **SIM Status**: Real-time cards showing signal quality and operator
* **Reports**: Detailed logs of messages and campaigns with CSV export

### REST API

* **Single SMS endpoint**:

  ```http
  POST /sms/api/send_single
  Headers:
    X-API-KEY: <user_api_key>
  JSON body:
    {
      "phone_number": "+2126xxxxxxx",
      "message":      "ALERT: disk usage > 90%",
      "port":         2       # optional, defaults to 1
    }
  ```
* **Response**: `201 Created` with JSON `{ "status": "queued", "message_id": <id> }`

---

## Database Seeding

* **Admin user**: `scripts/seed_admin.py`
* **Default roles**: `scripts/seed_roles.py`
* **Demo data**: `scripts/seed_demo.py`, `scripts/seed_inbox_demo.py`
* **Settings seed**: `scripts/seed_settings.py`

Run:

```bash
python scripts/seed_roles.py
python scripts/seed_admin.py
python scripts/seed_demo.py
python scripts/seed_inbox_demo.py
```

---

## Scheduler

* **Pending tasks** fetched by `services/scheduler.get_pending_tasks()`
* **Execution** marks tasks as sent and logs timestamps
* **USSD handling** in `run_scheduler.py` via `services.serial_handler.send_ussd`

---

## Services

* **`services/scheduler.py`**: Task polling & status update
* **`services/serial_handler.py`**: Serial communication (AT, USSD)
* **`services/sim_monitor.py`**: Update and log SIM status
* **`services/sms_parser.py`**: CSV number extraction

---

## Directory Structure

```
elmadkouryassine-sms_gateway/          # Project root
├── app.py                            # Flask application factory & routes
├── config.py                         # Configuration (DB URI)
├── get-pip.py                        # Bundled pip installer
├── run_scheduler.py                  # Background scheduler loop
├── curl.txt                          # Example cURL request
├── database/                         # SQLAlchemy models
│   ├── __init__.py
│   └── models.py                     # DB schema definitions
├── routes/                           # Flask Blueprints for features
│   ├── auth.py, auth_decorator.py
│   ├── admin.py                      # User & role management
│   ├── sms.py, sms_campaign.py       # SMS sending
│   ├── at.py                         # AT commands
│   ├── ussd.py                       # USSD sessions
│   ├── sim_status.py                 # SIM status pages
│   ├── inbox.py, routes_inbox.py     # Incoming messages
│   ├── tasks.py                      # Scheduled tasks view
│   ├── dashboard.py, reports.py      # APIs & reports
│   └── settings.py                   # Web-based settings
├── scripts/                          # DB seeding & test scripts
│   └── seed_*.py, test_db.py, test_sim_updates.py
├── services/                         # Core business logic
│   ├── scheduler.py
│   ├── serial_handler.py
│   ├── sim_monitor.py
│   └── sms_parser.py
├── static/                           # JavaScript, CSS assets
│   ├── *.js
│   └── style.css
└── templates/                        # Jinja2 templates (HTML)
    ├── base.html, dashboard.html, login.html
    ├── sms_*, ussd_*, at_command.html
    ├── admin/, reports/, settings/, partials/
    └── sim_status.html, inbox templates
```

---

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/XYZ`)
3. Commit your changes (`git commit -m "Add XYZ feature"`)
4. Push to branch (`git push origin feature/XYZ`)
5. Open a Pull Request

Please write tests for new features and follow PEP8 style.

---
