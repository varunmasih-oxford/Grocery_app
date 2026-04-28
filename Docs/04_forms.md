# Step 4 — Forms & Validation

In Step 3 we read form data directly from `request.POST`. That works, but Django has a better way — **Form classes** — that handle validation, error messages, and HTML rendering all in one place.

---

## Why Use Django Forms?

```
User submits HTML form
        ↓
Form class — validates fields, cleans data
        ↓
form.is_valid()?
   ✓ YES → Save to DB → redirect
   ✗ NO  → Show form with errors
```

---

## 1. Create `grocery/forms.py`

Create this file — it doesn't exist yet:

```python
from django import forms
from .models import Category, Item


class CategoryForm(forms.ModelForm):
    name = forms.CharField(
        max_length=100,
        widget=forms.TextInput(attrs={
            'placeholder': 'e.g. Dairy, Vegetables...',
            'class': 'form-input',
        })
    )

    class Meta:
        model = Category
        fields = ['name']


class ItemForm(forms.ModelForm):
    name = forms.CharField(
        max_length=200,
        widget=forms.TextInput(attrs={
            'placeholder': 'Item name',
            'class': 'form-input',
        })
    )
    quantity = forms.IntegerField(
        min_value=1,
        widget=forms.NumberInput(attrs={
            'placeholder': 'Qty',
            'class': 'form-input',
        })
    )
    amount = forms.DecimalField(
        min_value=0.01,
        max_digits=10,
        decimal_places=2,
        widget=forms.NumberInput(attrs={
            'placeholder': '0.00',
            'step': '0.01',
            'class': 'form-input',
        })
    )

    class Meta:
        model = Item
        fields = ['name', 'quantity', 'amount']
```

---

## What Each Piece Means

| Piece | Explanation |
|---|---|
| `forms.ModelForm` | A form tied directly to a model. Knows field types, max lengths, and constraints from the model automatically. |
| `class Meta` | Tells the form which model to use and which fields to include. We exclude `category` from `ItemForm` — it's set in the view from the URL. |
| `widget` | Controls how the field renders in HTML. Maps to `<input type="text">`, `<input type="number">` etc. The `attrs` dict adds HTML attributes. |
| `min_value=1` | Django validates this server-side. Even if someone bypasses the browser, the form rejects invalid values. |
| `step='0.01'` | Allows decimal input in the browser's number field. |

---

## 2. Update `views.py` to Use the Forms

```python
from django.shortcuts import render, get_object_or_404, redirect
from django.contrib import messages
from django.db.models import Sum
from .models import Category, Item
from .forms import CategoryForm, ItemForm


def index(request):
    categories = Category.objects.all()
    grand_total = Item.objects.aggregate(
        total=Sum('amount')
    )['total'] or 0
    category_form = CategoryForm()

    return render(request, 'grocery/index.html', {
        'categories': categories,
        'grand_total': grand_total,
        'category_form': category_form,
    })


def add_category(request):
    if request.method == 'POST':
        form = CategoryForm(request.POST)
        if form.is_valid():
            form.save()
            messages.success(request, f'Category "{form.cleaned_data["name"]}" created.')
        else:
            messages.error(request, 'Invalid category name.')
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
            item = form.save(commit=False)  # don't save yet
            item.category = category        # attach the category
            item.save()                     # now save
            messages.success(request, f'"{item.name}" added to {category.name}.')
        else:
            messages.error(request, 'Please fix the errors below.')
    return redirect('grocery:index')


def edit_item(request, pk):
    item = get_object_or_404(Item, pk=pk)
    if request.method == 'POST':
        form = ItemForm(request.POST, instance=item)  # bind form to existing item
        if form.is_valid():
            form.save()
            messages.success(request, f'"{item.name}" updated.')
            return redirect('grocery:index')
    else:
        form = ItemForm(instance=item)  # pre-fill form with existing data

    return render(request, 'grocery/edit_item.html', {
        'form': form,
        'item': item,
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

## Three Key Form Patterns Explained

### Pattern 1 — Create a new object
```python
form = ItemForm(request.POST)
if form.is_valid():
    form.save()
```

### Pattern 2 — Create with an extra field not in the form (`commit=False`)
```python
item = form.save(commit=False)  # builds the object but doesn't hit the DB
item.category = category         # set the field manually
item.save()                      # now write to DB
```
Used in `add_item` because `category` isn't in the form — it comes from the URL.

### Pattern 3 — Edit an existing object (`instance`)
```python
# GET — pre-fill the form with existing data
form = ItemForm(instance=item)

# POST — bind submitted data to the existing object
form = ItemForm(request.POST, instance=item)
form.save()  # updates the existing row, doesn't create a new one
```

---

## 3. Create the Edit Item Template

Create `grocery/templates/grocery/edit_item.html`:

```html
<!DOCTYPE html>
<html>
<head><title>Edit Item</title></head>
<body>
  <h1>Edit {{ item.name }}</h1>

  <form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Save changes</button>
  </form>

  <a href="{% url 'grocery:index' %}">Cancel</a>
</body>
</html>
```

| Template tag | Explanation |
|---|---|
| `{% csrf_token %}` | Required in every POST form. A hidden security token that prevents cross-site request forgery. Form gets 403 Forbidden without it. |
| `{{ form.as_p }}` | Renders all form fields wrapped in `<p>` tags, including error messages. Quick way to render a form — styled properly in Step 6. |

---

## 4. Test Validation in the Shell

```bash
python manage.py shell
```

```python
from grocery.forms import ItemForm

# should be INVALID — empty name
form = ItemForm(data={'name': '', 'quantity': 2, 'amount': 50})
print(form.is_valid())   # False
print(form.errors)       # {'name': ['This field is required.']}

# should be INVALID — negative quantity
form = ItemForm(data={'name': 'Milk', 'quantity': -1, 'amount': 50})
print(form.is_valid())   # False
print(form.errors)       # {'quantity': [...]}

# should be VALID
form = ItemForm(data={'name': 'Milk', 'quantity': 2, 'amount': 50})
print(form.is_valid())   # True
print(form.cleaned_data) # {'name': 'Milk', 'quantity': 2, 'amount': Decimal('50')}
```

> **Note:** `form.cleaned_data` — after validation passes, Django stores sanitized,
> correctly-typed values here. Notice `amount` becomes a `Decimal`, not a string.

---

## Checkpoint

Before moving to Step 5, confirm:

- [ ] `forms.py` created with `CategoryForm` and `ItemForm`
- [ ] `views.py` updated to use forms with `is_valid()` and `form.save()`
- [ ] Shell test shows invalid forms correctly reporting errors
- [ ] `edit_item.html` created and renders without errors at `/item/1/edit/`

---

*Next: Step 5 — Templates: Base Layout & Index Page*
