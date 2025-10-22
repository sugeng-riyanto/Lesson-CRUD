
## 🛒 Shopping List App – Implementation Guide

### ✅ Step 1: Set Up the Project Folder
```bash
cp -r lesson_crud_app/ shopping_list/
cd shopping_list/
```

We’ll now customize everything for shopping items.

---

### ✅ Step 2: Update the Database (`database.py`)

Replace the `lessons` table with the `items` table as specified:

```python
# database.py
import sqlite3

def init_db():
    conn = sqlite3.connect('app.db')
    c = conn.cursor()
    c.execute('''
        CREATE TABLE IF NOT EXISTS items (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            quantity INTEGER DEFAULT 1,
            category TEXT NOT NULL
        )
    ''')
    conn.commit()
    conn.close()
```

> 💡 Common categories: `"Produce"`, `"Dairy"`, `"Meat"`, `"Bakery"`, `"Household"`, `"Frozen"`, etc.

Call `init_db()` in `app.py` (see next step).

---

### ✅ Step 3: Update `app.py`

Here’s a clean, complete `app.py` for the Shopping List:

```python
# app.py
from flask import Flask, render_template, request, redirect, flash
import sqlite3

app = Flask(__name__)
app.secret_key = 'your_secret_key_here'

def get_db_connection():
    conn = sqlite3.connect('app.db')
    conn.row_factory = sqlite3.Row  # allows dict-like access (e.g., item['name'])
    return conn

@app.route('/')
def index():
    conn = get_db_connection()
    items = conn.execute('SELECT * FROM items ORDER BY category, name').fetchall()
    conn.close()

    # Group items by category for display
    categories = {}
    for item in items:
        cat = item['category']
        if cat not in categories:
            categories[cat] = []
        categories[cat].append(item)

    return render_template('index.html', categories=categories)

@app.route('/add', methods=['POST'])
def add_item():
    name = request.form['name'].strip()
    quantity = request.form.get('quantity', '1')
    category = request.form['category'].strip()

    if not name or not category:
        flash('Name and category are required!')
        return redirect('/')

    try:
        quantity = int(quantity)
        if quantity < 1:
            quantity = 1
    except ValueError:
        quantity = 1

    conn = get_db_connection()
    conn.execute('INSERT INTO items (name, quantity, category) VALUES (?, ?, ?)',
                 (name, quantity, category))
    conn.commit()
    conn.close()
    flash(f'✅ Added {name}!')
    return redirect('/')

@app.route('/edit/<int:id>', methods=['GET', 'POST'])
def edit_item(id):
    conn = get_db_connection()
    item = conn.execute('SELECT * FROM items WHERE id = ?', (id,)).fetchone()

    if request.method == 'POST':
        name = request.form['name'].strip()
        quantity = request.form.get('quantity', '1')
        category = request.form['category'].strip()

        if not name or not category:
            flash('Name and category are required!')
        else:
            try:
                quantity = int(quantity)
                if quantity < 1:
                    quantity = 1
            except ValueError:
                quantity = 1

            conn.execute('UPDATE items SET name = ?, quantity = ?, category = ? WHERE id = ?',
                         (name, quantity, category, id))
            conn.commit()
            conn.close()
            flash('🛒 Item updated!')
            return redirect('/')

    conn.close()
    return render_template('edit.html', item=item)

@app.route('/delete/<int:id>')
def delete_item(id):
    conn = get_db_connection()
    conn.execute('DELETE FROM items WHERE id = ?', (id,))
    conn.commit()
    conn.close()
    flash('🗑️ Item deleted!')
    return redirect('/')
```

> 🔐 Don’t forget to set a real secret key in production.

---

### ✅ Step 4: Update Templates

#### `templates/base.html`
Keep it clean and responsive:

```html
<!DOCTYPE html>
<html>
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
  <title>🛒 Shopping List</title>
</head>
<body>
  <div class="container">
    <h1>🛒 My Shopping List</h1>

    {% with messages = get_flashed_messages() %}
      {% if messages %}
        <div class="alert">
          {% for msg in messages %}{{ msg }}{% endfor %}
        </div>
      {% endif %}
    {% endwith %}

    {% block content %}{% endblock %}
  </div>
</body>
</html>
```

