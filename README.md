<div align="center">

# Worklyn

**PPE detection and attendance recording for workplace safety check-ins**

Worklyn is a Django research prototype for workplace safety check-ins. An administrator records each employee's attendance with a photo. A YOLOv8 model detects hardhats, masks, and safety vests in the photo, and the result is saved with the attendance record. A dashboard summarizes attendance by week, month, and year. Body temperature is simulated in the current version, not measured (see [Limitations](#limitations)).

![Django](https://img.shields.io/badge/Django-5.2-092E20?style=for-the-badge&logo=django&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.10--3.12-3776AB?style=for-the-badge&logo=python&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-2.2-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white)
![Ultralytics YOLO](https://img.shields.io/badge/Ultralytics-YOLOv8-111F68?style=for-the-badge&logo=yolo&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-4.9-5C3EE8?style=for-the-badge&logo=opencv&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-CDN-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white)
![Flowbite](https://img.shields.io/badge/Flowbite-3.1-1C64F2?style=for-the-badge&logo=flowbite&logoColor=white)
![Chart.js](https://img.shields.io/badge/Chart.js-FF6384?style=for-the-badge&logo=chartdotjs&logoColor=white)

</div>

---

## Table of Contents

- [Features](#features)
- [How It Works](#how-it-works)
- [Limitations](#limitations)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Data Model](#data-model)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Running the application](#running-the-application)
- [Configuration](#configuration)
- [Usage](#usage)
- [Routes](#routes)
- [Troubleshooting](#troubleshooting)
- [Production Notes](#production-notes)
- [Contributing](#contributing)
- [License](#license)

## Features

- **Employee management:** create, edit, and delete employees with a name, a unique employee ID number, and a job title.
- **Attendance recording:** record one attendance entry per employee per day with the status `Hadir` (present), `Izin` (permitted leave), `Sakit` (sick), or `Alfa` (absent without notice). Attach a photo by upload or by browser camera capture.
- **PPE detection:** for `Hadir` entries with a photo, a YOLOv8 model detects `Hardhat`, `Mask`, and `Safety Vest` and stores the result as Helm, Masker, and Rompi.
- **Face detection:** an OpenCV DNN face detector checks that a face is visible before a temperature value is stored.
- **Dashboard:** total employees, present and absent counts for today, and a Chart.js chart of attendance by status over the last 7 days, the current month, or the current year.
- **Authentication:** all pages require login. Users sign in with a username or an email address.

## How It Works

```mermaid
flowchart TD
    A["Admin submits the attendance form<br/>status + photo"] --> B{"Status is Hadir<br/>and a photo is attached?"}
    B -- No --> C["Save the record<br/>no temperature, no PPE data"]
    B -- Yes --> D["Face detection<br/>OpenCV DNN, confidence above 0.7<br/>stores a simulated temperature"]
    D --> E["PPE detection<br/>YOLO best.pt<br/>keeps Hardhat, Mask, Safety Vest"]
    E --> F["Save the record and its PPE detections"]
```

Inference runs synchronously inside the request that saves the attendance record.

## Limitations

Worklyn is a prototype. Know these points before using it beyond a demonstration:

- **Body temperature is simulated.** When a face is detected, the app stores a random value around 36.2 °C (range 35.7 to 37.5). No sensor or thermal camera is involved. Do not use these values for health screening.
- **Models load on every request.** The YOLO and face detection models are read from disk each time a `Hadir` entry with a photo is saved, so each submission is slow.
- **No automated tests.** `core/tests.py` is empty.
- **Attendance entries can be edited but not deleted** in the interface. A delete view exists but has no URL or template.
- **The Django admin has no Worklyn models.** `core/admin.py` registers nothing, so `/admin/` only manages users and groups.
- **No roles.** Every authenticated user has the same access to all pages.

## Tech Stack

| Layer | Technology | Purpose |
| :--- | :--- | :--- |
| Framework | Django 5.2 | Views, ORM, authentication, admin |
| Database | MySQL through `mysqlclient` | Relational storage |
| Configuration | `django-environ` | `.env` and `DATABASE_URL` parsing |
| PPE detection | Ultralytics YOLOv8 (8.1.27), PyTorch 2.2.1, torchvision 0.17.1 | Object detection on attendance photos |
| Face detection | OpenCV DNN (headless 4.9) with a Caffe SSD model | Face presence check |
| Frontend | Django templates, Tailwind CSS (Play CDN), Flowbite 3.1.2 | Interface and components |
| Charts | Chart.js (CDN) | Dashboard attendance chart |

## Project Structure

```text
django-worklyn/
├── config/                       # Settings, root URL configuration, WSGI and ASGI
├── core/                         # Application code
│   ├── models.py                 # Employee, PPE, Attendance, PPEDetection
│   ├── views.py                  # Auth, dashboard, CRUD views, ML inference
│   ├── forms.py
│   ├── backends.py               # Sign in with username or email
│   ├── migrations/
│   └── templates/core/           # Dashboard, employee, and attendance pages
├── templates/                    # Base layout and login page
├── ml_models/best.pt             # YOLO weights
├── deploy.prototxt.txt           # Face detector definition
├── res10_300x300_ssd_iter_140000.caffemodel   # Face detector weights
├── requirements.txt
└── .env.example
```

## Data Model

```mermaid
erDiagram
    EMPLOYEE ||--o{ ATTENDANCE : has
    ATTENDANCE ||--o{ PPE_DETECTION : has
    PPE ||--o{ PPE_DETECTION : "detected as"

    EMPLOYEE {
        string full_name
        string employee_id_number UK
        string role
    }
    ATTENDANCE {
        datetime check_in_time
        decimal body_temperature
        string status
        image image_attendance
    }
    PPE {
        string name UK
    }
```

`PPE` rows are created on demand when the first detection of each item is saved. Photos are stored under `media/attendance_image/`.

## Getting Started

### Prerequisites

| Tool | Version |
| :--- | :--- |
| Python | 3.10 to 3.12 (Django 5.2 needs 3.10 or later, PyTorch 2.2.1 publishes wheels up to 3.12) |
| pip and venv | Bundled with Python |
| MySQL server | Running locally or reachable over the network |
| Build libraries for `mysqlclient` | See [Troubleshooting](#troubleshooting) |

An internet connection is required in the browser, because Tailwind CSS, Flowbite, and Chart.js load from CDNs. PyTorch is a large download during installation.

### Installation

```bash
git clone https://github.com/wildanzm/django-worklyn.git
cd django-worklyn

python3 -m venv .venv

# macOS / Linux
source .venv/bin/activate

# Windows (PowerShell)
.venv\Scripts\Activate.ps1

pip install -r requirements.txt
```

Create the database and the environment file:

```sql
CREATE DATABASE worklyn_db CHARACTER SET utf8mb4;
```

```bash
cp .env.example .env
```

Edit `.env` (see [Configuration](#configuration)), then apply the migrations and create an administrator:

```bash
python manage.py migrate
python manage.py createsuperuser
```

### Running the application

```bash
python manage.py runserver
```

Open <http://127.0.0.1:8000/> and sign in with the account you created.

## Configuration

Settings are read from `.env` in the project root.

| Variable | Required | Description |
| :--- | :--- | :--- |
| `SECRET_KEY` | Yes | Django secret key. Use a long random value |
| `DEBUG` | No | `True` or `False`. Defaults to `False` |
| `DATABASE_URL` | Yes | Database connection URL |

```dotenv
SECRET_KEY='replace_with_a_strong_and_unique_secret_key'
DEBUG=True
DATABASE_URL='mysql://worklyn_user:YourSecurePassword@127.0.0.1:3306/worklyn_db'
```

For a quick local trial, `django-environ` also accepts `DATABASE_URL='sqlite:///db.sqlite3'`. `mysqlclient` must still install because it is listed in `requirements.txt`.

Other values live in `config/settings.py`: `ALLOWED_HOSTS` (empty), `TIME_ZONE` (`Asia/Jakarta`), and `LANGUAGE_CODE` (`id-ID`). The interface text is in Indonesian.

## Usage

1. Sign in at `/`. You land on the dashboard.
2. Open **Karyawan** and add your employees.
3. Open **Absensi**, pick a date, and choose an employee to record.
4. Select a status. For `Hadir`, upload a photo or open the camera and capture one, then save.
5. Return to the dashboard and switch the chart between weekly, monthly, and yearly views.

## Routes

| Path | Description |
| :--- | :--- |
| `/` | Login |
| `/logout/` | Log out |
| `/dashboard` | Dashboard (`?filter=weekly`, `monthly`, or `yearly`) |
| `/karyawan/` | Employee list |
| `/karyawan/add/` | Add an employee |
| `/karyawan/<id>/edit/` | Edit an employee |
| `/karyawan/<id>/delete/` | Delete an employee |
| `/absensi/` | Attendance list (`?filter_date=YYYY-MM-DD`) |
| `/absensi/record/<employee_id>/` | Record attendance |
| `/absensi/<id>/edit/` | Edit an attendance entry |
| `/admin/` | Django admin (users and groups) |

## Troubleshooting

| Symptom | Likely cause | Fix |
| :--- | :--- | :--- |
| `pip install` fails while building `mysqlclient` | System libraries are missing | Debian or Ubuntu: `sudo apt install build-essential pkg-config python3-dev default-libmysqlclient-dev`. macOS: `brew install mysql pkg-config` |
| `ImproperlyConfigured: Set the SECRET_KEY` | `.env` is missing or incomplete | Copy `.env.example` to `.env` and fill in the values |
| `Unknown database` or `Access denied` | The database does not exist or the credentials are wrong | Create the database and check `DATABASE_URL` |
| "Gagal memproses gambar" after saving attendance | A model file is missing or unreadable | Confirm `ml_models/best.pt`, `deploy.prototxt.txt`, and `res10_300x300_ssd_iter_140000.caffemodel` exist in the paths shown above |
| Pages render without styling | The browser cannot reach the CDNs | Check the internet connection |
| Uploaded photos return 404 | Media files are served only when `DEBUG=True` | Serve `media/` with your web server in production |
| Warning `staticfiles.W004` about `static/` | `STATICFILES_DIRS` points to a folder that does not exist | Create an empty `static/` folder or remove the setting |

## Production Notes

- Set `DEBUG=False`, a unique `SECRET_KEY`, and `ALLOWED_HOSTS` in `config/settings.py`.
- Serve the app with Gunicorn or uWSGI behind a reverse proxy, and serve `media/` from the proxy.
- Replace the Tailwind Play CDN with a compiled stylesheet. The Play CDN is intended for development.
- Load the YOLO and face detection models once at startup instead of per request.
- Remove the simulated temperature or replace it with a real measurement before any use on real employees.

## Contributing

1. Fork the repository and create a branch from `master`.
2. Make your change and add tests to `core/tests.py`.
3. Run `python manage.py test` and `python manage.py check`.
4. Commit using [Conventional Commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `docs:`), as in the existing history.
5. Open a pull request against `master`.

## License

Released under the [MIT License](LICENSE).

---

<div align="center">

Maintained by [@wildanzm](https://github.com/wildanzm)

</div>
