
# 🎓 **Comprehensive Flask CRUD Lesson with SQLite3 & Responsive Design**  
**Duration:** 60 Minutes  
**Level:** Beginner to Intermediate  
**Goal:** Build a fully functional, responsive **Lesson Management App** using Flask, SQLite3, HTML, and CSS.

---

## 📋 **Lesson Outline**

| Time | Topic |
|------|-------|
| 0–5 min | Introduction & Setup |
| 5–15 min | Database Setup (`database.py`) |
| 15–35 min | Flask App Logic (`app.py`) |
| 35–50 min | HTML Templates + Responsive CSS |
| 50–60 min | Testing, Debugging & Best Practices |

---

## 🛠️ **Step 1: Project Setup (5 min)**

### ✅ Create Project Folder
```bash
mkdir lesson_crud_app
cd lesson_crud_app
```

### ✅ Install Flask
```bash
pip install flask
```

### ✅ Create Folder Structure
```
lesson_crud_app/
├── app.py                 # Main Flask app
├── database.py            # Database logic
├── static/
│   └── style.css          # External CSS (responsive design)
└── templates/
    ├── base.html          # Base layout (shared structure)
    ├── index.html         # Home page (Read + Delete)
    └── edit.html          # Form for Create + Update
```

> 💡 **Tip**: Flask automatically looks for `templates/` and `static/` folders.

---

## 🗃️ **Step 2: Database Setup – `database.py` (10 min)**

Create `database.py`:

```python
# database.py
import sqlite3
from contextlib import closing

def init_db():
    """Create the 'lessons' table if it doesn't exist."""
    with closing(sqlite3.connect('lessons.db')) as conn:
        with closing(conn.cursor()) as cursor:
            cursor.execute('''
                CREATE TABLE IF NOT EXISTS lessons (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    title TEXT NOT NULL,
                    description TEXT NOT NULL,
                    duration INTEGER NOT NULL  -- in minutes
                )
            ''')
            conn.commit()

def get_db_connection():
    """Open a new database connection that returns rows as dictionaries."""
    conn = sqlite3.connect('lessons.db')
    conn.row_factory = sqlite3.Row  # Allows access by column name (e.g., row['title'])
    return conn

# Initialize database when this file is imported
init_db()
```

> ✅ **Why this works**:  
> - `init_db()` runs once to create the table.  
> - `get_db_connection()` safely connects and enables named column access.

---

## ⚙️ **Step 3: Flask App – `app.py` (20 min)**

Create `app.py`:

```python
# app.py
from flask import Flask, render_template, request, redirect, url_for, flash
from database import get_db_connection

app = Flask(__name__)
app.secret_key = 'super_secret_key_123'  # Needed for flash messages

# ===== HOME PAGE: READ all lessons =====
@app.route('/')
def index():
    conn = get_db_connection()
    lessons = conn.execute('SELECT * FROM lessons ORDER BY id DESC').fetchall()
    conn.close()
    return render_template('index.html', lessons=lessons)

# ===== CREATE: Add new lesson =====
@app.route('/create', methods=['GET', 'POST'])
def create():
    if request.method == 'POST':
        title = request.form['title']
        description = request.form['description']
        duration = request.form['duration']
        
        # Validation
        if not title or not description or not duration:
            flash('All fields are required!', 'error')
        else:
            conn = get_db_connection()
            conn.execute(
                'INSERT INTO lessons (title, description, duration) VALUES (?, ?, ?)',
                (title, description, int(duration))
            )
            conn.commit()
            conn.close()
            flash('Lesson created successfully!', 'success')
            return redirect(url_for('index'))
    
    return render_template('edit.html', lesson=None, title="Create New Lesson")

# ===== UPDATE: Edit existing lesson =====
@app.route('/<int:id>/edit', methods=['GET', 'POST'])
def edit(id):
    conn = get_db_connection()
    lesson = conn.execute('SELECT * FROM lessons WHERE id = ?', (id,)).fetchone()
    conn.close()
    
    if lesson is None:
        flash('Lesson not found!', 'error')
        return redirect(url_for('index'))
    
    if request.method == 'POST':
        title = request.form['title']
        description = request.form['description']
        duration = request.form['duration']
        
        if not title or not description or not duration:
            flash('All fields are required!', 'error')
        else:
            conn = get_db_connection()
            conn.execute(
                'UPDATE lessons SET title = ?, description = ?, duration = ? WHERE id = ?',
                (title, description, int(duration), id)
            )
            conn.commit()
            conn.close()
            flash('Lesson updated successfully!', 'success')
            return redirect(url_for('index'))
    
    return render_template('edit.html', lesson=lesson, title="Edit Lesson")

# ===== DELETE: Remove lesson =====
@app.route('/<int:id>/delete', methods=['POST'])
def delete(id):
    conn = get_db_connection()
    lesson = conn.execute('SELECT * FROM lessons WHERE id = ?', (id,)).fetchone()
    
    if lesson is None:
        conn.close()
        flash('Lesson not found!', 'error')
        return redirect(url_for('index'))
    
    conn.execute('DELETE FROM lessons WHERE id = ?', (id,))
    conn.commit()
    conn.close()
    flash('Lesson deleted successfully!', 'success')
    return redirect(url_for('index'))

# ===== RUN APP =====
if __name__ == '__main__':
    app.run(debug=True)
```

