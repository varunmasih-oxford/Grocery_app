# Grocery App — Full Learning Roadmap Index

Complete index of all 19 steps across 3 phases, with every topic and file covered in each step.

---

## Phase 1 — Django + HTML Templates

### Step 1 — Project Setup & Folder Structure
**Files:** `settings.py` · `manage.py`

- Create virtual environment and install Django
- `django-admin startproject` and `startapp` commands
- Register app in `INSTALLED_APPS`
- `APP_DIRS` and templates folder convention
- `requirements.txt` and runserver verification
- What every generated file does (`manage.py`, `wsgi.py`, etc.)

---

### Step 2 — Models: Category & Item
**Files:** `models.py` · `admin.py` · `migrations/`

- `CharField`, `DateTimeField`, `PositiveIntegerField`, `DecimalField`
- `ForeignKey` with `on_delete=CASCADE` and `related_name`
- `subtotal()` method using `aggregate` + `Sum`
- `__str__` and `Meta` ordering
- `makemigrations` and `migrate` commands
- Register models in admin, create superuser, test in `/admin`
- What SQL Django generated behind the scenes

---

### Step 3 — URLs & Views
**Files:** `urls.py` · `views.py` · `templates/grocery/index.html`

- Request flow: browser → project urls → app urls → view → response
- Create `grocery/urls.py` with `app_name` namespace
- `path()` with `<int:pk>` URL capture
- `include()` in project `urls.py`
- 6 views: `index`, `add_category`, `delete_category`, `add_item`, `edit_item`, `delete_item`
- `get_object_or_404`, `redirect`, `request.method` POST guard
- Post/Redirect/Get pattern explained
- Placeholder `index.html` to test routing

---

### Step 4 — Forms & Validation
**Files:** `forms.py` · `views.py` · `edit_item.html`

- `ModelForm` vs reading `request.POST` manually
- `CategoryForm` and `ItemForm` with widgets and attrs
- `class Meta` — model and fields
- Pattern 1: create new object with `form.save()`
- Pattern 2: `commit=False` to attach category before saving
- Pattern 3: `instance=` to edit an existing object
- `form.is_valid()`, `form.cleaned_data`, `form.errors`
- Shell testing of valid and invalid form inputs

---

### Step 5 — Templates: Base Layout & Index
**Files:** `base.html` · `index.html` · `edit_item.html`

- Template inheritance — `extends`, `block`, `endblock`
- `base.html` with sticky total bar and flash messages
- `{% load static %}` and `APP_DIRS` template discovery
- `index.html` with category loop and empty state
- HTML `<details>`/`<summary>` for zero-JS collapsible cards
- `{% url %}`, `{% for %}`, `{% if %}`, `{% empty %}`, `{% csrf_token %}`
- Looping over form fields for fine-grained rendering

---

### Step 6 — Category Cards, Item Table & Subtotal
**Files:** `models.py` · `views.py` · `style.css` · `index.html`

- Fix `subtotal()` — handle `None` from `aggregate`
- Add `item_count()` method to `Category`
- N+1 query problem and fix with `prefetch_related('items')`
- Create `static/grocery/` folder and `style.css`
- Full CSS: total bar, category cards, items table, add-item form
- `STATIC_URL` in `settings.py` and `static()` in `urls.py`
- `floatformat:2` filter for currency display
- Mobile responsive with `@media (max-width: 600px)`

---

### Step 7 — Grand Total & Flash Messages
**Files:** `context_processors.py` · `settings.py` · `views.py`

- Why `grand_total` breaks on non-index pages
- Create `context_processors.py` with `grand_total` function
- Register context processor in `TEMPLATES OPTIONS`
- Remove manual `grand_total` from all views
- `MESSAGE_TAGS` mapping to CSS class names
- Full message coverage table for every action
- How flash messages work internally (session lifecycle)
- Verify `MessageMiddleware` and `django.contrib.messages` in settings

---

### Step 8 — Responsive CSS & Final Polish
**Files:** `style.css` · `base.html` · `edit_item.html`

