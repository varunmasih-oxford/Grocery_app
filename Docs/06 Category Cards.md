# Step 6 — Category Cards, Item Table & Subtotal

In Step 5 we built the templates with inline styles. In this step we'll clean everything
up — proper subtotal calculation, a polished item table, and well-structured category cards.

---

## What We're Building

```
┌─────────────────────────────────────────────────────────┐  ← category card
│  Vegetables  [3 items]          Subtotal: ₹120.00  ✕   │  ← summary row
├──────────────┬──────┬────────────┬───────────────────────┤
│ Item         │ Qty  │ Amount (₹) │ Actions               │  ← table header
├──────────────┼──────┼────────────┼───────────────────────┤
│ Tomato       │ 2    │ 40.00      │ Edit  Delete          │
│ Onion        │ 1    │ 30.00      │ Edit  Delete          │
├──────────────┴──────┴────────────┴───────────────────────┤
│ [Item name]  [Qty]  [0.00]  [+ Add]                     │  ← add item form
└─────────────────────────────────────────────────────────┘
```

---

## 1. Fix the Subtotal Method in `models.py`

```python
# grocery/models.py
from django.db import models


class Category(models.Model):
    name = models.CharField(max_length=100)
    created_at = models.DateTimeField(auto_now_add=True)

    def subtotal(self):
        result = self.items.aggregate(
            total=models.Sum('amount')
        )['total']
        return result if result is not None else 0

    def item_count(self):
        return self.items.count()

    def __str__(self):
        return self.name

    class Meta:
        ordering = ['created_at']


class Item(models.Model):
    category = models.ForeignKey(
        Category,
        on_delete=models.CASCADE,
        related_name='items'
    )
    name     = models.CharField(max_length=200)
    quantity = models.PositiveIntegerField(default=1)
    amount   = models.DecimalField(max_digits=10, decimal_places=2)

    def __str__(self):
        return self.name

    class Meta:
        ordering = ['name']
```

> **Note:** No migration needed — we only added Python methods, not new columns.

---

## 2. Fix the N+1 Query Problem in `views.py`

Right now the `index` view runs **one extra SQL query per category** to fetch its items.
With 10 categories that's 11 queries. Fix it with `prefetch_related`:

```python
# grocery/views.py
def index(request):
    # prefetch_related fetches all items in ONE extra query
    # instead of one query per category
    categories = Category.objects.prefetch_related('items').all()

    grand_total = Item.objects.aggregate(
        total=Sum('amount')
    )['total'] or 0

    category_form = CategoryForm()

    return render(request, 'grocery/index.html', {
        'categories': categories,
        'grand_total': grand_total,
        'category_form': category_form,
    })
```

**`prefetch_related('items')`** — Django fetches all categories, then fetches **all** their
items in one second query and joins them in Python. Total: 2 queries regardless of how many
categories you have.

| Without prefetch_related | With prefetch_related |
|---|---|
| 1 query for categories + 1 per category | 1 query for categories + 1 for all items |
| 11 queries for 10 categories | Always 2 queries |

---

## 3. Create the Static CSS File

```bash
mkdir -p grocery/static/grocery
```

Create `grocery/static/grocery/style.css`:

