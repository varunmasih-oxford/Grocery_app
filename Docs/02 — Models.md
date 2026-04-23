# Step 2 — Models: Category & Item

Models are Python classes that Django translates into database tables. Each class = one table. Each attribute = one column.

---

## The Plan

We need two tables with a relationship:

- One `Category` has many `Item`s
- Each `Item` belongs to exactly one `Category`

```
Category                        Item
─────────────────               ─────────────────────────
id          (auto)     1──┐     id          (auto)
name        CharField     └──▶  name        CharField
created_at  DateTimeField  many quantity    PositiveIntegerField
                               amount      DecimalField
                               category_id ← ForeignKey
```

---

## 1. Write the Models

Open `grocery/models.py` and replace everything with this:

```python
from django.db import models


class Category(models.Model):
    name = models.CharField(max_length=100)
    created_at = models.DateTimeField(auto_now_add=True)

    def subtotal(self):
        # sum all item amounts belonging to this category
        result = self.items.aggregate(total=models.Sum('amount'))['total']
        return result or 0

    def __str__(self):
        return self.name

    class Meta:
        ordering = ['created_at']


class Item(models.Model):
    category = models.ForeignKey(
        Category,
        on_delete=models.CASCADE,
        related_name='items'   # lets you do category.items.all()
    )
    name = models.CharField(max_length=200)
    quantity = models.PositiveIntegerField(default=1)
    amount = models.DecimalField(max_digits=10, decimal_places=2)

    def __str__(self):
        return self.name

    class Meta:
        ordering = ['name']
```

---

## What Each Piece Means

| Field | Explanation |
|---|---|
| `CharField(max_length=100)` | Text column with a max length. Required for all string fields. |
| `auto_now_add=True` | Automatically sets timestamp on creation. Never set manually. |
| `PositiveIntegerField` | Only allows whole numbers ≥ 0. Perfect for quantity. |
| `DecimalField(max_digits=10, decimal_places=2)` | Stores money values precisely. Never use `FloatField` for currency. |
| `ForeignKey` | Creates the relationship. Django adds `category_id` column to `Item` automatically. |
| `on_delete=CASCADE` | If a `Category` is deleted, all its `Item`s are deleted too. |
| `related_name='items'` | Lets you write `category.items.all()`. Without this, Django defaults to `category.item_set.all()`. |

### About `subtotal()`

A Python method, not a column. It uses `aggregate` to run a `SUM` SQL query on the fly:

```python
def subtotal(self):
    result = self.items.aggregate(total=models.Sum('amount'))['total']
    return result or 0
```

Returns `0` instead of `None` if the category has no items yet.

---

## 2. Run Migrations

Every time you change models, run two commands:

```bash
# Generate the migration file
python manage.py makemigrations
```

This creates `grocery/migrations/0001_initial.py` — a script describing what changed.

```bash
# Apply the migration to the database
python manage.py migrate
```

This creates the actual tables in `db.sqlite3`. You should see:

```
Applying grocery.0001_initial... OK
```

---

## 3. Register Models in Admin

Open `grocery/admin.py` and add:

```python
from django.contrib import admin
from .models import Category, Item

admin.site.register(Category)
admin.site.register(Item)
```

This gives you a visual interface to add test data without writing views yet.

---

## 4. Create a Superuser & Test in Admin

```bash
python manage.py createsuperuser
```

Enter a username, email (optional), and password. Then run the server:

```bash
python manage.py runserver
```

Go to `http://127.0.0.1:8000/admin` and try:

- Adding a `Category` called "Vegetables"
- Adding an `Item` inside it — name "Tomato", quantity 2, amount 40.00

If you can save both and see them listed, your models are working perfectly.

---

## What Django Did Behind the Scenes

When you ran `migrate`, Django created two SQL tables:

```sql
CREATE TABLE grocery_category (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    name        VARCHAR(100) NOT NULL,
    created_at  DATETIME NOT NULL
);

CREATE TABLE grocery_item (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    name        VARCHAR(200) NOT NULL,
    quantity    INTEGER NOT NULL,
    amount      DECIMAL(10, 2) NOT NULL,
    category_id INTEGER NOT NULL REFERENCES grocery_category(id)
);
```

You never wrote this SQL — Django generated it from your Python classes.

---

## Checkpoint

Before moving to Step 3, confirm:

- [ ] `makemigrations` and `migrate` ran without errors
- [ ] You can log into `/admin` and create a Category + Item
- [ ] Deleting a Category in admin also deletes its Items

---

*Next: Step 3 — URLs & Views*
