# Worklyn - PPE & Temperature Detection System

**Worklyn** is a web-based research prototype designed to enhance workplace safety by automatically detecting Personal Protective Equipment (PPE) and body temperature during employee check-ins. The system leverages a machine learning model for real-time image processing to ensure compliance with safety protocols.

## 🚀 Key Features

  - **Comprehensive Admin Panel:** A customized Django admin interface for managing employee data, PPE types, and attendance records.
  - **AI-Powered PPE Detection:** Utilizes a YOLOv8 model to accurately detect essential safety gear such as `Hardhats`, `Masks`, and `Safety Vests`.
  - **Automated Temperature Screening:** Employs an OpenCV DNN model for face detection as a basis for simulated, non-contact temperature checks.
  - **Modern & Responsive UI:** The front-end is built with **Flowbite** and **Tailwind CSS** for a clean, intuitive, and responsive user experience across all devices.


## 🛠️ Tech Stack

  - **Backend:** Django 5.2
  - **Database:** MySQL
  - **Machine Learning:** PyTorch, YOLOv8, OpenCV
  - **Frontend:** HTML, CSS, Tailwind CSS, Flowbite


## ⚙️ Setup and Installation

Follow these steps to get the project running locally.

### 1. Prerequisites

  - Python 3.8+
  - PIP & Venv
  - MySQL Database Server

### 2. Installation Guide

1.  **Clone the repository:**

    ```bash
    git clone https://github.com/wildanzm/django-worklyn.git
    cd django-ohs
    ```

2.  **Create and activate a virtual environment:**

    ```bash
    # Create the venv
    python -m venv venv

    # Activate on macOS/Linux
    source venv/bin/activate

    # Activate on Windows
    .\venv\Scripts\activate
    ```

3.  **Install all dependencies:**

    ```bash
    pip install -r requirements.txt
    ```

4.  **Configure Environment Variables:**

      - Create a new file named `.env` in the project's root directory.
      - Add the following configuration, replacing the placeholder values with your actual credentials.
        ```env
        SECRET_KEY='replace_with_a_strong_and_unique_secret_key'
        DEBUG=True
        DATABASE_URL='mysql://DB_USER:DB_PASSWORD@DB_HOST:DB_PORT/DB_NAME'
        ```
        **Example `DATABASE_URL`**: `mysql://worklyn_user:YourSecurePassword@127.0.0.1:3306/worklyn_db`

5.  **Database Setup:**

      - Ensure you have created an empty database in MySQL that matches the name in your `.env` file.

6.  **Run Migrations:**

    ```bash
    python manage.py migrate
    ```

7.  **Create a Superuser (Admin):**

    ```bash
    python manage.py createsuperuser
    ```

    Follow the prompts to create your admin username, email, and password.

8.  **Run the Development Server:**

    ```bash
    python manage.py runserver
    ```


### 3. Usage

  - **Login Page:** Access the application's entry point at `http://127.0.0.1:8000/`.
  - **Main Dashboard:** After logging in, you will be redirected to the main dashboard at `http://127.0.0.1:8000/dashboard`.
  - **Admin Panel:** Access the Django admin interface at `http://127.0.0.1:8000/admin/` for low-level data management.