```css
/* ── Reset & base ── */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  background: #f0f2f5;
  color: #2c3e50;
  font-size: 15px;
  line-height: 1.5;
}

/* ── Grand total bar ── */
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
  box-shadow: 0 2px 8px rgba(0,0,0,0.15);
}
.total-bar h1 { font-size: 1rem; font-weight: 600; }
.total-bar .amount { font-size: 1.2rem; font-weight: 700; }

/* ── Container ── */
.container { max-width: 900px; margin: 0 auto; padding: 1.5rem 1rem; }

/* ── Flash messages ── */
.messages { list-style: none; margin-bottom: 1.25rem; }
.messages li {
  padding: 10px 16px;
  border-radius: 6px;
  margin-bottom: 8px;
  font-size: 0.9rem;
}
.messages li.success { background: #d4edda; color: #155724; border-left: 4px solid #28a745; }
.messages li.error   { background: #f8d7da; color: #721c24; border-left: 4px solid #dc3545; }

/* ── Page heading ── */
.page-title { font-size: 1.5rem; font-weight: 700; margin-bottom: 1.25rem; }

/* ── Add category form ── */
.add-category-form {
  display: flex;
  gap: 8px;
  margin-bottom: 2rem;
  flex-wrap: wrap;
}

/* ── Inputs & buttons ── */
input[type="text"],
input[type="number"] {
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 0.9rem;
  outline: none;
  transition: border-color 0.2s;
}
input:focus { border-color: #2d6a4f; }

button, .btn {
  padding: 8px 16px;
  background: #2d6a4f;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 0.9rem;
  cursor: pointer;
  transition: background 0.2s;
}
button:hover, .btn:hover { background: #245a41; }

.btn-danger {
  background: none;
  color: #c0392b;
  border: none;
  cursor: pointer;
  font-size: 0.85rem;
  padding: 4px 8px;
}
.btn-danger:hover { text-decoration: underline; background: none; }

.btn-link {
  background: none;
  color: #2980b9;
  border: none;
  cursor: pointer;
  font-size: 0.85rem;
  padding: 4px 8px;
  text-decoration: none;
}
.btn-link:hover { text-decoration: underline; }

/* ── Category card ── */
.category-card {
  background: white;
  border-radius: 10px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.08);
  margin-bottom: 1.25rem;
  overflow: hidden;
}

.category-card summary {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 14px 20px;
  cursor: pointer;
  user-select: none;
  background: #f8fffe;
  border-bottom: 1px solid #e8f5e9;
  list-style: none;
}
.category-card summary::-webkit-details-marker { display: none; }

.category-header-left { display: flex; align-items: center; gap: 10px; }
.category-name { font-size: 1rem; font-weight: 600; }
.item-count {
  font-size: 0.78rem;
  color: #888;
  background: #f0f0f0;
  padding: 2px 8px;
  border-radius: 20px;
}
.category-header-right { display: flex; align-items: center; gap: 12px; }
.subtotal-label { font-size: 0.9rem; color: #555; }
.subtotal-amount { font-weight: 700; color: #2d6a4f; }

/* ── Items table ── */
.items-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.9rem;
}
.items-table th {
  padding: 10px 16px;
  text-align: left;
  font-size: 0.78rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #888;
  background: #fafafa;
  border-bottom: 1px solid #eee;
}
.items-table td {
  padding: 10px 16px;
  border-bottom: 1px solid #f5f5f5;
  color: #333;
}
.items-table tr:last-child td { border-bottom: none; }
.items-table tr:hover td { background: #fafffe; }
.empty-row td { color: #bbb; font-style: italic; text-align: center; padding: 20px; }

/* ── Add item form (inside card) ── */
.add-item-form {
  display: flex;
  gap: 8px;
  padding: 12px 16px;
  background: #f8fffe;
  border-top: 1px solid #eee;
  flex-wrap: wrap;
}
.add-item-form input[type="text"]   { flex: 2; min-width: 120px; }
.add-item-form input[type="number"] { flex: 1; min-width: 70px; }

/* ── Empty state ── */
.empty-state {
  text-align: center;
  padding: 3rem 1rem;
  color: #aaa;
  font-size: 1rem;
}

/* ── Responsive ── */
@media (max-width: 600px) {
  .category-card summary { flex-direction: column; align-items: flex-start; gap: 8px; }
  .items-table th:last-child,
  .items-table td:last-child { display: none; }
  .add-item-form { flex-direction: column; }
}
```

---

