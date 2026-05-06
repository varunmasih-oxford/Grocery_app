# Step 7 — Grand Total & Flash Messages

In this step we'll make the grand total bulletproof, ensure flash messages work perfectly
across all actions, and add a context processor so `grand_total` is available in every
template automatically — not just when the view passes it.

---

## The Problem We're Solving

Right now `grand_total` is passed manually in the `index` view. But `base.html` uses
`{{ grand_total }}` — if any other view renders a page (like `edit_item`) without passing
`grand_total`, the total bar shows nothing.

```
WITHOUT context processor:
  index view      → passes grand_total ✓
  edit_item view  → no grand_total     ✗
  future views    → must remember      ✗

WITH context processor:
  ALL views → grand_total always in every template ✓
```

---

## 1. Create a Context Processor

Create a new file `grocery/context_processors.py`:

```python
from django.db.models import Sum
from .models import Item


def grand_total(request):
    """
    Makes grand_total available in every template automatically.
    Django calls this function for every request and merges
    the returned dict into the template context.
    """
    total = Item.objects.aggregate(
        total=Sum('amount')
    )['total'] or 0

    return {'grand_total': total}
```

This is just a function that takes `request` and returns a dictionary. Django merges
this dict into every template's context automatically.

---

## 2. Register It in `settings.py`

Find the `TEMPLATES` setting and add it to `context_processors`:

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
                'grocery.context_processors.grand_total',  # ← add this
            ],
        },
    },
]
```

---

## 3. Clean Up `views.py`

Now that `grand_total` comes from the context processor, remove it from every view:

```python
from django.shortcuts import render, get_object_or_404, redirect
from django.contrib import messages
from .models import Category, Item
from .forms import CategoryForm, ItemForm


def index(request):
    categories = Category.objects.prefetch_related('items').all()
    category_form = CategoryForm()

    return render(request, 'grocery/index.html', {
        'categories': categories,
        'category_form': category_form,
        # grand_total removed — comes from context processor now
    })


def add_category(request):
    if request.method == 'POST':
        form = CategoryForm(request.POST)
        if form.is_valid():
            form.save()
            messages.success(
                request,
                f'Category "{form.cleaned_data["name"]}" created.'
            )
        else:
            messages.error(request, 'Category name cannot be empty.')
    return redirect('grocery:index')


def delete_category(request, pk):
    category = get_object_or_404(Category, pk=pk)
    if request.method == 'POST':
        name = category.name
        category.delete()
        messages.success(request, f'Category "{name}" deleted.')
    return redirect('grocery:index')


def add_item(request, pk):
    category = get_object_or_404(Category, pk=pk)
    if request.method == 'POST':
        form = ItemForm(request.POST)
        if form.is_valid():
            item = form.save(commit=False)
            item.category = category
            item.save()
            messages.success(
                request,
                f'"{item.name}" added to {category.name}.'
            )
        else:
            messages.error(request, 'Please check all fields are valid.')
    return redirect('grocery:index')


def edit_item(request, pk):
    item = get_object_or_404(Item, pk=pk)
    if request.method == 'POST':
        form = ItemForm(request.POST, instance=item)
        if form.is_valid():
            form.save()
            messages.success(request, f'"{item.name}" updated successfully.')
            return redirect('grocery:index')
        else:
            messages.error(request, 'Please fix the errors below.')
    else:
        form = ItemForm(instance=item)

    return render(request, 'grocery/edit_item.html', {
        'form': form,
        'item': item,
        # grand_total available automatically via context processor
    })


def delete_item(request, pk):
    item = get_object_or_404(Item, pk=pk)
    if request.method == 'POST':
        name = item.name
        item.delete()
        messages.success(request, f'"{name}" deleted.')
    return redirect('grocery:index')
```

---

## 4. Configure the Messages Framework

Add this to the bottom of `settings.py`:

```python
from django.contrib.messages import constants as messages_constants

MESSAGE_TAGS = {
    messages_constants.DEBUG:   'debug',
    messages_constants.INFO:    'info',
    messages_constants.SUCCESS: 'success',
    messages_constants.WARNING: 'warning',
    messages_constants.ERROR:   'error',
}
```

This maps Django's internal message levels to CSS class names. Our `style.css` already
has `.messages li.success` and `.messages li.error` — this wires them together.

---

## 5. Message Coverage Across All Views

| Action | Message type | Example text |
|---|---|---|
| Category created | success | `Category "Dairy" created.` |
| Category deleted | success | `Category "Dairy" deleted.` |
| Category name empty | error | `Category name cannot be empty.` |
| Item added | success | `"Milk" added to Dairy.` |
| Item form invalid | error | `Please check all fields are valid.` |
| Item updated | success | `"Milk" updated successfully.` |
| Item edit invalid | error | `Please fix the errors below.` |
| Item deleted | success | `"Milk" deleted.` |

---

## 6. How the Messages Framework Works Internally

```
1. View calls messages.success(request, "Done!")
        ↓
2. Message stored in the session (server-side)
        ↓
3. Next request — context processor reads messages from session
        ↓
4. Template renders {{ message }} inside {% for message in messages %}
        ↓
5. Message cleared from session — gone on next refresh
```

> **Why redirect after POST?** The message shows on the redirected GET request,
> then clears. If you render directly without redirecting, the message won't
> clear properly on refresh.

---

## 7. Verify Middleware & Installed Apps

If messages aren't showing, check `settings.py`:

```python
INSTALLED_APPS = [
    ...
    'django.contrib.messages',   # ← must be present
    ...
]

MIDDLEWARE = [
    ...
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',  # ← must be present
    ...
]
```

---

## 8. Manual Test Scenarios

### Scenario A — Grand total on edit page
1. Go to `/` — note the grand total
2. Click Edit on any item
3. Grand total bar should show the correct total on the edit page too

### Scenario B — Grand total updates after add
1. Note current grand total
2. Add an item with amount ₹50
3. After redirect, grand total should increase by ₹50

### Scenario C — Grand total updates after delete
1. Note current grand total
2. Delete an item with amount ₹30
3. After redirect, grand total should decrease by ₹30

### Scenario D — Flash messages are one-time
```
Add a category → message appears on next page load ✓
Refresh the page → message is gone                 ✓
```

---

## Checkpoint

Before moving to Step 8, confirm:

- [ ] `context_processors.py` created and registered in `settings.py`
- [ ] Grand total shows correctly on both `/` and `/item/<id>/edit/`
- [ ] Grand total updates correctly after adding/deleting items
- [ ] Flash messages appear once and disappear on refresh
- [ ] `MESSAGE_TAGS` added to `settings.py`
- [ ] No `grand_total` passed manually in any 
