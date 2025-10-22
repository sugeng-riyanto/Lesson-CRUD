


## 🎮 Game Collection Tracker – Implementation Guide

### ✅ Step 1: Set Up the Project Folder
```bash
cp -r lesson_crud_app/ game_tracker/
cd game_tracker/
```

Now adapt everything for games.

---

### ✅ Step 2: Update the Database (`database.py`)

Use the `games` table schema:

```python
# database.py
import sqlite3

def init_db():
    conn = sqlite3.connect('app.db')
    c = conn.cursor()
    c.execute('''
        CREATE TABLE IF NOT EXISTS games (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            platform TEXT NOT NULL,   -- e.g., "PC", "PlayStation", "Board Game"
            status TEXT NOT NULL      -- e.g., "Owned", "Wishlist", "Played"
        )
    ''')
    conn.commit()
    conn.close()
```

> 💡 Common platforms: `"PC"`, `"Nintendo Switch"`, `"PlayStation 5"`, `"Xbox"`, `"Board Game"`, `"Mobile"`  
> Common statuses: `"Owned"`, `"Wishlist"`, `"Played"`, `"Completed"`, `"Abandoned"`

Call `init_db()` in `app.py`.

---

### ✅ Step 3: Update `app.py`

Here’s a clean, fully functional `app.py`:

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
    games = conn.execute('SELECT * FROM games ORDER BY name').fetchall()
    total = conn.execute('SELECT COUNT(*) as count FROM games').fetchone()['count']
    conn.close()

    # Optional: Group by status or platform (we'll do status here)
    statuses = ["Wishlist", "Owned", "Played", "Completed"]
    grouped = {}
    for status in statuses:
        grouped[status] = [g for g in games if g['status'] == status]

    return render_template('index.html', grouped=grouped, total=total)

@app.route('/add', methods=['POST'])
def add_game():
    name = request.form['name'].strip()
    platform = request.form['platform'].strip()
    status = request.form['status']

    if not name or not platform or not status:
        flash('All fields are required!')
        return redirect('/')

    conn = get_db_connection()
    conn.execute('INSERT INTO games (name, platform, status) VALUES (?, ?, ?)',
                 (name, platform, status))
    conn.commit()
    conn.close()
    flash(f'🎮 Added {name}!')
    return redirect('/')

@app.route('/edit/<int:id>', methods=['GET', 'POST'])
def edit_game(id):
    conn = get_db_connection()
    game = conn.execute('SELECT * FROM games WHERE id = ?', (id,)).fetchone()

    if request.method == 'POST':
        name = request.form['name'].strip()
        platform = request.form['platform'].strip()
        status = request.form['status']

        if not name or not platform or not status:
            flash('All fields are required!')
        else:
            conn.execute('UPDATE games SET name = ?, platform = ?, status = ? WHERE id = ?',
                         (name, platform, status, id))
            conn.commit()
            conn.close()
            flash('💾 Game updated!')
            return redirect('/')

    conn.close()
    return render_template('edit.html', game=game)

@app.route('/delete/<int:id>')
def delete_game(id):
    conn = get_db_connection()
    conn.execute('DELETE FROM games WHERE id = ?', (id,))
    conn.commit()
    conn.close()
    flash('🗑️ Game removed!')
    return redirect('/')
```

---

### ✅ Step 4: Update Templates

#### `templates/base.html`
Keep it game-themed and responsive:

```html
<!DOCTYPE html>
<html>
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
  <title>🎮 My Game Collection</title>
</head>
<body>
  <div class="container">
    <h1>🎮 My Game Collection</h1>
    <p class="total">Total games: {{ total or 0 }}</p>

    {% with messages = get_flashed_messages() %}
      {% if messages %}
        <div class="alert">{{ messages[0] }}</div>
      {% endif %}
    {% endwith %}

    {% block content %}{% endblock %}
  </div>
</body>
</html>
```

> 💡 We’ll pass `total` from `index()`—see next template.

---

#### `templates/index.html`
Group games by **status** and show total count:

```html
<!-- Add New Game Form -->
<form method="POST" action="/add">
  <input type="text" name="name" placeholder="Game Name" required>
  
  <input type="text" name="platform" placeholder="Platform (e.g., PC, Switch)" required>
  
  <select name="status" required>
    <option value="">— Status —</option>
    <option value="Wishlist">Wishlist</option>
    <option value="Owned">Owned</option>
    <option value="Played">Played</option>
    <option value="Completed">Completed</option>
    <option value="Abandoned">Abandoned</option>
  </select>
  
  <button type="submit">➕ Add Game</button>