## 4. Update `base.html` to Use the Stylesheet

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
    <h1>🛒 Grocery List</h1>
    <span class="amount">Grand Total: ₹{{ grand_total|floatformat:2 }}</span>
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

## 5. Update `index.html` to Use CSS Classes

```html
{% extends 'grocery/base.html' %}

{% block title %}Grocery List{% endblock %}

{% block content %}

  <p class="page-title">My Grocery List</p>

  <!-- Add Category Form -->
  <form method="post" action="{% url 'grocery:add_category' %}"
        class="add-category-form">
    {% csrf_token %}
    {{ category_form.name }}
    <button type="submit">+ Add Category</button>
  </form>

  {% if not categories %}
    <div class="empty-state">
      <p>No categories yet. Add one above to get started!</p>
    </div>
  {% endif %}

  {% for category in categories %}
    <details class="category-card" open>

      <summary>
        <div class="category-header-left">
          <span class="category-name">{{ category.name }}</span>
          <span class="item-count">{{ category.item_count }} items</span>
        </div>
        <div class="category-header-right">
          <span class="subtotal-label">
            Subtotal: <span class="subtotal-amount">
              ₹{{ category.subtotal|floatformat:2 }}
            </span>
          </span>
          <form method="post"
                action="{% url 'grocery:delete_category' category.pk %}"
                style="display:inline;"
                onsubmit="return confirm('Delete {{ category.name }} and all its items?')">
            {% csrf_token %}
            <button type="submit" class="btn-danger">✕ Delete</button>
          </form>
        </div>
      </summary>

      <!-- Items table -->
      <table class="items-table">
        <thead>
          <tr>
            <th>Item</th>
            <th>Qty</th>
            <th>Amount (₹)</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          {% for item in category.items.all %}
            <tr>
              <td>{{ item.name }}</td>
              <td>{{ item.quantity }}</td>
              <td>{{ item.amount|floatformat:2 }}</td>
              <td>
                <a href="{% url 'grocery:edit_item' item.pk %}"
                   class="btn-link">Edit</a>
                <form method="post"
                      action="{% url 'grocery:delete_item' item.pk %}"
                      style="display:inline;">
                  {% csrf_token %}
                  <button type="submit" class="btn-danger"
                          onclick="return confirm('Delete {{ item.name }}?')">
                    Delete
                  </button>
                </form>
              </td>
            </tr>
          {% empty %}
            <tr class="empty-row">
              <td colspan="4">No items yet — add one below.</td>
            </tr>
          {% endfor %}
        </tbody>
      </table>

      <!-- Add item form -->
      <form method="post"
            action="{% url 'grocery:add_item' category.pk %}"
            class="add-item-form">
        {% csrf_token %}
        <input type="text"   name="name"     placeholder="Item name"  required>
        <input type="number" name="quantity" placeholder="Qty"
               min="1" value="1" required>
        <input type="number" name="amount"   placeholder="0.00"
               step="0.01" min="0.01" required>
        <button type="submit">+ Add</button>
      </form>

    </details>
  {% endfor %}

{% endblock %}
```

---

## 6. Serve Static Files in Development

In `settings.py`, confirm:

```python
STATIC_URL = '/static/'
```

In `grocery_project/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('grocery.urls', namespace='grocery')),
] + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

---

## The `floatformat` Filter

| Input | `\|floatformat:2` output |
|---|---|
| `120` | `120.00` |
| `99.9` | `99.90` |
| `50.123` | `50.12` |

Always use it when displaying currency values in templates.

---

## Checkpoint

Before moving to Step 7, confirm:

- [ ] Static files load — no 404 for `style.css` in browser dev tools
- [ ] Category cards collapse and expand on click
- [ ] Subtotal shows correctly per category
- [ ] `floatformat:2` formats all amounts to 2 decimal places
- [ ] Table has hover highlight on rows
- [ ] On mobile (resize browser) layout stacks correctly

---

*Next: Step 7 — Grand Total & Flash Messages*
