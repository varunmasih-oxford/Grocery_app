# Step 8 — Responsive CSS & Final Polish

This is the last step of Phase 1. We'll complete the CSS, make everything fully
responsive, polish the edit item page, and do a final review of the complete
project structure.

---

## Responsive Breakpoints

```
Desktop  ≥ 900px          Tablet  600–900px       Mobile  < 600px
─────────────────         ─────────────────       ─────────────────
┌─────────────────┐       ┌───────────────┐       ┌─────────────┐
│ 🛒  Grand: ₹350 │       │ 🛒  ₹350      │       │     🛒      │
├─────────────────┤       ├───────────────┤       │   ₹350.00   │
│ [name input   ] │       │ [name input ] │       ├─────────────┤
│ [+ Add Cat    ] │       │ [+ Add      ] │       │ [name     ] │
├─────────────────┤       ├───────────────┤       │ [+ Add Cat] │
│ Vegetables  ₹120│       │ Vegetables ₹120       ├─────────────┤
│ Item│Qty│Amt│Act│       │ Item│Qty│Amt  │       │ Vegetables  │
│ Tom │ 2 │40 │Ed │       │ Tom │ 2 │40   │       │ Subtotal:₹120│
└─────────────────┘       └───────────────┘       │ Item│Qty│Amt│
                                                   │ Tom │ 2 │40 │
                                                   └─────────────┘
```

---

## 1. Final `style.css` — Complete Version

Replace `grocery/static/grocery/style.css` with:

