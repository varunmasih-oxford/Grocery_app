# Step 5 — Templates: Base Layout & Index Page

Templates are HTML files with Django template tags mixed in. We'll build a `base.html`
that all other pages inherit from, then the main `index.html`.

---

## How Template Inheritance Works

```
          base.html
    ┌──────────────────────┐
    │ <html> <head> <nav>  │
    │ {% block content %}  │
    │ {% endblock %}       │
    │ <footer>             │
    └──────────────────────┘
           ↙       ↓       ↘
    index.html  edit_item.html  future pages
    extends     extends         extends
    base.html   base.html       base.html
```

`base.html` defines the shell — `<html>`, `<head>`, navbar, footer, CSS.
Child templates **extend** it and only fill in the `{% block content %}` section.
Every page automatically gets the same layout.

---

## 1. Create `base.html`

Create `grocery/templates/grocery/base.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{% block title %}Grocery List{% endblock %}</title>
  {% load static %}
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { font-family: sans-serif; background: #f5f5f5; color: #333; }
    .container { max-width: 900px; margin: 0 auto; padding: 1rem; }

    /* flash messages */
    .messages { list-style: none; margin-bottom: 1rem; }
    .messages li { padding: 10px 14px; border-radius: 6px; margin-bottom: 6px; }
    .messages li.success { background: #d4edda; color: #155724; }
    .messages li.error   { background: #f8d7da; color: #721c24; }
  </style>
</head>
<body>

  <!-- sticky grand total bar -->
  <div style="background:#2d6a4f; color:white; padding:10px 20px;
              text-align:right; font-weight:bold;">
    Grand Total: ₹{{ grand_total }}
  </div>

  <div class="container">

    <!-- flash messages -->
    {% if messages %}
      <ul class="messages">
        {% for message in messages %}
          <li class="{{ message.tags }}">{{ message }}</li>
        {% endfor %}
      </ul>
    {% endif %}

    <!-- each child template fills this block -->
    {% block content %}{% endblock %}

  </div>

</body>
</html>
```

### Key Tags in `base.html`

| Tag | Explanation |
|---|---|
| `{% load static %}` | Enables the `{% static %}` tag to reference CSS/JS files. Used fully in Step 8. |
| `{% block title %}` | Child templates can override the page title. Default is "Grocery List". |
| `{% if messages %}` | Only renders the messages list if there are any queued flash messages. |
| `{{ message.tags }}` | Outputs `"success"` or `"error"` as a CSS class on the `<li>`. |
| `{% block content %}{% endblock %}` | Where child templates inject their content. |

---

## 2. Create `index.html`

Replace the placeholder `grocery/templates/grocery/index.html` with:

```html
{% extends 'grocery/base.html' %}

{% block title %}Grocery List{% endblock %}

{% block content %}

  <h1 style="margin: 1.5rem 0 1rem;">My Grocery List</h1>

  <!-- Add Category Form -->
  <form method="post" action="{% url 'grocery:add_category' %}"
        style="display:flex; gap:8px; margin-bottom:2rem;">
    {% csrf_token %}
    {{ category_form.name }}
    <button type="submit">+ Add Category</button>
  </form>

  <!-- Empty state -->
  {% if not categories %}
    <p style="color:#888;">No categories yet. Add one above!</p>
  {% endif %}

  <!-- Category list -->
  {% for category in categories %}
    <details open style="background:white; border-radius:8px;
                         padding:1rem; margin-bottom:1rem;
                         box-shadow:0 1px 3px rgba(0,0,0,0.1);">

      <!-- Category header -->
      <summary style="cursor:pointer; display:flex;
                      justify-content:space-between; align-items:center;">
        <span style="font-size:1.1rem; font-weight:600;">
          {{ category.name }}
          <small style="color:#888; font-weight:400;">
            ({{ category.items.count }} items)
          </small>
        </span>
        <span>
          Subtotal: <strong>₹{{ category.subtotal }}</strong>
          &nbsp;
          <!-- Delete category -->
          <form method="post"
                action="{% url 'grocery:delete_category' category.pk %}"
                style="display:inline;"
                onsubmit="return confirm('Delete {{ category.name }} and all its items?')">
            {% csrf_token %}
            <button type="submit"
                    style="background:none; border:none;
                           color:#c0392b; cursor:pointer;">
              ✕
            </button>
          </form>
        </span>
      </summary>

      <!-- Items table -->
      <table style="width:100%; border-collapse:collapse; margin-top:1rem;">
        <thead>
          <tr style="border-bottom:2px solid #eee; text-align:left;">
            <th style="padding:8px;">Item</th>
            <th style="padding:8px;">Qty</th>
            <th style="padding:8px;">Amount (₹)</th>
            <th style="padding:8px;">Actions</th>
          </tr>
        </thead>
        <tbody>
          {% for item in category.items.all %}
            <tr style="border-bottom:1px solid #f0f0f0;">
              <td style="padding:8px;">{{ item.name }}</td>
              <td style="padding:8px;">{{ item.quantity }}</td>
              <td style="padding:8px;">{{ item.amount }}</td>
              <td style="padding:8px;">
                <a href="{% url 'grocery:edit_item' item.pk %}">Edit</a>
                &nbsp;
                <form method="post"
                      action="{% url 'grocery:delete_item' item.pk %}"
                      style="display:inline;">
                  {% csrf_token %}
                  <button type="submit"
                          style="background:none; border:none;
                                 color:#c0392b; cursor:pointer;"
                          onclick="return confirm('Delete {{ item.name }}?')">
                    Delete
                  </button>
                </form>
              </td>
            </tr>
          {% empty %}
            <tr>
              <td colspan="4" style="padding:8px; color:#aaa;">
                No items yet.
              </td>
            </tr>
          {% endfor %}
        </tbody>
      </table>

      <!-- Add item form -->
      <form method="post"
            action="{% url 'grocery:add_item' category.pk %}"
            style="display:flex; gap:8px; margin-top:1rem; flex-wrap:wrap;">
        {% csrf_token %}
        <input type="text"   name="name"     placeholder="Item name"  required>
        <input type="number" name="quantity" placeholder="Qty" min="1" required>
        <input type="number" name="amount"   placeholder="0.00"
               step="0.01" min="0.01" required>
        <button type="submit">Add Item</button>
      </form>

    </details>
  {% endfor %}

{% endblock %}
```

