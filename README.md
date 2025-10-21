
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
# Import the sqlite3 module to interact with SQLite databases
import sqlite3

# Import 'closing' to ensure resources like database connections are properly closed
from contextlib import closing

def init_db():
    """Create the 'lessons' table if it doesn't already exist."""
    # Open a connection to 'lessons.db' and ensure it closes automatically after use
    with closing(sqlite3.connect('lessons.db')) as conn:
        # Create a cursor to execute SQL commands, and ensure it closes after use
        with closing(conn.cursor()) as cursor:
            # Execute SQL to create the 'lessons' table if it doesn't exist
            cursor.execute('''
                CREATE TABLE IF NOT EXISTS lessons (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,  -- Unique ID, auto-increments
                    title TEXT NOT NULL,                   -- Lesson title (required)
                    description TEXT NOT NULL,             -- Lesson description (required)
                    duration INTEGER NOT NULL              -- Duration in minutes (required)
                )
            ''')
            # Save (commit) the table creation to the database
            conn.commit()

def get_db_connection():
    """Open a new database connection that returns rows as dictionary-like objects."""
    # Connect to the SQLite database file 'lessons.db'
    conn = sqlite3.connect('lessons.db')
    # Set row_factory to sqlite3.Row so we can access columns by name (e.g., row['title'])
    conn.row_factory = sqlite3.Row
    # Return the connection object for use in other functions
    return conn

# Automatically initialize the database when this file is imported (e.g., by app.py)
init_db()
```

> ✅ **Why this works**:  
> - `init_db()` runs once to create the table.  
> - `get_db_connection()` safely connects and enables named column access.

---

## ⚙️ **Step 3: Flask App – `app.py` (20 min)**

Create `app.py`:

```python
# Import core Flask components
from flask import Flask, render_template, request, redirect, url_for, flash

# Import our custom database helper function
from database import get_db_connection

# Create a Flask application instance
app = Flask(__name__)

# Set a secret key for secure session handling (required for flash messages)
app.secret_key = 'super_secret_key_123'

# ===== ROUTE: Home page (READ all lessons) =====
@app.route('/')
def index():
    # Open a connection to the database
    conn = get_db_connection()
    # Fetch all lessons, sorted by ID in descending order (newest first)
    lessons = conn.execute('SELECT * FROM lessons ORDER BY id DESC').fetchall()
    # Close the database connection to free resources
    conn.close()
    # Render the 'index.html' template and pass the lessons data to it
    return render_template('index.html', lessons=lessons)

# ===== ROUTE: Create a new lesson (handles GET and POST) =====
@app.route('/create', methods=['GET', 'POST'])
def create():
    # If the form was submitted (POST request)
    if request.method == 'POST':
        # Get form data from the request
        title = request.form['title']
        description = request.form['description']
        duration = request.form['duration']
        
        # Validate that all fields are filled
        if not title or not description or not duration:
            # Show an error flash message
            flash('All fields are required!', 'error')
        else:
            # Connect to the database
            conn = get_db_connection()
            # Insert new lesson using a parameterized query (prevents SQL injection)
            conn.execute(
                'INSERT INTO lessons (title, description, duration) VALUES (?, ?, ?)',
                (title, description, int(duration))  # Convert duration to integer
            )
            # Save the changes to the database
            conn.commit()
            # Close the connection
            conn.close()
            # Show success message
            flash('Lesson created successfully!', 'success')
            # Redirect to the home page
            return redirect(url_for('index'))
    
    # For GET requests (or if validation failed), show the empty form
    return render_template('edit.html', lesson=None, title="Create New Lesson")

