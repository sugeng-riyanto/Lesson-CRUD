

---

## 🧭 Step-by-Step Implementation Guide

### ✅ Step 1: Choose Your Project
Pick **one** of the five options (e.g., **To-Do List Manager**). Stick with it—don’t mix features from multiple projects unless you’re confident.

> 💡 Tip: **To-Do List** or **Book Library** are the easiest to start with.

---

### ✅ Step 2: Duplicate & Rename Your Base App
1. Copy your working `lesson_crud_app/` folder.
2. Rename it (e.g., `todo_app/` or `book_tracker/`).
3. Delete any lesson-specific data or comments.

---

### ✅ Step 3: Update the Database Schema (`database.py`)
Replace the `lessons` table with your new table.

**Example for To-Do List**:
```python
# database.py
import sqlite3

def init_db():
    conn = sqlite3.connect('app.db')
    c = conn.cursor()
    c.execute('''
        CREATE TABLE IF NOT EXISTS tasks (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            title TEXT NOT NULL,
            description TEXT,
            completed BOOLEAN DEFAULT 0
        )
    ''')
    conn.commit()
    conn.close()
```

> 🔁 **Key**: Match the table name and columns **exactly** as in the assignment.

---

### ✅ Step 4: Update `app.py` Routes
Replace every reference to `lesson` with your new entity (e.g., `task`, `book`).

#### Key Changes:
- Route names: `/lessons` → `/tasks`
- Variable names: `lesson` → `task`
- Database queries: `SELECT * FROM lessons` → `SELECT * FROM tasks`

**Example: Create Route (To-Do)**
```python
@app.route('/add', methods=['POST'])
def add_task():
    title = request.form['title']
    description = request.form.get('description', '')
    completed = 1 if request.form.get('completed') else 0
    conn = sqlite3.connect('app.db')
    c = conn.cursor()
    c.execute('INSERT INTO tasks (title, description, completed) VALUES (?, ?, ?)',
              (title, description, completed))
    conn.commit()
    conn.close()
    flash('Task added!')
    return redirect('/')
```

> ⚠️ **Important**: For boolean fields like `completed`, HTML checkboxes only send data if checked. Use `.get()` safely.

---

### ✅ Step 5: Update Templates
Update `templates/index.html` and `templates/edit.html`.

#### In `index.html`:
- Change headings: “Lessons” → “Tasks”
- Loop over your new items: `{% for task in tasks %}`
- Display relevant fields (e.g., show completion status)

**To-Do Example: Show Completion Visually**
```html
<li class="{% if task.completed %}completed{% endif %}">
  <strong>{{ task.title }}</strong>
  {% if task.description %} — {{ task.description }}{% endif %}
</li>
```

Add CSS:
```css
.completed {
  text-decoration: line-through;
  color: #888;
}
```

#### In `edit.html`:
- Update form fields to match your schema
- For **dropdowns** (e.g., Book status), use `<select>`
- For **checkboxes** (e.g., completed), use:
  ```html
  <input type="checkbox" name="completed" value="1" {% if task.completed %}checked{% endif %}>
  ```

> 💡 **Pro Tip**: Use `value="1"` for checkboxes—SQLite stores booleans as 0/1.

---

### ✅ Step 6: Add Flash Messages & Responsive Design
- Keep the flash message block from your base app in `base.html`:
  ```html
  {% with messages = get_flashed_messages() %}
    {% if messages %}
      <div class="alert">
        {% for msg in messages %}{{ msg }}{% endfor %}
      </div>
    {% endif %}
  {% endwith %}
  ```
- Reuse your existing `style.css`—just tweak colors/icons if desired.
- Test on mobile (use browser dev tools → toggle device toolbar).

---

### ✅ Step 7: Test Thoroughly
1. Run the app: `python app.py`
2. Test all CRUD actions:
   - ✅ Create a new item
   - ✅ View it on the homepage
   - ✅ Edit it (check if form pre-fills correctly)
   - ✅ Delete it
3. Verify:
   - Completed tasks appear strikethrough
   - Dropdowns save correctly
   - No 500 errors on form submission

---

### ✅ Step 8: Write `README.md`
Include:
```markdown
# To-Do List Manager

A Flask app to manage personal tasks with completion status.

## How to Run
1. Install Flask: `pip install flask`
2. Run: `python app.py`
3. Open http://localhost:5000

## Features
- Add, edit, delete tasks
- Mark tasks as complete
- Responsive design
```

> 📸 Optional: Add a screenshot in a `screenshots/` folder and link it.

---

## 🛑 Common Mistakes to Avoid
| Mistake | Fix |
|-------|------|
| Forgetting to update table name in SQL queries | Double-check `INSERT INTO tasks`, not `lessons` |
| Checkbox not saving correctly | Use `completed = 1 if request.form.get('completed') else 0` |
| Jinja2 variable name mismatch | `{% for book in books %}`, not `{% for lesson in lessons %}` |
| Not initializing the DB | Call `init_db()` at the bottom of `app.py` or in a setup script |

---

## 🎉 Final Tip
> **You’re not coding from scratch—you’re adapting**.  
> Every line you change should have a clear purpose:  
> “This was ‘lesson.title’, now it’s ‘task.title’.”

Good luck! You’ve got this. 💪
