
## 📘 Book Library Tracker – Implementation Guide

### ✅ Step 1: Set Up the Project Folder
```bash
cp -r lesson_crud_app/ book_library/
cd book_library/
```

Rename files or variables as needed—but we’ll do that systematically below.

---

### ✅ Step 2: Update the Database (`database.py`)

Replace your `lessons` table with the `books` table:

```python
# database.py
import sqlite3

def init_db():
    conn = sqlite3.connect('app.db')
    c = conn.cursor()
    c.execute('''
        CREATE TABLE IF NOT EXISTS books (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            title TEXT NOT NULL,
            author TEXT NOT NULL,
            status TEXT NOT NULL
        )
    ''')
    conn.commit()
    conn.close()
```

> 💡 The `status` field will hold values like `"To Read"`, `"Reading"`, or `"Completed"`.

Call `init_db()` at the bottom of `app.py` (or in a setup block) to ensure the table exists.

---

### ✅ Step 3: Update `app.py`

Replace all lesson-related logic with book-related logic.

#### Full Example `app.py` (minimal & clean):

```python
# app.py
from flask import Flask, render_template, request, redirect, flash
import sqlite3

app = Flask(__name__)
app.secret_key = 'your_secret_key_here'

def get_db_connection():
    conn = sqlite3.connect('app.db')
    conn.row_factory = sqlite3.Row  # enables dict-like access
    return conn

@app.route('/')
def index():
    conn = get_db_connection()
    books = conn.execute('SELECT * FROM books ORDER BY id DESC').fetchall()
    conn.close()
    return render_template('index.html', books=books)

@app.route('/add', methods=['POST'])
def add_book():
    title = request.form['title'].strip()
    author = request.form['author'].strip()
    status = request.form['status']

    if not title or not author or not status:
        flash('All fields are required!')
        return redirect('/')

    conn = get_db_connection()
    conn.execute('INSERT INTO books (title, author, status) VALUES (?, ?, ?)',
                 (title, author, status))
    conn.commit()
    conn.close()
    flash('Book added successfully!')
    return redirect('/')

@app.route('/edit/<int:id>', methods=['GET', 'POST'])
def edit_book(id):
    conn = get_db_connection()
    book = conn.execute('SELECT * FROM books WHERE id = ?', (id,)).fetchone()

    if request.method == 'POST':
        title = request.form['title'].strip()
        author = request.form['author'].strip()
        status = request.form['status']

        if not title or not author or not status:
            flash('All fields are required!')
        else:
            conn.execute('UPDATE books SET title = ?, author = ?, status = ? WHERE id = ?',
                         (title, author, status, id))
            conn.commit()
            conn.close()
            flash('Book updated!')
            return redirect('/')

    conn.close()
    return render_template('edit.html', book=book)

@app.route('/delete/<int:id>')
def delete_book(id):
    conn = get_db_connection()
    conn.execute('DELETE FROM books WHERE id = ?', (id,))
    conn.commit()
    conn.close()
    flash('Book deleted!')
    return redirect('/')
```

> 🔐 Don’t forget to set a real `secret_key` for sessions (used by `flash`).

---

### ✅ Step 4: Update Templates

#### `templates/base.html`
Keep your responsive layout and flash messages:
```html
<!DOCTYPE html>
<html>
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
  <title>📚 My Book Library</title>
</head>
<body>
  <div class="container">
    <h1>📚 My Book Library</h1>
    
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
Display books and include a filter by status (optional but recommended):

```html
<!-- Add new book form -->
<form method="POST" action="/add">
  <input type="text" name="title" placeholder="Book Title" required>
  <input type="text" name="author" placeholder="Author" required>
  <select name="status" required>
    <option value="">— Select Status —</option>
    <option value="To Read">To Read</option>
    <option value="Reading">Reading</option>
    <option value="Completed">Completed</option>
  </select>
  <button type="submit">➕ Add Book</button>
</form>

<hr>

<!-- Display books grouped by status (optional enhancement) -->
{% set statuses = ["To Read", "Reading", "Completed"] %}
{% for status in statuses %}
  {% set books_in_status = [] %}
  {% for book in books if book.status == status %}
    {% if books_in_status.append(book) %}{% endif %}
  {% endfor %}

  {% if books_in_status %}
    <h2>📖 {{ status }}</h2>
    <ul class="book-list">
      {% for book in books_in_status %}
        <li>
          <strong>{{ book.title }}</strong> by {{ book.author }}
          <a href="/edit/{{ book.id }}">✏️ Edit</a>
          <a href="/delete/{{ book.id }}" onclick="return confirm('Delete?')">🗑️ Delete</a>
        </li>
      {% endfor %}
    </ul>
  {% endif %}
{% endfor %}
```

> 🌟 This groups books visually by status—clean and user-friendly!

---

#### `templates/edit.html`
Edit form with dropdown for status:

```html
<h2>✏️ Edit Book</h2>
<form method="POST">
  <input type="text" name="title" value="{{ book.title }}" required>
  <input type="text" name="author" value="{{ book.author }}" required>
  
  <select name="status" required>
    <option value="To Read" {% if book.status == 'To Read' %}selected{% endif %}>To Read</option>
    <option value="Reading" {% if book.status == 'Reading' %}selected{% endif %}>Reading</option>
    <option value="Completed" {% if book.status == 'Completed' %}selected{% endif %}>Completed</option>
  </select>

  <button type="submit">💾 Save Changes</button>
  <a href="/">← Cancel</a>
</form>
```

---

### ✅ Step 5: Styling (`static/style.css`)
Reuse your responsive CSS. Add minor tweaks:

```css
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

input, select, button {
  padding: 8px;
  margin: 5px 0;
  width: 100%;
  box-sizing: border-box;
}

.book-list {
  list-style: none;
  padding: 0;
}

.book-list li {
  padding: 10px;
  margin: 8px 0;
  background: #f9f9f9;
  border-radius: 4px;
}

.alert {
  padding: 10px;
  background: #e7f3ff;
  color: #004085;
  border: 1px solid #bee5eb;
  border-radius: 4px;
  margin-bottom: 15px;
}

@media (max-width: 600px) {
  .container { padding: 10px; }
}
```

---

### ✅ Step 6: Test Your App
Run it:
```bash
python app.py
```
Go to `http://localhost:5000` and:
- Add a book titled *"Dune"* by *"Frank Herbert"* with status *"To Read"*
- Edit it to *"Reading"*
- Delete it

✅ All should work smoothly with flash feedback.

---

### ✅ Step 7: Write `README.md`

```markdown
# 📖 Book Library Tracker

A Flask web app to manage your personal book collection with reading status.

## Features
- Add, edit, and delete books
- Track reading status: "To Read", "Reading", or "Completed"
- Books grouped by status on the homepage
- Fully responsive design

## How to Run
1. Install Flask:  
   ```bash
   pip install flask
   ```
2. Run the app:  
   ```bash
   python app.py
   ```
3. Open your browser to: http://localhost:5000