```css
/* ═══════════════════════════════════════
   RESET & BASE
═══════════════════════════════════════ */
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI',
               Roboto, Helvetica, Arial, sans-serif;
  background: #f0f2f5;
  color: #2c3e50;
  font-size: 15px;
  line-height: 1.6;
  min-height: 100vh;
}

a {
  color: #2980b9;
  text-decoration: none;
}
a:hover { text-decoration: underline; }

/* ═══════════════════════════════════════
   STICKY GRAND TOTAL BAR
═══════════════════════════════════════ */
.total-bar {
  background: #2d6a4f;
  color: white;
  padding: 12px 24px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  position: sticky;
  top: 0;
  z-index: 100;
  box-shadow: 0 2px 8px rgba(0,0,0,0.2);
}

.total-bar .brand {
  font-size: 1rem;
  font-weight: 700;
  letter-spacing: 0.02em;
}

.total-bar .grand-total-amount {
  font-size: 1.1rem;
  font-weight: 700;
  background: rgba(255,255,255,0.15);
  padding: 4px 14px;
  border-radius: 20px;
}

/* ═══════════════════════════════════════
   LAYOUT CONTAINER
═══════════════════════════════════════ */
.container {
  max-width: 900px;
  margin: 0 auto;
  padding: 1.5rem 1rem 4rem;
}

/* ═══════════════════════════════════════
   FLASH MESSAGES
═══════════════════════════════════════ */
.messages {
  list-style: none;
  margin-bottom: 1.25rem;
}

.messages li {
  padding: 10px 16px;
  border-radius: 6px;
  margin-bottom: 8px;
  font-size: 0.9rem;
  display: flex;
  align-items: center;
  gap: 8px;
  animation: slideIn 0.2s ease;
}

@keyframes slideIn {
  from { opacity: 0; transform: translateY(-6px); }
  to   { opacity: 1; transform: translateY(0); }
}

.messages li.success {
  background: #d4edda;
  color: #155724;
  border-left: 4px solid #28a745;
}
.messages li.success::before { content: '✓'; font-weight: 700; }

.messages li.error {
  background: #f8d7da;
  color: #721c24;
  border-left: 4px solid #dc3545;
}
.messages li.error::before { content: '✕'; font-weight: 700; }

/* ═══════════════════════════════════════
   PAGE TITLE
═══════════════════════════════════════ */
.page-title {
  font-size: 1.4rem;
  font-weight: 700;
  color: #2c3e50;
  margin: 1rem 0 1.25rem;
}

/* ═══════════════════════════════════════
   ADD CATEGORY FORM
═══════════════════════════════════════ */
.add-category-form {
  display: flex;
  gap: 8px;
  margin-bottom: 1.75rem;
  flex-wrap: wrap;
}

.add-category-form input {
  flex: 1;
  min-width: 200px;
}

/* ═══════════════════════════════════════
   INPUTS
═══════════════════════════════════════ */
input[type="text"],
input[type="number"] {
  padding: 9px 12px;
  border: 1.5px solid #ddd;
  border-radius: 7px;
  font-size: 0.9rem;
  font-family: inherit;
  outline: none;
  transition: border-color 0.2s, box-shadow 0.2s;
  background: white;
  width: 100%;
}

input[type="text"]:focus,
input[type="number"]:focus {
  border-color: #2d6a4f;
  box-shadow: 0 0 0 3px rgba(45,106,79,0.1);
}

input::placeholder { color: #bbb; }

/* ═══════════════════════════════════════
   BUTTONS
═══════════════════════════════════════ */
.btn,
button[type="submit"] {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 9px 18px;
  background: #2d6a4f;
  color: white;
  border: none;
  border-radius: 7px;
  font-size: 0.9rem;
  font-family: inherit;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s, transform 0.1s;
  white-space: nowrap;
}

.btn:hover,
button[type="submit"]:hover { background: #245a41; }

.btn:active,
button[type="submit"]:active { transform: scale(0.98); }

.btn-danger {
  background: none !important;
  color: #c0392b;
  border: none;
  cursor: pointer;
  font-size: 0.85rem;
  padding: 4px 6px;
  font-family: inherit;
  border-radius: 4px;
  transition: background 0.15s;
}
.btn-danger:hover {
  background: #fdf0f0 !important;
  text-decoration: none;
}

.btn-secondary {
  background: #f0f0f0;
  color: #555;
  border: 1.5px solid #ddd;
  border-radius: 7px;
  padding: 9px 18px;
  font-size: 0.9rem;
  font-family: inherit;
  cursor: pointer;
  text-decoration: none;
  display: inline-block;
  transition: background 0.15s;
}
.btn-secondary:hover { background: #e4e4e4; text-decoration: none; }

/* ═══════════════════════════════════════
   EMPTY STATE
═══════════════════════════════════════ */
.empty-state {
  text-align: center;
  padding: 4rem 1rem;
  color: #bbb;
}

.empty-state p {
  font-size: 1rem;
  margin-bottom: 0.5rem;
}

/* ═══════════════════════════════════════
   CATEGORY CARD
═══════════════════════════════════════ */
.category-card {
  background: white;
  border-radius: 12px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.07),
              0 4px 12px rgba(0,0,0,0.04);
  margin-bottom: 1.25rem;
  overflow: hidden;
  transition: box-shadow 0.2s;
}

.category-card:hover {
  box-shadow: 0 2px 8px rgba(0,0,0,0.1),
              0 6px 20px rgba(0,0,0,0.06);
}

.category-card summary {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 14px 20px;
  cursor: pointer;
  user-select: none;
  background: linear-gradient(to right, #f8fffe, #f0faf5);
  border-bottom: 1px solid #e8f5e9;
  list-style: none;
  gap: 12px;
}

.category-card summary::-webkit-details-marker { display: none; }
.category-card summary::marker { display: none; }

.category-card[open] summary { border-bottom-color: #d4edda; }

.category-header-left {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}

.category-name {
  font-size: 1rem;
  font-weight: 700;
  color: #1a3c2a;
}

/* collapse indicator arrow */
.category-name::before {
  content: '▶';
  font-size: 0.65rem;
  color: #2d6a4f;
  margin-right: 6px;
  display: inline-block;
  transition: transform 0.2s;
}

details[open] .category-name::before {
  transform: rotate(90deg);
}

.item-count {
  font-size: 0.75rem;
  color: #888;
  background: #f0f0f0;
  padding: 2px 9px;
  border-radius: 20px;
  font-weight: 500;
}

.category-header-right {
  display: flex;
  align-items: center;
  gap: 14px;
  flex-shrink: 0;
}

.subtotal-label { font-size: 0.88rem; color: #666; }
.subtotal-amount { font-weight: 700; color: #2d6a4f; font-size: 1rem; }

/* ═══════════════════════════════════════
   ITEMS TABLE
═══════════════════════════════════════ */
.items-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.9rem;
}

.items-table th {
  padding: 9px 16px;
  text-align: left;
  font-size: 0.72rem;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: #999;
  background: #fafafa;
  border-bottom: 1.5px solid #eee;
  font-weight: 600;
}

.items-table td {
  padding: 10px 16px;
  border-bottom: 1px solid #f5f5f5;
  color: #333;
  vertical-align: middle;
}

.items-table tbody tr:last-child td { border-bottom: none; }
.items-table tbody tr:hover td { background: #f8fffc; }

.items-table .amount-cell {
  font-weight: 500;
  color: #2c3e50;
  font-variant-numeric: tabular-nums;
}

.items-table .actions-cell {
  display: flex;
  align-items: center;
  gap: 4px;
}

.empty-row td {
  text-align: center;
  color: #ccc;
  font-style: italic;
  padding: 24px;
  font-size: 0.88rem;
}

/* ═══════════════════════════════════════
   ADD ITEM FORM (inside card)
═══════════════════════════════════════ */
.add-item-form {
  display: flex;
  gap: 8px;
  padding: 12px 16px;
  background: #f8fffe;
  border-top: 1px solid #eef8f2;
  flex-wrap: wrap;
  align-items: center;
}

.add-item-form input[name="name"]     { flex: 3; min-width: 130px; }
.add-item-form input[name="quantity"] { flex: 1; min-width: 70px; }
.add-item-form input[name="amount"]   { flex: 1; min-width: 90px; }
.add-item-form button { padding: 9px 14px; font-size: 0.85rem; flex-shrink: 0; }

/* ═══════════════════════════════════════
   EDIT ITEM PAGE
═══════════════════════════════════════ */
.edit-card {
  max-width: 440px;
  margin: 2rem auto;
  background: white;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.08);
}

.edit-card h2 {
  font-size: 1.2rem;
  font-weight: 700;
  margin-bottom: 1.5rem;
  color: #1a3c2a;
  padding-bottom: 0.75rem;
  border-bottom: 1.5px solid #eee;
}

.form-field { margin-bottom: 1.1rem; }

.form-field label {
  display: block;
  font-size: 0.85rem;
  font-weight: 600;
  color: #555;
  margin-bottom: 5px;
}

.form-field input { width: 100%; }

.field-error {
  color: #c0392b;
  font-size: 0.8rem;
  margin-top: 4px;
  display: flex;
  align-items: center;
  gap: 4px;
}
.field-error::before { content: '⚠'; }

.edit-actions {
  display: flex;
  gap: 10px;
  margin-top: 1.75rem;
  padding-top: 1rem;
  border-top: 1px solid #eee;
}

/* ═══════════════════════════════════════
   RESPONSIVE BREAKPOINTS
═══════════════════════════════════════ */

/* Tablet */
@media (max-width: 768px) {
  .total-bar { padding: 10px 16px; }
  .total-bar .brand { font-size: 0.9rem; }
  .container { padding: 1rem 0.75rem 3rem; }
}

/* Mobile */
@media (max-width: 600px) {
  .total-bar {
    flex-direction: column;
    gap: 4px;
    padding: 10px 16px;
    text-align: center;
  }

  .add-category-form { flex-direction: column; }

  .category-card summary {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }

  .category-header-right {
    width: 100%;
    justify-content: space-between;
  }

  /* hide Actions column on mobile */
  .items-table th:last-child,
  .items-table td:last-child { display: none; }

  .add-item-form { flex-direction: column; }
  .add-item-form input { width: 100%; }

  .edit-card { margin: 1rem; padding: 1.25rem; }
  .edit-actions { flex-direction: column; }
}
```