> 🔒 **Security Notes**:  
> - Uses **parameterized queries** (`?`) to prevent SQL injection.  
> - Delete only via **POST** (not GET) to avoid accidental deletions.  
> - All inputs **validated** before saving.

---

## 🎨 **Step 4: HTML + Responsive CSS (15 min)**

### 📁 Create `static/style.css`

```css
/* static/style.css */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Segoe UI', system-ui, sans-serif;
    line-height: 1.6;
    color: #333;
    background: #f8f9fa;
}

.container {
    width: 90%;
    max-width: 1000px;
    margin: 0 auto;
    padding: 20px;
}

/* Header */
header {
    background: #28a745;
    color: white;
    padding: 1rem 0;
    margin-bottom: 2rem;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

header h1 {
    text-align: center;
    font-size: 2rem;
}

/* Alerts */
.alert {
    padding: 12px 16px;
    margin-bottom: 20px;
    border-radius: 6px;
    font-weight: 500;
}
.alert-success { background: #d4edda; color: #155724; border: 1px solid #c3e6cb; }
.alert-error { background: #f8d7da; color: #721c24; border: 1px solid #f5c6cb; }

/* Buttons */
.btn {
    display: inline-block;
    padding: 10px 18px;
    text-decoration: none;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    font-size: 16px;
    transition: opacity 0.2s;
}
.btn:hover { opacity: 0.9; }

.btn-primary { background: #28a745; color: white; }
.btn-secondary { background: #6c757d; color: white; }
.btn-danger { background: #dc3545; color: white; }

/* Forms */
.form-group {
    margin-bottom: 18px;
}
.form-group label {
    display: block;
    margin-bottom: 6px;
    font-weight: 600;
}
.form-group input,
.form-group textarea {
    width: 100%;
    padding: 10px;
    border: 1px solid #ced4da;
    border-radius: 6px;
    font-size: 16px;
}
.form-group textarea {
    height: 120px;
    resize: vertical;
}

/* Lesson Cards */
.lesson-card {
    background: white;
    padding: 20px;
    margin-bottom: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.08);
    border-left: 4px solid #28a745;
}
.lesson-card h3 {
    color: #28a745;
    margin-bottom: 10px;
}
.lesson-meta {
    display: flex;
    justify-content: space-between;
    margin-top: 15px;
    padding-top: 15px;
    border-top: 1px solid #eee;
}
.lesson-duration {
    background: #e9ecef;
    padding: 4px 10px;
    border-radius: 20px;
    font-size: 14px;
    font-weight: 600;
}
.action-buttons {
    display: flex;
    gap: 10px;
}

/* Responsive Design */
@media (max-width: 768px) {
    .container { width: 95%; padding: 15px; }
    header h1 { font-size: 1.6rem; }
    .lesson-meta { flex-direction: column; gap: 10px; }
    .action-buttons { flex-direction: column; }
    .btn { width: 100%; text-align: center; }
}
```

### 📄 Update `templates/base.html`

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Lesson Manager{% endblock %}</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
    <header>
        <div class="container">
            <h1>📚 Lesson Manager</h1>
        </div>
    </header>

    <div class="container">
        <!-- Flash Messages -->
        {% with messages = get_flashed_messages(with_categories=true) %}
            {% if messages %}
                {% for category, message in messages %}
                    <div class="alert alert-{{ category }}">{{ message }}</div>
                {% endfor %}
            {% endif %}
        {% endwith %}

        <!-- Page Content -->
        {% block content %}{% endblock %}
    </div>