---

## 3. Update `edit_item.html` to Extend Base

Replace your placeholder `edit_item.html`:

```html
{% extends 'grocery/base.html' %}

{% block title %}Edit {{ item.name }}{% endblock %}

{% block content %}

  <div style="max-width:400px; margin:2rem auto; background:white;
              padding:2rem; border-radius:8px;
              box-shadow:0 1px 3px rgba(0,0,0,0.1);">
    <h2 style="margin-bottom:1.5rem;">Edit Item</h2>

    <form method="post">
      {% csrf_token %}

      {% for field in form %}
        <div style="margin-bottom:1rem;">
          <label style="display:block; font-weight:500; margin-bottom:4px;">
            {{ field.label }}
          </label>
          {{ field }}
          {% if field.errors %}
            <p style="color:#c0392b; font-size:0.85rem; margin-top:4px;">
              {{ field.errors.0 }}
            </p>
          {% endif %}
        </div>
      {% endfor %}

      <div style="display:flex; gap:10px; margin-top:1.5rem;">
        <button type="submit">Save changes</button>
        <a href="{% url 'grocery:index' %}">Cancel</a>
      </div>
    </form>
  </div>

{% endblock %}
```

> **Note:** Looping over `{% for field in form %}` lets you render each field individually
> with its own label and error message — more control than `{{ form.as_p }}`.

---

## 4. Key Template Tags Summary

| Tag | What it does |
|---|---|
| `{% extends 'grocery/base.html' %}` | Inherit the base layout |
| `{% block content %}` | Define a replaceable section |
| `{% url 'grocery:index' %}` | Reverse a URL by name — never hardcode paths |
| `{% for item in list %}` | Loop over a queryset or list |
| `{% empty %}` | Runs if the loop had nothing to iterate |
| `{% if condition %}` | Conditional rendering |
| `{% csrf_token %}` | Required hidden security field in every POST form |
| `{{ variable }}` | Output a variable's value |
| `{{ variable\|floatformat:2 }}` | Apply a filter to format output |

---

## 5. Test It

```bash
python manage.py runserver
```

Visit `http://127.0.0.1:8000/` and confirm:

- [ ] Grand total bar appears at the top
- [ ] Flash messages show after adding/deleting
- [ ] `<details>` element collapses/expands categories with zero JavaScript
- [ ] Adding an item updates the subtotal on refresh
- [ ] Edit and Delete links work

---

## Checkpoint

Before moving to Step 6, confirm:

- [ ] `base.html` created with `{% block content %}`
- [ ] `index.html` extends base and shows categories + items
- [ ] `edit_item.html` extends base and pre-fills existing data
- [ ] Flash messages appear after create/delete actions
- [ ] No `TemplateDoesNotExist` errors in the terminal

---

*Next: Step 6 — Category Cards, Item Table & Subtotal*