</form>

<hr>

<!-- Display Games Grouped by Status -->
{% for status, games in grouped.items() %}
  {% if games %}
    <h2>🕹️ {{ status }}</h2>
    <ul class="game-list">
      {% for game in games %}
        <li>
          <strong>{{ game.name }}</strong> 
          <em>({{ game.platform }})</em>
          <a href="/edit/{{ game.id }}">✏️ Edit</a>
          <a href="/delete/{{ game.id }}" onclick="return confirm('Remove this game?')">🗑️ Delete</a>
        </li>
      {% endfor %}
    </ul>
  {% endif %}
{% endfor %}

{% if not grouped.values() | selectattr('0') | list %}
  <p>📭 Your collection is empty! Add your first game.</p>
{% endif %}
```

> 🔁 This groups games logically—great for tracking what you want to play vs. what you’ve finished.

---

#### `templates/edit.html`
Edit form with dropdown for status:

```html
<h2>✏️ Edit Game</h2>
<form method="POST">
  <input type="text" name="name" value="{{ game.name }}" placeholder="Game Name" required>
  
  <input type="text" name="platform" value="{{ game.platform }}" placeholder="Platform" required>
  
  <select name="status" required>
    <option value="Wishlist" {% if game.status == 'Wishlist' %}selected{% endif %}>Wishlist</option>
    <option value="Owned" {% if game.status == 'Owned' %}selected{% endif %}>Owned</option>
    <option value="Played" {% if game.status == 'Played' %}selected{% endif %}>Played</option>
    <option value="Completed" {% if game.status == 'Completed' %}selected{% endif %}>Completed</option>
    <option value="Abandoned" {% if game.status == 'Abandoned' %}selected{% endif %}>Abandoned</option>
  </select>
  
  <button type="submit">💾 Save Changes</button>
  <a href="/">← Cancel</a>
</form>
```

---

### ✅ Step 5: Styling (`static/style.css`)

Game-inspired, clean, and responsive:

```css
.container {
  max-width: 750px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Segoe UI', system-ui, sans-serif;
  background: #f8f9fa;
}

.total {
  font-size: 1.1em;
  color: #4a5568;
  margin-bottom: 15px;
}

input, select, button {
  width: 100%;
  padding: 10px;
  margin: 8px 0;
  box-sizing: border-box;
  border: 1px solid #ccc;
  border-radius: 6px;
}

.game-list {
  list-style: none;
  padding: 0;
}

.game-list li {
  padding: 12px;
  margin: 8px 0;
  background: white;
  border-radius: 6px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

.alert {
  padding: 10px;
  background: #e6f4ea;
  color: #137333;
  border: 1px solid #87cb96;
  border-radius: 6px;
  margin-bottom: 16px;
}

button {
  background: #d32f2f; /* Game red */
  color: white;
  border: none;
  cursor: pointer;
  font-weight: bold;
}

button:hover {
  background: #b71c1c;
}

a {
  margin-left: 10px;
  text-decoration: none;
  color: #1976d2;
}

@media (max-width: 600px) {
  .container { padding: 12px; }
  .total { font-size: 1em; }
}
```

---

### ✅ Step 6: Test Your App
Run:
```bash
python app.py
```

Try:
- Add **"The Legend of Zelda: Tears of the Kingdom"**, platform **"Nintendo Switch"**, status **"Completed"**
- Add **"Stardew Valley"**, platform **"PC"**, status **"Wishlist"**
- Edit Stardew Valley to **"Owned"**
- Delete a game

✅ Total count updates, and games appear under the right status section.

---

### ✅ Step 7: Write `README.md`

```markdown
# 🎮 Game Collection Tracker

Track your video games and board games by platform and status.

## Features
- Add, edit, and delete games
- Categorize by platform (PC, Switch, Board Game, etc.)
- Track status: Wishlist, Owned, Played, Completed, Abandoned
- Games grouped by status on homepage
- Total game count displayed
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
3. Visit: http://localhost:5000


