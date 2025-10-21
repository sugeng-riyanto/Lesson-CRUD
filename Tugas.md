# 📝 **Flask CRUD Project Assignments**  
**Based on: Lesson Management App with Flask + SQLite3 + Responsive Design**

After completing the core lesson CRUD app, apply your skills to **one** of the following real-world projects. Each builds on the same concepts: **Flask routes, SQLite3, Jinja2 templates, form handling, and responsive CSS**.

---

## 🎯 **Assignment Instructions**

1. **Choose ONE project** from the options below.  
2. Reuse the structure from the lesson app (`app.py`, `database.py`, `templates/`, `static/`).  
3. **Modify only what’s necessary** (e.g., table schema, form fields, labels).  
4. Ensure your app is **fully responsive** and includes **flash messages**.  
5. Submit your code with a `README.md` explaining your project.

---

## 📚 **Project Options**

### 1. 📝 **To-Do List Manager**
**Description**: Create, read, update, and delete personal tasks with completion status.

**Database Table**:
```sql
tasks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    description TEXT,
    completed BOOLEAN DEFAULT 0  -- 0 = not done, 1 = done
)
```

**Features**:
- Mark tasks as complete/incomplete (add a checkbox or toggle button)
- Show completed vs. pending tasks visually (e.g., strikethrough)
- Add due date (optional bonus)

**Hint**:  
> Use a hidden input or checkbox in your form to toggle `completed`. In SQLite, store as `0` or `1`. In Python, convert to `bool()` when displaying.

---

### 2. 📖 **Book Library Tracker**
**Description**: Manage your personal book collection with title, author, and status.

**Database Table**:
```sql
books (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    author TEXT NOT NULL,
    status TEXT NOT NULL  -- e.g., "To Read", "Reading", "Completed"
)
```

**Features**:
- Dropdown for `status` (use `<select>` in form)
- Filter books by status on the homepage
- Add ISBN or cover image URL (optional)

**Hint**:  
> In `edit.html`, replace the duration input with a `<select>` element:
> ```html
> <select name="status">
>   <option value="To Read" {% if lesson.status == 'To Read' %}selected{% endif %}>To Read</option>
>   <!-- add other options -->
> </select>
> ```

---

### 3. 🛒 **Shopping List App**
**Description**: Track grocery or shopping items with quantity and category.

**Database Table**:
```sql
items (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    quantity INTEGER DEFAULT 1,
    category TEXT NOT NULL  -- e.g., "Produce", "Dairy", "Household"
)
```

**Features**:
- Group items by category on the homepage
- Increment/decrement quantity with buttons (optional)
- Add "Purchased" toggle (like To-Do)

**Hint**:  
> Use CSS to visually group items:
> ```html
> {% for category in categories %}
>   <h3>{{ category }}</h3>
>   {% for item in items if item.category == category %}
>     <!-- display item -->
>   {% endfor %}
> {% endfor %}
> ```

---

### 4. 💡 **Idea Journal / Notebook**
**Description**: Capture and organize creative ideas, notes, or quotes.

**Database Table**:
```sql
ideas (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    content TEXT NOT NULL,
    tags TEXT  -- e.g., "work, inspiration" (store as comma-separated string)
)
```

**Features**:
- Display tags as clickable badges
- Add search by title or tags (bonus)
- Include a "favorite" star toggle (optional)

**Hint**:  
> To display tags, split the string in Jinja2:
> ```jinja2
> {% for tag in idea.tags.split(',') if idea.tags %}
>   <span class="tag">{{ tag.strip() }}</span>
> {% endfor %}
> ```

---

### 5. 🎮 **Game Collection Tracker**
**Description**: Track video games or board games you own or want to play.

**Database Table**:
```sql
games (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    platform TEXT NOT NULL,  -- e.g., "PC", "PlayStation", "Board Game"
    status TEXT NOT NULL     -- e.g., "Owned", "Wishlist", "Played"
)
```

**Features**:
- Filter by platform or status
- Add rating (1–5 stars) using radio buttons (bonus)
- Show total count of games

**Hint**:  
> Use radio buttons for rating:
> ```html
> {% for i in range(1, 6) %}
>   <input type="radio" name="rating" value="{{ i }}" {% if game.rating == i %}checked{% endif %}>
>   <label>{{ i }} ★</label>
> {% endfor %}
> ```

---

## 🛠️ **General Tips for All Projects**

✅ **Reuse Code**: Copy your working `lesson_crud_app` folder and rename it.  
✅ **Update Names**: Change "lesson" → your entity (e.g., "task", "book") in all files.  
✅ **Test Early**: Run the app after each change to catch errors quickly.  
✅ **Keep It Simple**: Focus on core CRUD first; add bonuses later.  
✅ **Responsive First**: Use the same CSS framework—just update colors/icons if desired.

---

## 📤 **Submission Checklist**

- [ ] `app.py` with proper CRUD routes  
- [ ] `database.py` with correct table schema  
- [ ] `templates/` with `base.html`, `index.html`, `edit.html`  
- [ ] `static/style.css` (responsive!)  
- [ ] `README.md` with:  
  - Project name  
  - How to run it  
  - Screenshot (optional but encouraged)