---

## 2. Final `edit_item.html`

```html
{% extends 'grocery/base.html' %}

{% block title %}Edit {{ item.name }}{% endblock %}

{% block content %}

  <div class="edit-card">
    <h2>✏️ Edit Item</h2>

    <form method="post">
      {% csrf_token %}

      {% for field in form %}
        <div class="form-field">
          <label for="{{ field.id_for_label }}">{{ field.label }}</label>
          {{ field }}
          {% if field.errors %}
            <p class="field-error">{{ field.errors.0 }}</p>
          {% endif %}
        </div>
      {% endfor %}

      <div class="edit-actions">
        <button type="submit">Save changes</button>
        <a href="{% url 'grocery:index' %}" class="btn-secondary">Cancel</a>
      </div>
    </form>
  </div>

{% endblock %}
```

---

## 3. Final `base.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{% block title %}Grocery List{% endblock %}</title>
  {% load static %}
  <link rel="stylesheet" href="{% static 'grocery/style.css' %}">
</head>
<body>

  <div class="total-bar">
    <span class="brand">🛒 Grocery List</span>
    <span class="grand-total-amount">
      Grand Total: ₹{{ grand_total|floatformat:2 }}
    </span>
  </div>

  <div class="container">

    {% if messages %}
      <ul class="messages">
        {% for message in messages %}
          <li class="{{ message.tags }}">{{ message }}</li>
        {% endfor %}
      </ul>
    {% endif %}

    {% block content %}{% endblock %}

  </div>