# ===== ROUTE: Edit an existing lesson =====
@app.route('/<int:id>/edit', methods=['GET', 'POST'])
def edit(id):
    # Connect to the database
    conn = get_db_connection()
    # Fetch the lesson with the given ID
    lesson = conn.execute('SELECT * FROM lessons WHERE id = ?', (id,)).fetchone()
    # Close the connection
    conn.close()
    
    # If no lesson was found with that ID
    if lesson is None:
        # Show error message
        flash('Lesson not found!', 'error')
        # Redirect to home
        return redirect(url_for('index'))
    
    # If the form was submitted (POST)
    if request.method == 'POST':
        # Get updated form data
        title = request.form['title']
        description = request.form['description']
        duration = request.form['duration']
        
        # Validate inputs
        if not title or not description or not duration:
            flash('All fields are required!', 'error')
        else:
            # Reconnect to database
            conn = get_db_connection()
            # Update the lesson with new values
            conn.execute(
                'UPDATE lessons SET title = ?, description = ?, duration = ? WHERE id = ?',
                (title, description, int(duration), id)
            )
            # Save changes
            conn.commit()
            # Close connection
            conn.close()
            # Show success message
            flash('Lesson updated successfully!', 'success')
            # Redirect to home
            return redirect(url_for('index'))
    
    # For GET requests, show the form pre-filled with current lesson data
    return render_template('edit.html', lesson=lesson, title="Edit Lesson")

# ===== ROUTE: Delete a lesson (POST only for safety) =====
@app.route('/<int:id>/delete', methods=['POST'])
def delete(id):
    # Connect to database
    conn = get_db_connection()
    # Try to find the lesson
    lesson = conn.execute('SELECT * FROM lessons WHERE id = ?', (id,)).fetchone()
    
    # If lesson doesn't exist
    if lesson is None:
        conn.close()
        flash('Lesson not found!', 'error')
        return redirect(url_for('index'))
    
    # Delete the lesson from the database
    conn.execute('DELETE FROM lessons WHERE id = ?', (id,))
    # Save the deletion
    conn.commit()
    # Close connection
    conn.close()
    # Show success message
    flash('Lesson deleted successfully!', 'success')
    # Go back to home
    return redirect(url_for('index'))

# ===== START THE APP IN DEVELOPMENT MODE =====
if __name__ == '__main__':
    # Run the Flask development server with auto-reload enabled
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
/* Reset default browser margins and padding; include border in element width */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

/* Base body styling: clean, readable font and light background */
body {
    font-family: 'Segoe UI', system-ui, sans-serif; /* Modern, readable font stack */
    line-height: 1.6;                               /* Comfortable line spacing */
    color: #333;                                    /* Dark gray text for readability */
    background: #f8f9fa;                            /* Light gray background */
}

/* Center content with max width for readability on large screens */
.container {
    width: 90%;               /* Responsive width */
    max-width: 1000px;        /* Prevent overly wide lines on big monitors */
    margin: 0 auto;           /* Center horizontally */
    padding: 20px;            /* Add breathing room */
}

/* Header styling: green banner with white text */
header {
    background: #28a745;      /* Bootstrap-style success green */
    color: white;
    padding: 1rem 0;          /* Vertical padding */
    margin-bottom: 2rem;      /* Space below header */
    border-radius: 8px;       /* Rounded corners */
    box-shadow: 0 2px 10px rgba(0,0,0,0.1); /* Subtle shadow for depth */
}

/* Center and size the main title */
header h1 {
    text-align: center;
    font-size: 2rem;
}

