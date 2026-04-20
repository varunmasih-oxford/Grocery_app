```md
## Step 1 — Django project setup & folder structure

Let's get your project created and understand what every file does before writing a single line of app code.

---

### What we're building



grocery_project/          ← Django project root
├── manage.py
├── requirements.txt
├── grocery_project/      ← project config package
│   ├── **init**.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── grocery/              ← your app
├── **init**.py
├── admin.py
├── apps.py
├── models.py
├── views.py
├── urls.py           ← you'll create this manually
├── forms.py          ← you'll create this manually
├── migrations/
└── templates/
└── grocery/
```
````

---

### 1. Create a virtual environment

```bash
mkdir grocery_project
cd grocery_project

python -m venv venv

# activate it
source venv/bin/activate        # Mac / Linux
venv\Scripts\activate           # Windows
````

A virtualenv keeps this project's packages isolated from your system Python. You'll see `(venv)` in your terminal when it's active.

---

### 2. Install Django & create the project

```bash
pip install django

django-admin startproject grocery_project .
```

The `.` at the end is important — it creates the project in the current folder instead of making an extra nested folder.

---

### 3. Create the app

```bash
python manage.py startapp grocery
```

Django separates "project" (config) from "app" (feature). Your project can have many apps. Here we have one: `grocery`.

---

### 4. Register the app in settings

Open `grocery_project/settings.py` and find `INSTALLED_APPS`. Add your app:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'grocery',              # ← add this
]
```

Django won't recognize your models or templates unless the app is registered here.

---

### 5. Create the templates folder

Django looks for templates inside each app. Create this structure manually:

```bash
mkdir -p grocery/templates/grocery
```

The extra `grocery/` subfolder inside templates is a Django convention — it namespaces your templates so they don't clash with other apps.

---

### 6. Tell Django where templates live

In `settings.py`, find `TEMPLATES` and confirm `APP_DIRS` is `True`:

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,   # ← this makes Django scan app/templates/ folders
        ...
    },
]
```

---

### 7. Create requirements.txt

```bash
pip freeze > requirements.txt
```

This captures exactly what's installed. Anyone cloning your project runs:

```bash
pip install -r requirements.txt
```

---

### 8. Verify everything works

```bash
python manage.py runserver
```

Open:

```
http://127.0.0.1:8000
```

You should see the Django welcome rocket page. If you do, your setup is clean.

---

### Checkpoint

Before moving to Step 2, confirm these three things:

1. `python manage.py runserver` shows no errors
2. You see the rocket page at `http://127.0.0.1:8000`
3. Your folder structure matches the tree at the top

---

Once you're ready, move to the next step.

👉 Say **"step 2"** to continue.

```
```