</body>
</html>
```

---

## 4. Final Project Structure

```
grocery_project/
├── manage.py
├── requirements.txt
├── db.sqlite3
│
├── grocery_project/
│   ├── __init__.py
│   ├── settings.py            ← INSTALLED_APPS, TEMPLATES, MESSAGE_TAGS
│   ├── urls.py                ← includes grocery.urls
│   └── wsgi.py
│
└── grocery/
    ├── __init__.py
    ├── admin.py               ← Category, Item registered
    ├── apps.py
    ├── context_processors.py  ← grand_total
    ├── forms.py               ← CategoryForm, ItemForm
    ├── models.py              ← Category, Item, subtotal()
    ├── urls.py                ← 6 URL patterns
    ├── views.py               ← 6 views
    ├── migrations/
    │   ├── __init__.py
    │   └── 0001_initial.py
    ├── static/
    │   └── grocery/
    │       └── style.css
    └── templates/
        └── grocery/
            ├── base.html
            ├── index.html
            └── edit_item.html
```

---

## 5. Final End-to-End Test Checklist

### Setup
- [ ] `python manage.py runserver` starts with no errors
- [ ] `http://127.0.0.1:8000/` loads with sticky green total bar

### Categories
- [ ] Add category "Vegetables" → success message appears
- [ ] Add category with empty name → error message appears
- [ ] Two categories visible on page
- [ ] Click category header → collapses/expands smoothly

### Items
- [ ] Add item "Tomato", qty 2, amount 40.00 to Vegetables
- [ ] Subtotal shows ₹40.00, grand total updates
- [ ] Add item "Milk", qty 1, amount 55.00 to Dairy
- [ ] Grand total now shows ₹95.00
- [ ] Edit Tomato → change amount to 50.00
- [ ] Grand total updates to ₹105.00
- [ ] Delete Milk → grand total drops to ₹50.00

### Validation
- [ ] Add item with empty name → rejected
- [ ] Add item with quantity 0 → rejected
- [ ] Add item with negative amount → rejected

### Responsive
- [ ] Resize browser to 400px wide → layout stacks correctly
- [ ] Sticky total bar visible while scrolling
- [ ] No horizontal scroll on mobile width

---

## Phase 1 Complete 🎉

You now have a fully working Django + HTML grocery app with:

| Feature | Done |
|---|---|
| Category & Item models with ForeignKey | ✓ |
| 6 views handling all CRUD operations | ✓ |
| Django forms with server-side validation | ✓ |
| Template inheritance with `base.html` | ✓ |
| Context processor for `grand_total` | ✓ |
| Flash messages for every user action | ✓ |
| Responsive CSS across all screen sizes | ✓ |
| Static file serving | ✓ |

---

## What's Next — Phase 2

| Step | Topic |
|---|---|
| Step 9  | Install & configure Django REST Framework |
| Step 10 | Serializers for Category and Item |
| Step 11 | ViewSets & router URLs |
| Step 12 | CORS & API testing |

---

*Next: Step 9 — Install & Configure Django REST Framework*