- Three breakpoints: desktop ≥900px, tablet 600–900px, mobile <600px
- Complete final `style.css` with animations, transitions, focus states
- Collapse indicator arrow using CSS `::before` and `details[open]`
- `edit-card`, `form-field`, `field-error`, `edit-actions` CSS classes
- Final `base.html` and `edit_item.html` using all CSS classes
- Complete project folder structure reference
- Full end-to-end test checklist (setup, categories, items, validation, responsive)
- Phase 1 completion summary

---

## Phase 2 — Django REST Framework

### Step 9 — Install & Configure DRF
**Files:** `settings.py` · `requirements.txt`

- `pip install djangorestframework`
- Add to `INSTALLED_APPS`, explore the browsable API
- Understanding what DRF adds on top of Django

---

### Step 10 — Serializers
**Files:** `serializers.py`

- `ModelSerializer` for Category and Item
- Nested serializers — items list inside CategorySerializer
- Computed fields — `subtotal` as a `SerializerMethodField`

---

### Step 11 — ViewSets & Router URLs
**Files:** `views.py` · `urls.py`

- `ModelViewSet` for full CRUD in one class
- `DefaultRouter` auto-generates all URL patterns
- Mount API endpoints under `/api/`
- Keep existing Django template views intact — API is additive

---

### Step 12 — CORS & API Testing
**Files:** `settings.py` · Postman / curl

- `pip install django-cors-headers`
- Allow requests from `http://localhost:5173` (Vite dev server)
- Test all endpoints with Postman or curl
- Understand what each HTTP method does (GET, POST, PUT, PATCH, DELETE)

---

## Phase 3 — React Frontend

### Step 13 — React Intro & Vite Setup
**Files:** `App.jsx` · `vite.config.js`

- What is React and why use it
- JSX — HTML-like syntax inside JavaScript
- Components — reusable UI building blocks
- Bootstrap project with `npm create vite@latest`

---

### Step 14 — useState & useEffect
**Files:** `App.jsx`

- `useState` — local component state
- `useEffect` — run code when component mounts
- Fetch categories from the Django API on page load
- Handle loading and error states

---

### Step 15 — CategoryCard Component
**Files:** `CategoryCard.jsx`

- Props — passing data from parent to child
- Rendering a list with `.map()`
- Collapsible UI in React (useState toggle)
- Displaying subtotal derived from item amounts

---

### Step 16 — Create & Delete (POST / DELETE)
**Files:** `App.jsx` · `AddItemForm.jsx`

- Controlled inputs in React with `useState`
- `fetch` POST to create a category or item
- `fetch` DELETE to remove a record
- Update local state after response — no full re-fetch

---

### Step 17 — Inline Edit (PUT / PATCH)
**Files:** `ItemRow.jsx`

- Edit-in-place UI pattern
- Toggle between display and edit mode with `useState`
- `fetch` PATCH to update a single field
- Save on blur or on button click

---

### Step 18 — Grand Total & Live State
**Files:** `App.jsx`

- Derived state — calculate grand total from existing state
- Lifting state up — share state between sibling components
- Real-time recalculation without extra API calls
- Sticky grand total header in React

---

### Step 19 — Responsive CSS & Final Polish
**Files:** `App.css` · `components/`

- Plain CSS or CSS Modules in a React project
- Mobile-first CSS Grid layout
- Final styling pass — match Phase 1 visual design
- Phase 2 + 3 completion and next steps

---

## MD Files Saved

| File | Step |
|---|---|
| `step2_django_models.md` | Step 2 — Models |
| `step3_urls_and_views.md` | Step 3 — URLs & Views |
| `step4_forms_and_validation.md` | Step 4 — Forms |
| `step5_templates_base_and_index.md` | Step 5 — Templates |
| `step6_category_cards_and_subtotal.md` | Step 6 — Cards & Subtotal |
| `step7_grand_total_and_messages.md` | Step 7 — Grand Total |
| `step8_responsive_css_final_polish.md` | Step 8 — CSS & Polish |
| `all_steps_index.md` | This file — Full Roadmap |

---

*Phase 1 complete · Phase 2 starts at Step 9*
