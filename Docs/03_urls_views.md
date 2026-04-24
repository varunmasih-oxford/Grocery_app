# Step 3 — URLs & Views

Views receive a request and return a response. URLs tell Django which view to call for each path.

---

## How the Request Flow Works

```
Browser → project/urls.py → grocery/urls.py → views.py → HTML response
```

Django first checks the **project** urls.py, which hands off to the **app** urls.py, which calls the right **view**, which returns a response.

---

## The URLs We Need

| URL | View | Purpose |
|---|---|---|
| `/` | `index` | List all categories + items |
| `/category/add/` | `add_category` | Create a new category |
| `/category/<id>/delete/` | `delete_category` | Delete a category |
| `/category/<id>/item/add/` | `add_item` | Add item to a category |
| `/item/<id>/edit/` | `edit_item` | Edit an item |
| `/item/<id>/delete/` | `delete_item` | Delete an item |

---

## 1. Create `grocery/urls.py`

This file doesn't exist yet — create it manually:

```python
from django.urls import path
from . import views

app_name = 'grocery'

urlpatterns = [
    path('', views.index, name='index'),
    path('category/add/', views.add_category, name='add_category'),
    path('category/<int:pk>/delete/', views.delete_category, name='delete_category'),
    path('category/<int:pk>/item/add/', views.add_item, name='add_item'),
    path('item/<int:pk>/edit/', views.edit_item, name='edit_item'),
    path('item/<int:pk>/delete/', views.delete_item, name='delete_item'),
]
```

**`<int:pk>`** — captures a number from the URL and passes it to the view as `pk`.
So `/item/5/delete/` gives the view `pk=5`.

**`app_name = 'grocery'`** — namespaces your URLs so you can refer to them as `grocery:index`
in templates, avoiding clashes with other apps.

---

## 2. Connect to the Project urls.py

Open `grocery_project/urls.py` and add your app's URLs:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('grocery.urls', namespace='grocery')),
]
```

`include()` delegates anything not matched by `admin/` to your grocery app's urls.py.

---

## 3. Write the Views

Open `grocery/views.py` and write all 6 views:

```python
from django.shortcuts import render, get_object_or_404, redirect
from django.contrib import messages
from django.db.models import Sum
from .models import Category, Item


def index(request):
    categories = Category.objects.all()
    grand_total = Item.objects.aggregate(
        total=Sum('amount')
    )['total'] or 0

    return render(request, 'grocery/index.html', {
        'categories': categories,
        'grand_total': grand_total,
    })


def add_category(request):
    if request.method == 'POST':
        name = request.POST.get('name', '').strip()
        if name:
            Category.objects.create(name=name)
            messages.success(request, f'Category "{name}" created.')
        else:
            messages.error(request, 'Category name cannot be empty.')
    return redirect('grocery:index')


def delete_category(request, pk):
    category = get_object_or_404(Category, pk=pk)
    if request.method == 'POST':
        category.delete()
        messages.success(request, f'Category "{category.name}" deleted.')
    return redirect('grocery:index')


def add_item(request, pk):
    category = get_object_or_404(Category, pk=pk)
    if request.method == 'POST':
        name = request.POST.get('name', '').strip()
        quantity = request.POST.get('quantity')
        amount = request.POST.get('amount')
        if name and quantity and amount:
            Item.objects.create(
                category=category,
                name=name,
                quantity=quantity,
                amount=amount,
            )
            messages.success(request, f'"{name}" added to {category.name}.')
        else:
            messages.error(request, 'All fields are required.')
    return redirect('grocery:index')


def edit_item(request, pk):
    item = get_object_or_404(Item, pk=pk)
    if request.method == 'POST':
        name = request.POST.get('name', '').strip()
        quantity = request.POST.get('quantity')
        amount = request.POST.get('amount')
        if name and quantity and amount:
            item.name = name
            item.quantity = quantity
            item.amount = amount
            item.save()
            messages.success(request, f'"{name}" updated.')
        else:
            messages.error(request, 'All fields are required.')
        return redirect('grocery:index')

    # GET request — show edit form
    return render(request, 'grocery/edit_item.html', {'item': item})


def delete_item(request, pk):
    item = get_object_or_404(Item, pk=pk)
    if request.method == 'POST':
        name = item.name
        item.delete()
        messages.success(request, f'"{name}" deleted.')
    return redirect('grocery:index')
```

---

## What Each Pattern Means

| Pattern | Explanation |
|---|---|
| `get_object_or_404(Category, pk=pk)` | Fetches the object or returns a 404 page. Never use `.get()` directly — it raises an unhandled exception if the record doesn't exist. |
| `if request.method == 'POST'` | All data-changing operations only happen on POST. A GET request to a delete URL should never delete anything. |
| `redirect('grocery:index')` | After any POST, always redirect instead of rendering. Prevents duplicate submissions on browser refresh (Post/Redirect/Get pattern). |
| `request.POST.get('name', '').strip()` | Safely reads form data. `.get()` returns `None` instead of crashing if the field is missing. `.strip()` removes accidental whitespace. |
| `messages.success / messages.error` | Queues a one-time flash message shown on the next page load. Displayed in templates in Step 5. |

---

## 4. Create a Placeholder Template

We haven't built templates yet (Step 5), but create a minimal one so you can test URL routing now.

```bash
mkdir -p grocery/templates/grocery
```

Create `grocery/templates/grocery/index.html`:

```html
<!DOCTYPE html>
<html>
<head><title>Grocery</title></head>
<body>
  <h1>Grocery List</h1>
  <p>Grand total: {{ grand_total }}</p>
  {% for category in categories %}
    <h2>{{ category.name }}</h2>
  {% endfor %}
</body>
</html>
```

---

## 5. Test the Routes

```bash
python manage.py runserver
```

Visit these URLs and confirm no 500 errors:

- `http://127.0.0.1:8000/` — shows your categories
- `http://127.0.0.1:8000/category/add/` — redirects to index (POST only)
- `http://127.0.0.1:8000/admin/` — still works fine

To test `add_category`, open Django shell:

```bash
python manage.py shell
```

```python
from grocery.models import Category
Category.objects.create(name='Dairy')
Category.objects.all()   # should show <QuerySet [<Category: Dairy>]>
exit()
```

Refresh `http://127.0.0.1:8000/` — you should see "Dairy" appear.

---

## Checkpoint

Before moving to Step 4, confirm:

- [ ] No import errors when running the server
- [ ] `/` loads and shows categories you added in admin
- [ ] URL names resolve — in the shell:
  ```python
  from django.urls import reverse
  reverse('grocery:index')   # returns '/'
  ```

---

*Next: Step 4 — Forms & Validation*