/* Alert messages for user feedback */
.alert {
    padding: 12px 16px;
    margin-bottom: 20px;
    border-radius: 6px;
    font-weight: 500;
}
.alert-success { background: #d4edda; color: #155724; border: 1px solid #c3e6cb; }
.alert-error   { background: #f8d7da; color: #721c24; border: 1px solid #f5c6cb; }

/* Button base styles */
.btn {
    display: inline-block;
    padding: 10px 18px;
    text-decoration: none;    /* Remove underline from links styled as buttons */
    border: none;
    border-radius: 6px;
    cursor: pointer;
    font-size: 16px;
    transition: opacity 0.2s; /* Smooth hover effect */
}
.btn:hover { opacity: 0.9; }

/* Button color variants */
.btn-primary   { background: #28a745; color: white; }
.btn-secondary { background: #6c757d; color: white; }
.btn-danger    { background: #dc3545; color: white; }

/* Form layout */
.form-group {
    margin-bottom: 18px; /* Space between form fields */
}
.form-group label {
    display: block;
    margin-bottom: 6px;
    font-weight: 600; /* Bold labels */
}
.form-group input,
.form-group textarea {
    width: 100%;
    padding: 10px;
    border: 1px solid #ced4da; /* Light gray border */
    border-radius: 6px;
    font-size: 16px; /* Prevents zoom on iOS */
}
.form-group textarea {
    height: 120px;
    resize: vertical; /* Allow vertical resizing only */
}

/* Lesson card styling */
.lesson-card {
    background: white;
    padding: 20px;
    margin-bottom: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.08);
    border-left: 4px solid #28a745; /* Accent color on left */
}
.lesson-card h3 {
    color: #28a745;
    margin-bottom: 10px;
}
.lesson-meta {
    display: flex;
    justify-content: space-between; /* Push items to left and right */
    margin-top: 15px;
    padding-top: 15px;
    border-top: 1px solid #eee; /* Separator line */
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
    gap: 10px; /* Space between buttons */
}

/* Mobile responsiveness */
@media (max-width: 768px) {
    .container { width: 95%; padding: 15px; }
    header h1 { font-size: 1.6rem; }
    .lesson-meta { flex-direction: column; gap: 10px; } /* Stack meta info vertically */
    .action-buttons { flex-direction: column; }        /* Stack buttons vertically */
    .btn { width: 100%; text-align: center; }          /* Full-width buttons on mobile */
}
```

### 📄 Update `templates/base.html`

```html
<!-- Base template: shared layout for all pages -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper scaling on mobile devices -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- Page title (can be overridden by child templates) -->
    <title>{% block title %}Lesson Manager{% endblock %}</title>
    <!-- Link to external CSS file in the 'static' folder -->
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
    <!-- Green header banner -->
    <header>
        <div class="container">
            <h1>📚 Lesson Manager</h1>
        </div>
    </header>

    <div class="container">
        <!-- Display flash messages (e.g., success/error) -->
        {% with messages = get_flashed_messages(with_categories=true) %}
            {% if messages %}
                {% for category, message in messages %}
                    <!-- Apply CSS class based on message type (success/error) -->
                    <div class="alert alert-{{ category }}">{{ message }}</div>
                {% endfor %}
            {% endif %}
        {% endwith %}

        <!-- Placeholder for page-specific content (defined in child templates) -->
        {% block content %}{% endblock %}
    </div>
</body>
</html>
```

### 📄 Create `templates/index.html`

```html
<!-- Home page: lists all lessons -->
{% extends "base.html" %}

{% block content %}
<!-- Button to create a new lesson -->
<div style="text-align: center; margin-bottom: 25px;">
    <a href="{{ url_for('create') }}" class="btn btn-primary">➕ Add New Lesson</a>
</div>

<!-- If there are lessons, display them as cards -->
{% if lessons %}
    {% for lesson in lessons %}
    <div class="lesson-card">
        <h3>{{ lesson.title }}</h3>
        <p>{{ lesson.description }}</p>
        <div class="lesson-meta">
            <!-- Show duration with icon -->
            <span class="lesson-duration">⏱️ {{ lesson.duration }} min</span>
            <!-- Edit and Delete buttons -->
            <div class="action-buttons">
                <a href="{{ url_for('edit', id=lesson.id) }}" class="btn btn-secondary">✏️ Edit</a>
                <!-- Delete uses a form (POST) for safety -->
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
    <!-- Show friendly message when no lessons exist -->
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
<!-- Form for creating or editing a lesson -->
{% extends "base.html" %}

{% block content %}
<!-- Page title (either "Create New Lesson" or "Edit Lesson") -->
<h2 style="text-align: center; margin-bottom: 25px;">{{ title }}</h2>

<!-- Form submits via POST to either /create or /<id>/edit -->
<form method="POST">
    <!-- Title input field -->
    <div class="form-group">
        <label for="title">Lesson Title</label>
        <!-- Pre-fill value if editing; empty if creating -->
        <input type="text" id="title" name="title" 
               value="{{ lesson.title if lesson else '' }}" required>
    </div>

    <!-- Description textarea -->
    <div class="form-group">
        <label for="description">Description</label>
        <textarea id="description" name="description" required>
{{ lesson.description if lesson else '' }}</textarea>
    </div>

    <!-- Duration number input (1–600 minutes) -->
    <div class="form-group">
        <label for="duration">Duration (minutes)</label>
        <input type="number" id="duration" name="duration" min="1" max="600"
               value="{{ lesson.duration if lesson else '' }}" required>
    </div>

    <!-- Submit and Cancel buttons -->
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