---

#### `templates/index.html`
Group items by category and include an "Add Item" form:

```html
<!-- Add New Item Form -->
<form method="POST" action="/add">
  <input type="text" name="name" placeholder="Item name (e.g., Apples)" required>
  
  <input type="number" name="quantity" value="1" min="1" placeholder="Qty">
  
  <input type="text" name="category" placeholder="Category (e.g., Produce)" required>
  
  <button type="submit">➕ Add to List</button>
</form>

<hr>

<!-- Display Items Grouped by Category -->
{% if categories %}
  {% for category, items in categories.items() %}
    <h2>📦 {{ category }}</h2>
    <ul class="item-list">
      {% for item in items %}
        <li>
          <strong>{{ item.name }}</strong>
          {% if item.quantity != 1 %} × {{ item.quantity }}{% endif %}
          <a href="/edit/{{ item.id }}">✏️ Edit</a>
          <a href="/delete/{{ item.id }}" onclick="return confirm('Remove from list?')">🗑️ Delete</a>
        </li>
      {% endfor %}
    </ul>
  {% endfor %}
{% else %}
  <p>🛒 Your list is empty! Add your first item.</p>
{% endif %}
```

> 🌟 This groups items visually—much easier to shop from!

---

#### `templates/edit.html`
Edit form with quantity and category fields:

```html
<h2>✏️ Edit Item</h2>
<form method="POST">
  <input type="text" name="name" value="{{ item.name }}" placeholder="Item name" required>
  
  <input type="number" name="quantity" value="{{ item.quantity }}" min="1">
  
  <input type="text" name="category" value="{{ item.category }}" placeholder="Category" required>
  
  <button type="submit">💾 Save Changes</button>
  <a href="/">← Cancel</a>
</form>
```

> 💡 You can later enhance this with a `<select>` dropdown for categories if you want fixed options.

---

### ✅ Step 5: Styling (`static/style.css`)

Reuse your responsive base, with small tweaks:

```css
.container {
  max-width: 700px;
  margin: 0 auto;
  padding: 20px;
  font-family: system-ui, sans-serif;
}

input, button {
  padding: 8px;
  margin: 6px 0;
  width: 100%;
  box-sizing: border-box;
}

input[type="number"] {
  width: 80px;
}

.item-list {
  list-style: none;
  padding: 0;
}

.item-list li {
  padding: 10px;
  margin: 8px 0;
  background: #f8f9fa;
  border-left: 4px solid #4CAF50;
  border-radius: 4px;
}

.alert {
  padding: 10px;
  background: #fff8e1;
  color: #5d4037;
  border: 1px solid #ffe082;
  border-radius: 4px;
  margin-bottom: 15px;
}

button {
  background: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
}

button:hover {
  background: #45a049;
}

a {
  margin-left: 10px;
  text-decoration: none;
  color: #1976d2;
}

@media (max-width: 600px) {
  .container { padding: 12px; }
  input[type="number"] { width: 70px; }
}
```

---

### ✅ Step 6: Test Your App
Run:
```bash
python app.py
```

Try:
- Add **"Milk"**, quantity **2**, category **"Dairy"**
- Add **"Bananas"**, quantity **6**, category **"Produce"**
- Edit milk to quantity **1**
- Delete bananas

✅ Items should appear grouped under their categories.

---

### ✅ Step 7: Write `README.md`

```markdown
# 🛒 Shopping List App

A responsive Flask app to manage grocery or household shopping items with quantity and category.

## Features
- Add, edit, and delete shopping items
- Organize items by category (e.g., Produce, Dairy)
- Quantity support (defaults to 1)
- Mobile-friendly design

## How to Run
1. Install Flask:
   ```bash
   pip install flask
   ```
2. Run the app:
   ```bash
   python app.py
   ```
3. Visit: http://localhost:5000

> Built with Flask, SQLite3, and vanilla CSS.
```

---