</body>
</html>
```

### 📄 Create `templates/index.html`

```html
<!-- templates/index.html -->
{% extends "base.html" %}

{% block content %}
<div style="text-align: center; margin-bottom: 25px;">
    <a href="{{ url_for('create') }}" class="btn btn-primary">➕ Add New Lesson</a>
</div>

{% if lessons %}
    {% for lesson in lessons %}
    <div class="lesson-card">
        <h3>{{ lesson.title }}</h3>
        <p>{{ lesson.description }}</p>
        <div class="lesson-meta">
            <span class="lesson-duration">⏱️ {{ lesson.duration }} min</span>
            <div class="action-buttons">
                <a href="{{ url_for('edit', id=lesson.id) }}" class="btn btn-secondary">✏️ Edit</a>
                <form method="POST" action="{{ url_for('delete', id=lesson.id) }}" style="display:inline;">
                    <button type="submit" class="btn btn-danger" 
                            onclick="return confirm('Delete this lesson?')">
                        🗑️ Delete
                    </button>
                </form>
            </div>
        </div>
    </div>
    {% endfor %}
{% else %}
    <div style="text-align: center; padding: 40px; background: white; border-radius: 8px;">
        <h3>📭 No lessons yet</h3>
        <p>Create your first lesson to get started!</p>
        <a href="{{ url_for('create') }}" class="btn btn-primary">Create Lesson</a>
    </div>
{% endif %}
{% endblock %}
```

### 📄 Create `templates/edit.html`

```html
<!-- templates/edit.html -->
{% extends "base.html" %}

{% block content %}
<h2 style="text-align: center; margin-bottom: 25px;">{{ title }}</h2>

<form method="POST">
    <div class="form-group">
        <label for="title">Lesson Title</label>
        <input type="text" id="title" name="title" 
               value="{{ lesson.title if lesson else '' }}" required>
    </div>

    <div class="form-group">
        <label for="description">Description</label>
        <textarea id="description" name="description" required>{{ lesson.description if lesson else '' }}</textarea>
    </div>

    <div class="form-group">
        <label for="duration">Duration (minutes)</label>
        <input type="number" id="duration" name="duration" min="1" max="600"
               value="{{ lesson.duration if lesson else '' }}" required>
    </div>

    <div style="text-align: center; gap: 12px; display: flex; justify-content: center;">
        <button type="submit" class="btn btn-primary">💾 Save Lesson</button>
        <a href="{{ url_for('index') }}" class="btn btn-secondary">⬅️ Cancel</a>
    </div>
</form>
{% endblock %}
```

---

## 🧪 **Step 5: Run & Test (10 min)**

### ▶️ Start the App
```bash
python app.py
```

### 🔍 Test All Features
1. Open `http://localhost:5000`
2. **Create**: Click "Add New Lesson" → Fill form → Save
3. **Read**: See lessons on homepage
4. **Update**: Click "Edit" → Modify → Save
5. **Delete**: Click "Delete" → Confirm

### 📱 Test Responsiveness
- Resize browser window
- Use **DevTools → Toggle Device Toolbar** (Chrome)
- Try on phone: layout stacks vertically on small screens

---

## 🏁 **Key Takeaways**

✅ **Flask CRUD Routes**  
- `GET /` → Read  
- `POST /create` → Create  
- `POST /<id>/edit` → Update  
- `POST /<id>/delete` → Delete  

✅ **SQLite3 Best Practices**  
- Parameterized queries (`?`)  
- Connection management  
- Auto-increment IDs  

✅ **Responsive Design**  
- Mobile-first CSS  
- Flexbox + Media Queries  
- Touch-friendly buttons  

✅ **User Experience**  
- Flash messages (success/error)  
- Confirmation for delete  
- Form validation  

---

## 🚀 **Next Steps (Homework Ideas)**

1. Add **search** functionality
2. Include **lesson categories** (e.g., Math, Science)
3. Add **date created** field
4. Deploy to **PythonAnywhere** or **Render**

---

## 💡 **Troubleshooting Tips**

| Issue | Solution |
|------|---------|
| CSS not loading | Check `static/style.css` path and `url_for()` |
| Database locked | Always `conn.close()` after use |
| Form not submitting | Ensure `method="POST"` and all fields have `name` |
| 404 errors | Check route names and URL parameters |


