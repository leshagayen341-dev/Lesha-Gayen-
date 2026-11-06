function showDetails(button) {
  document.getElementById("modal-title").textContent = button.dataset.title;
  document.getElementById("modal-author").textContent = "by " + button.dataset.author;
  document.getElementById("modal-genre").textContent = button.dataset.genre;
  document.getElementById("modal-rating").textContent = "★ " + button.dataset.rating + " / 5";
  document.getElementById("modal-year").textContent = button.dataset.year;
  document.getElementById("modal-description").textContent = button.dataset.description;
  const modal = document.getElementById("book-modal");
  modal.classList.add("open");
  modal.setAttribute("aria-hidden", "false");
}

function closeDetails() {
  const modal = document.getElementById("book-modal");
  modal.classList.remove("open");
  modal.setAttribute("aria-hidden", "true");
}

document.getElementById("book-modal").addEventListener("click", function (event) {
  if (event.target === this) closeDetails();
});

document.addEventListener("keydown", function (event) {
  if (event.key === "Escape") closeDetails();
});

import sqlite3

# Connect to database
conn = sqlite3.connect("library.db")
cursor = conn.cursor()

# Create table
cursor.execute("""
CREATE TABLE IF NOT EXISTS books (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    author TEXT NOT NULL,
    category TEXT NOT NULL
)
""")

# Insert books
books = [
    ("Python Programming", "Reema Thareja", "Programming"),
    ("Let Us C", "Yashavant Kanetkar", "Programming"),
    ("Wings of Fire", "APJ Abdul Kalam", "Biography"),
    ("The Alchemist", "Paulo Coelho", "Fiction"),
    ("Rich Dad Poor Dad", "Robert Kiyosaki", "Finance"),
    ("Introduction to AI", "Stuart Russell", "Artificial Intelligence")
]

cursor.executemany("""
INSERT INTO books (title, author, category)
VALUES (?, ?, ?)
""", books)

# Save changes
conn.commit()

print("Database created successfully!")

# Close connection
conn.close()

import sqlite3
import tkinter as tk
from tkinter import messagebox

# Database Connection
conn = sqlite3.connect("library.db")
cursor = conn.cursor()

# Create Books Table
cursor.execute("""
CREATE TABLE IF NOT EXISTS books (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    author TEXT NOT NULL,
    category TEXT NOT NULL
)
""")

conn.commit()


# Function to Add Book
def add_book():
    title = title_entry.get()
    author = author_entry.get()
    category = category_entry.get()

    if title == "" or author == "" or category == "":
        messagebox.showwarning("Warning", "Please fill all fields")
    else:
        cursor.execute(
            "INSERT INTO books (title, author, category) VALUES (?, ?, ?)",
            (title, author, category)
        )
        conn.commit()

        messagebox.showinfo("Success", "Book Added Successfully!")

        title_entry.delete(0, tk.END)
        author_entry.delete(0, tk.END)
        category_entry.delete(0, tk.END)

        view_books()


# Function to View All Books
def view_books():
    book_list.delete(0, tk.END)

    cursor.execute("SELECT * FROM books")
    books = cursor.fetchall()

    for book in books:
        book_list.insert(
            tk.END,
            f"{book[0]}. {book[1]} - {book[2]} ({book[3]})"
        )


# Function to Search Books
def search_book():
    search = search_entry.get()

    book_list.delete(0, tk.END)

    cursor.execute(
        "SELECT * FROM books WHERE title LIKE ? OR author LIKE ?",
        ('%' + search + '%', '%' + search + '%')
    )

    books = cursor.fetchall()

    if books:
        for book in books:
            book_list.insert(
                tk.END,
                f"{book[0]}. {book[1]} - {book[2]} ({book[3]})"
            )
    else:
        messagebox.showinfo("Result", "No Book Found")


# Function to Recommend Books
def recommend_books():
    category = category_entry.get()

    if category == "":
        messagebox.showwarning("Warning", "Enter a category first")
        return

    book_list.delete(0, tk.END)

    cursor.execute(
        "SELECT * FROM books WHERE category LIKE ?",
        ('%' + category + '%',)
    )

    books = cursor.fetchall()

    if books:
        for book in books:
            book_list.insert(
                tk.END,
                f"{book[0]}. {book[1]} - {book[2]} ({book[3]})"
            )
    else:
        messagebox.showinfo(
            "Recommendation",
            "No books available in this category"
        )


# Function to Delete Book
def delete_book():
    selected = book_list.curselection()

    if not selected:
        messagebox.showwarning("Warning", "Please select a book")
        return

    book_text = book_list.get(selected[0])
    book_id = book_text.split(".")[0]

    cursor.execute("DELETE FROM books WHERE id = ?", (book_id,))
    conn.commit()

    messagebox.showinfo("Success", "Book Deleted Successfully!")

    view_books()


# Main Window
root = tk.Tk()
root.title("Library Book Recommendation System")
root.geometry("650x600")
root.configure(bg="#EAF2F8")


# Heading
heading = tk.Label(
    root,
    text="LIBRARY BOOK RECOMMENDATION SYSTEM",
    font=("Arial", 16, "bold"),
    bg="#EAF2F8",
    fg="#154360"
)
heading.pack(pady=15)


# Input Fields
tk.Label(root, text="Book Title", bg="#EAF2F8").pack()
title_entry = tk.Entry(root, width=40)
title_entry.pack(pady=5)

tk.Label(root, text="Author Name", bg="#EAF2F8").pack()
author_entry = tk.Entry(root, width=40)
author_entry.pack(pady=5)

tk.Label(root, text="Category", bg="#EAF2F8").pack()
category_entry = tk.Entry(root, width=40)
category_entry.pack(pady=5)


# Buttons
tk.Button(
    root, text="Add Book", width=20,
    command=add_book, bg="#28B463", fg="white"
).pack(pady=5)

tk.Button(
    root, text="View All Books", width=20,
    command=view_books, bg="#3498DB", fg="white"
).pack(pady=5)


# Search Section
tk.Label(root, text="Search by Title or Author", bg="#EAF2F8").pack(pady=5)

search_entry = tk.Entry(root, width=40)
search_entry.pack(pady=5)

tk.Button(
    root, text="Search Book", width=20,
    command=search_book, bg="#8E44AD", fg="white"
).pack(pady=5)


# Recommendation Button
tk.Button(
    root, text="Recommend Books", width=20,
    command=recommend_books, bg="#E67E22", fg="white"
).pack(pady=5)


# Book List
tk.Label(
    root, text="Available Books",
    font=("Arial", 12, "bold"),
    bg="#EAF2F8"
).pack(pady=5)

book_list = tk.Listbox(root, width=75, height=10)
book_list.pack(pady=5)


# Delete Button
tk.Button(
    root, text="Delete Selected Book", width=20,
    command=delete_book, bg="#E74C3C", fg="white"
).pack(pady=10)


# Load Books
view_books()

# Run Application
root.mainloop()

# Close Database
conn.close()
 :root {
  --ink: #28243a; --muted: #777286; --purple: #7654c8; --purple-dark: #5639a5;
  --lavender: #eee9fb; --cream: #fbf9f5; --line: #eeeaf3;
}
* { box-sizing: border-box; }
html { scroll-behavior: smooth; }
body { margin: 0; background: var(--cream); color: var(--ink); font-family: "DM Sans", Arial, sans-serif; }
a { color: inherit; text-decoration: none; }
.topbar { height: 76px; padding: 0 7%; display: flex; align-items: center; justify-content: space-between; background: #fff; border-bottom: 1px solid var(--line); position: sticky; top: 0; z-index: 10; }
.brand { font-size: 21px; font-weight: 800; letter-spacing: -.8px; white-space: nowrap; }
.brand > span:last-child { color: var(--purple); }
.brand-icon { display: inline-grid; place-items: center; width: 31px; height: 31px; margin-right: 6px; border-radius: 10px; color: white; background: var(--purple); }
nav { display: flex; gap: 30px; color: #6d6879; font-size: 13px; font-weight: 700; }
nav a:hover, nav a.active { color: var(--purple); }
.nav-cta { background: var(--ink); color: white; border-radius: 30px; padding: 12px 18px; font-size: 12px; font-weight: 700; }
.nav-cta span { color: #d9c6ff; margin-left: 8px; }
.hero { min-height: 440px; background: linear-gradient(110deg,#211c35 0%,#35294e 55%,#5a4670 100%); padding: 72px 8%; display: flex; align-items: center; justify-content: space-between; position: relative; overflow: hidden; color: white; }
.hero:after { content: ""; position: absolute; width: 460px; height: 460px; border: 1px solid #ffffff18; border-radius: 50%; right: 7%; top: -130px; }
.hero-glow { position: absolute; width: 360px; height: 360px; border-radius: 50%; background: #a57ae6; filter: blur(120px); opacity: .22; right: 13%; top: 60px; }
.hero-copy { position: relative; z-index: 2; max-width: 600px; }
.pill { display: inline-block; border: 1px solid #ffffff35; background: #ffffff12; padding: 9px 14px; border-radius: 30px; color: #e8ddff; font-size: 10px; font-weight: 800; letter-spacing: 1.6px; }
.spark { color: #e9c77b; margin-right: 7px; }
h1 { font: 800 clamp(42px,6vw,68px)/1.06 "Playfair Display", Georgia, serif; margin: 24px 0 16px; letter-spacing: -1.5px; }
h1 em, h2 em { color: #c8a8ff; font-style: normal; }
.hero-text { color: #d7d0e2; max-width: 460px; font-size: 15px; line-height: 1.8; }
.searchbar { display: flex; align-items: center; max-width: 570px; background: white; padding: 6px; border-radius: 12px; margin-top: 28px; box-shadow: 0 12px 35px #130d2b44; }
.search-icon { color: #8b829c; font-size: 27px; padding: 0 12px; }
.searchbar input { min-width: 0; flex: 1; border: 0; outline: 0; padding: 12px 4px; font: 13px "DM Sans"; color: var(--ink); }
.searchbar button, .recommend-form button { border: 0; border-radius: 9px; background: var(--purple); color: white; padding: 13px 19px; font: 700 12px "DM Sans"; cursor: pointer; }
.searchbar button:hover, .recommend-form button:hover { background: var(--purple-dark); }
.searchbar button span { margin-left: 12px; }
.hero-note { color: #c9bfd8; font-size: 11px; margin-top: 17px; }
.hero-note span { color: #b9e2bd; margin-right: 6px; }
.hero-art { width: 330px; height: 300px; position: relative; z-index: 1; margin-right: 6%; }
.orbit { position: absolute; border: 1px solid #ffffff35; border-radius: 50%; transform: rotate(-25deg); }
.orbit-one { width: 310px; height: 180px; left: 0; top: 55px; }
.orbit-two { width: 250px; height: 250px; left: 30px; top: 18px; }
.book-stack { position: absolute; left: 75px; top: 62px; transform: rotate(-8deg); filter: drop-shadow(0 22px 17px #100b2080); }
.book { width: 185px; height: 68px; border-radius: 5px 12px 12px 5px; margin-bottom: 7px; display: flex; align-items: center; justify-content: space-between; padding: 0 18px; font: 700 13px/1.2 Georgia; letter-spacing: 1px; box-shadow: inset 8px 0 #ffffff25, inset 0 -5px #00000018; }
.book small { font-size: 25px; }
.book-top { background: #c9a8e9; color: #342347; }
.book-mid { background: #d6ad77; color: #4a2c22; margin-left: 15px; }
.book-bottom { background: #7c9c91; color: #172e2a; margin-left: -4px; }
.floating-star { position: absolute; color: #f1d38f; font-size: 30px; }
.star-one { top: 26px; right: 28px; }.star-two { bottom: 34px; left: 12px; font-size: 42px; }
.art-caption { position: absolute; bottom: 2px; right: 4px; color: #e4d9f1; font: italic 14px Georgia; }
.stats-row { display: flex; align-items: center; justify-content: space-around; padding: 25px 7%; background: #fff; border-bottom: 1px solid var(--line); }
.stats-row div { display: flex; flex-direction: column; gap: 2px; }
.stats-row strong { font-size: 20px; color: var(--purple-dark); }
.stats-row span { font-size: 11px; color: var(--muted); }
.stats-row .stats-message { display: block; color: #6e667d; font: italic 15px Georgia; }
.stats-message span { color: #b77a9e; font-size: 19px; margin-left: 6px; }
.content-section { padding: 65px 8% 15px; }
.section-head { display: flex; align-items: end; justify-content: space-between; gap: 20px; margin-bottom: 24px; }
.eyebrow { color: var(--purple); font-size: 10px; font-weight: 800; letter-spacing: 1.8px; }
h2 { font: 700 32px "Playfair Display", Georgia, serif; margin: 7px 0 0; }
h2 em { color: var(--purple); }
.section-head > p, .subtle { color: var(--muted); font-size: 12px; }
.genre-list { display: flex; gap: 10px; flex-wrap: wrap; }
.genre-chip { background: #fff; border: 1px solid var(--line); border-radius: 30px; padding: 11px 17px; font-size: 12px; font-weight: 700; color: #625b70; transition: .2s; }
.genre-chip:hover, .genre-chip.chosen { background: var(--purple); border-color: var(--purple); color: white; box-shadow: 0 6px 15px #7654c833; }
.books-section { padding-top: 55px; }
.sort-form { display: flex; align-items: center; gap: 9px; color: var(--muted); font-size: 11px; }
.sort-form select, .recommend-form select { border: 1px solid #e5deef; background: white; padding: 10px 13px; border-radius: 8px; color: var(--ink); font: 600 11px "DM Sans"; }
.book-grid { display: grid; grid-template-columns: repeat(2,minmax(0,1fr)); gap: 17px; }
.book-card { display: flex; gap: 17px; padding: 15px; background: #fff; border: 1px solid var(--line); border-radius: 14px; box-shadow: 0 5px 18px #33264b08; transition: .2s; }
.book-card:hover { transform: translateY(-3px); box-shadow: 0 12px 28px #33264b12; }
.cover { width: 112px; min-width: 112px; min-height: 157px; border-radius: 7px 11px 11px 7px; display: flex; flex-direction: column; justify-content: space-between; padding: 13px 10px; color: white; box-shadow: inset 7px 0 #ffffff20, 3px 5px 9px #27203a22; text-align: center; }
.cover-pink { background: linear-gradient(145deg,#bd6f8d,#5f345c); }.cover-rose { background: linear-gradient(145deg,#b77b83,#643d55); }.cover-peach { background: linear-gradient(145deg,#d98e67,#7c4c4c); }.cover-blue { background: linear-gradient(145deg,#5985a5,#263a62); }.cover-slate { background: linear-gradient(145deg,#596477,#20263c); }.cover-gold { background: linear-gradient(145deg,#c69b4b,#614a31); }.cover-green { background: linear-gradient(145deg,#67927c,#294d45); }.cover-purple { background: linear-gradient(145deg,#9170c5,#392d68); }.cover-orange { background: linear-gradient(145deg,#e39b55,#884a2e); }
.cover-decoration { font-size: 24px; }.cover-title { font: 700 14px/1.25 Georgia; text-transform: uppercase; }.cover-author { font-size: 8px; opacity: .85; }
.book-details { min-width: 0; flex: 1; display: flex; flex-direction: column; align-items: flex-start; }
.book-meta { display: flex; justify-content: space-between; width: 100%; gap: 8px; font-size: 10px; color: var(--purple); font-weight: 700; }
.rating { color: #b8872c; white-space: nowrap; }
.book-details h3 { font: 700 17px/1.25 "Playfair Display", Georgia; margin: 9px 0 3px; }
.author { color: var(--muted); font-size: 11px; margin: 0; }
.description { color: #777183; font-size: 11px; line-height: 1.55; margin: 9px 0 12px; }
.details-btn { margin-top: auto; border: 0; background: #f0ebfa; color: var(--purple-dark); border-radius: 7px; padding: 9px 12px; font: 700 10px "DM Sans"; cursor: pointer; }
.details-btn:hover { background: var(--purple); color: white; }.details-btn span { margin-left: 9px; }
.result-count { color: #918a9d; font-size: 10px; text-align: right; margin-top: 15px; }
.empty-state { text-align: center; padding: 55px 15px; background: white; border: 1px solid var(--line); border-radius: 15px; }
.empty-state div { font-size: 38px; color: var(--purple); }.empty-state h3 { margin-bottom: 5px; }.empty-state p { color: var(--muted); font-size: 13px; }.empty-state a { color: var(--purple); font-weight: 700; font-size: 12px; }
.recommend-banner { margin: 65px 8% 15px; padding: 34px 38px; border-radius: 18px; background: #eee8fa; display: flex; align-items: center; justify-content: space-between; gap: 25px; }
.recommend-banner h2 { font-size: 27px; }.recommend-banner p { color: #706982; font-size: 12px; }
.recommend-form { min-width: 235px; display: flex; flex-direction: column; gap: 9px; }.recommend-form label { font-size: 11px; font-weight: 700; }.recommend-form button { background: var(--ink); }
.top-section { padding-top: 55px; padding-bottom: 65px; }
.top-list { display: grid; grid-template-columns: repeat(2,minmax(0,1fr)); gap: 10px; }
.top-item { display: flex; align-items: center; gap: 13px; background: white; border: 1px solid var(--line); padding: 13px 16px; border-radius: 10px; }
.top-number { color: #b5a5d3; font: 700 16px Georgia; }.top-emoji { font-size: 22px; }.top-info { display: flex; flex-direction: column; flex: 1; min-width: 0; }.top-info strong { font-size: 12px; }.top-info small { color: var(--muted); font-size: 10px; }.top-rating { color: #b8872c; font-size: 11px; font-weight: 700; }
footer { background: #211c35; color: white; padding: 32px 8%; text-align: center; }footer .brand { display: inline-block; }footer p { color: #d1c9df; font-size: 11px; margin: 12px 0 4px; }footer small { color: #9d94ad; font-size: 10px; }
.modal { display: none; position: fixed; inset: 0; z-index: 30; background: #171326aa; padding: 20px; align-items: center; justify-content: center; }.modal.open { display: flex; }.modal-panel { position: relative; background: white; border-radius: 17px; max-width: 430px; width: 100%; padding: 32px; box-shadow: 0 20px 60px #0003; }.modal-panel h2 { margin: 10px 0 5px; font-size: 27px; }.modal-panel > p:last-of-type { color: #625d6c; font-size: 13px; line-height: 1.7; margin: 22px 0; }.modal-close { position: absolute; right: 16px; top: 10px; border: 0; background: none; font-size: 28px; cursor: pointer; color: #777; }.modal-tags { display: flex; gap: 8px; flex-wrap: wrap; margin-top: 17px; }.modal-tags span { background: #f0ebfa; color: var(--purple-dark); border-radius: 20px; padding: 6px 10px; font-size: 10px; font-weight: 700; }
@media (max-width: 850px) { nav { gap: 14px; }.hero-art { transform: scale(.8); margin-right: -25px; }.hero { padding-left: 6%; padding-right: 6%; }.content-section { padding-left: 6%; padding-right: 6%; } }
@media (max-width: 650px) { .topbar { height: auto; min-height: 65px; padding: 13px 5%; flex-wrap: wrap; gap: 12px; }.brand { font-size: 18px; }nav { order: 3; width: 100%; justify-content: space-between; font-size: 11px; }.nav-cta { font-size: 10px; padding: 9px 12px; }.hero { padding: 55px 6% 48px; min-height: 0; }.hero-art { display: none; }h1 { font-size: 47px; }.searchbar { flex-wrap: wrap; }.searchbar input { width: 60%; }.searchbar button { flex: 1; }.stats-row { flex-wrap: wrap; gap: 20px; justify-content: flex-start; }.stats-row > div { width: 40%; }.stats-row .stats-message { width: 100%; }.section-head { align-items: flex-start; flex-direction: column; }h2 { font-size: 28px; }.book-grid, .top-list { grid-template-columns: 1fr; }.book-card { gap: 13px; }.cover { width: 100px; min-width: 100px; min-height: 145px; }.book-details h3 { font-size: 15px; }.recommend-banner { margin: 45px 5% 10px; padding: 25px; flex-direction: column; align-items: stretch; }.recommend-form { min-width: 0; }.sort-form { align-self: stretch; justify-content: flex-start; } }
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Library Book Recommendations</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600;700;800&family=Playfair+Display:wght@700;800&display=swap" rel="stylesheet">
  <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
  <header class="topbar">
    <a class="brand" href="/"><span class="brand-icon">✦</span> Book<span>Bloom</span></a>
    <nav>
      <a class="active" href="/">Home</a>
      <a href="#discover">Discover</a>
      <a href="#top-books">Top books</a>
      <a href="#about">About</a>
    </nav>
    <a class="nav-cta" href="#discover">Find a book <span>↗</span></a>
  </header>

  <main>
    <section class="hero">
      <div class="hero-glow"></div>
      <div class="hero-copy">
        <div class="pill"><span class="spark">✦</span> YOUR NEXT CHAPTER STARTS HERE</div>
        <h1>Find a book.<br><em>Find your world.</em></h1>
        <p class="hero-text">A little inspiration, a lot of imagination. Discover books that match your mood and curiosity.</p>
        <form class="searchbar" action="/" method="get">
          <span class="search-icon">⌕</span>
          <input name="search" value="{{ search }}" placeholder="Search title, author or genre..." aria-label="Search books">
          <input type="hidden" name="genre" value="{{ selected_genre }}">
          <button type="submit">Search <span>→</span></button>
        </form>
        <div class="hero-note"><span>✓</span> Explore {{ total_books }} handpicked sample books across different genres</div>
      </div>
      <div class="hero-art" aria-label="Decorative stack of books">
        <div class="orbit orbit-one"></div><div class="orbit orbit-two"></div>
        <div class="floating-star star-one">✦</div><div class="floating-star star-two">✧</div>
        <div class="book-stack">
          <div class="book book-top"><span>THE ART OF<br>IMAGINATION</span><small>✦</small></div>
          <div class="book book-mid"><span>WANDER<br> & WONDER</span><small>✧</small></div>
          <div class="book book-bottom"><span>THE QUIET<br>UNIVERSE</span><small>☾</small></div>
        </div>
        <div class="art-caption">Stories worth staying up for.</div>
      </div>
    </section>

    <section class="stats-row">
      <div><strong>{{ total_books }}+</strong><span>Sample books</span></div>
      <div><strong>{{ genres|length - 1 }}</strong><span>Genres to explore</span></div>
      <div><strong>4.8 ★</strong><span>Highest sample rating</span></div>
      <div class="stats-message">Your reading journey, made simpler <span>♡</span></div>
    </section>

    <section class="content-section" id="discover">
      <div class="section-head">
        <div><div class="eyebrow">A WORLD OF STORIES</div><h2>Browse by <em>category</em></h2></div>
        <p>Choose the kind of story you feel like reading today.</p>
      </div>
      <div class="genre-list">
        {% for genre in genres %}
        <a class="genre-chip {% if selected_genre == genre %}chosen{% endif %}" href="/?genre={{ genre|urlencode }}">
          {% if genre == "All" %}✧{% elif genre == "Romance" %}♡{% elif genre == "Thriller" %}⚡{% elif genre == "Mystery" %}⌕{% elif genre == "Fantasy" %}✦{% elif genre == "Science" %}◎{% elif genre == "Biography" %}▤{% elif genre == "Self Help" %}☀{% else %}◈{% endif %}
          {{ genre }}
        </a>
        {% endfor %}
      </div>
    </section>

    <section class="content-section books-section">
      <div class="section-head">
        <div><div class="eyebrow">PICKED FOR CURIOUS MINDS</div><h2>{% if selected_genre != "All" %}{{ selected_genre }}{% else %}Recommended{% endif %} <em>reads</em></h2></div>
        <form class="sort-form" action="/" method="get">
          <input type="hidden" name="search" value="{{ search }}">
          <input type="hidden" name="genre" value="{{ selected_genre }}">
          <label for="sort">Sort by</label>
          <select id="sort" name="sort" onchange="this.form.submit()">
            <option value="recommended" {% if selected_sort == "recommended" %}selected{% endif %}>Recommended</option>
            <option value="rating" {% if selected_sort == "rating" %}selected{% endif %}>Highest rating</option>
            <option value="title" {% if selected_sort == "title" %}selected{% endif %}>Book title</option>
          </select>
        </form>
      </div>

      {% if books %}
      <div class="book-grid">
        {% for book in books %}
        <article class="book-card">
          <div class="cover cover-{{ book.color }}">
            <div class="cover-decoration">{{ book.emoji }}</div>
            <div class="cover-title">{{ book.title }}</div>
            <div class="cover-author">{{ book.author }}</div>
          </div>
          <div class="book-details">
            <div class="book-meta"><span>{{ book.genre }}</span><span class="rating">★ {{ book.rating }}</span></div>
            <h3>{{ book.title }}</h3>
            <p class="author">by {{ book.author }}</p>
            <p class="description">{{ book.description }}</p>
            <button class="details-btn" type="button"
              data-title="{{ book.title|e }}" data-author="{{ book.author|e }}"
              data-genre="{{ book.genre|e }}" data-rating="{{ book.rating }}"
              data-year="{{ book.year }}" data-description="{{ book.description|e }}"
              onclick="showDetails(this)">View details <span>↗</span></button>
          </div>
        </article>
        {% endfor %}
      </div>
      {% else %}
      <div class="empty-state"><div>⌕</div><h3>No books found</h3><p>Try another keyword or choose a different category.</p><a href="/">Clear filters</a></div>
      {% endif %}
      <p class="result-count">Showing {{ books|length }} of {{ total_books }} sample books</p>
    </section>

    <section class="recommend-banner">
      <div><div class="eyebrow">NOT SURE WHAT TO READ?</div><h2>Let us find your <em>next favourite.</em></h2><p>Pick a genre and we'll show you its highest-rated sample books.</p></div>
      <form action="/recommend" method="post" class="recommend-form">
        <label for="recommend-genre">I feel like reading</label>
        <select id="recommend-genre" name="genre">
          {% for genre in genres %}<option value="{{ genre }}">{{ genre }}</option>{% endfor %}
        </select>
        <button type="submit">Recommend for me →</button>
      </form>
    </section>

    <section class="content-section top-section" id="top-books">
      <div class="section-head"><div><div class="eyebrow">READER FAVOURITES</div><h2>Top-rated <em>books</em></h2></div><span class="subtle">Sorted by sample rating</span></div>
      <div class="top-list">
        {% for book in top_books %}
        <div class="top-item"><span class="top-number">0{{ loop.index }}</span><span class="top-emoji">{{ book.emoji }}</span><div class="top-info"><strong>{{ book.title }}</strong><small>{{ book.author }} · {{ book.genre }}</small></div><span class="top-rating">★ {{ book.rating }}</span></div>
        {% endfor %}
      </div>
    </section>
  </main>

  <footer id="about">
    <a class="brand" href="/"><span class="brand-icon">✦</span> Book<span>Bloom</span></a>
    <p>Library Book Recommendations · A Python Flask PBL project</p>
    <small>Book details and ratings are sample data for demonstration.</small>
  </footer>

  <div class="modal" id="book-modal" aria-hidden="true">
    <div class="modal-panel" role="dialog" aria-modal="true" aria-labelledby="modal-title">
      <button class="modal-close" onclick="closeDetails()" aria-label="Close">×</button>
      <div class="eyebrow">BOOK DETAILS</div>
      <h2 id="modal-title"></h2>
      <p id="modal-author" class="author"></p>
      <div class="modal-tags"><span id="modal-genre"></span><span id="modal-rating"></span><span id="modal-year"></span></div>
      <p id="modal-description"></p>
      <button class="details-btn" onclick="closeDetails()">Done</button>
    </div>
  </div>
  <script src="{{ url_for('static', filename='script.js') }}"></script>
</body>
</html>
Library Book Recommendations
A responsive Library Book Recommendation System built with Python Flask, HTML, CSS and JavaScript for a college Project-Based Learning (PBL) assignment.
Features
Search books by title, author or genre
Browse and filter books by category
Sort books by rating or title
Get genre-based recommendations
View book details in a pop-up window
View top-rated sample books
Responsive layout for mobile and desktop
Technologies
Python 3
Flask
HTML5
CSS3
JavaScript
Jinja2
Folder Structure
Library-Book-Recommendations-Pro/
├── app.py
├── requirements.txt
├── README.md
├── templates/
│   └── index.html
└── static/
    ├── style.css
    └── script.js
Run locally
Install Python 3.
Install dependencies: pip install -r requirements.txt
Run: python app.py
Open the local web address shown by Flask (usually http://127.0.0.1:5000).
Important note
The books, descriptions and ratings are sample data for demonstration. Add or edit entries in the BOOKS list in app.py to customize the collection. Ratings are illustrative, not live user ratings.from flask import Flask, render_template, request

app = Flask(__name__)

# Sample dataset for the college project.
BOOKS = [
    {"id": 1, "title": "The Fault in Our Stars", "author": "John Green", "genre": "Romance", "rating": 4.5, "year": 2012, "description": "A moving story about friendship, love and finding meaning in everyday moments.", "emoji": "💗", "color": "pink"},
    {"id": 2, "title": "Pride and Prejudice", "author": "Jane Austen", "genre": "Romance", "rating": 4.7, "year": 1813, "description": "A timeless novel about family, first impressions and unexpected love.", "emoji": "🌷", "color": "rose"},
    {"id": 3, "title": "The Notebook", "author": "Nicholas Sparks", "genre": "Romance", "rating": 4.3, "year": 1996, "description": "A heartfelt story about lasting love and unforgettable memories.", "emoji": "💌", "color": "peach"},
    {"id": 4, "title": "The Silent Patient", "author": "Alex Michaelides", "genre": "Thriller", "rating": 4.4, "year": 2019, "description": "A gripping psychological mystery filled with secrets and surprises.", "emoji": "🕵️", "color": "blue"},
    {"id": 5, "title": "Gone Girl", "author": "Gillian Flynn", "genre": "Thriller", "rating": 4.2, "year": 2012, "description": "A suspenseful story where nothing is quite as it first appears.", "emoji": "🌑", "color": "slate"},
    {"id": 6, "title": "Murder on the Orient Express", "author": "Agatha Christie", "genre": "Mystery", "rating": 4.6, "year": 1934, "description": "Detective Hercule Poirot investigates a puzzling train mystery.", "emoji": "🚂", "color": "gold"},
    {"id": 7, "title": "And Then There Were None", "author": "Agatha Christie", "genre": "Mystery", "rating": 4.7, "year": 1939, "description": "Ten strangers arrive on an island and a clever mystery unfolds.", "emoji": "🔎", "color": "green"},
    {"id": 8, "title": "Harry Potter and the Philosopher's Stone", "author": "J. K. Rowling", "genre": "Fantasy", "rating": 4.8, "year": 1997, "description": "A young boy discovers a magical world and an extraordinary destiny.", "emoji": "🪄", "color": "purple"},
    {"id": 9, "title": "The Hobbit", "author": "J. R. R. Tolkien", "genre": "Fantasy", "rating": 4.7, "year": 1937, "description": "Bilbo Baggins joins an unexpected adventure beyond his home.", "emoji": "🐉", "color": "green"},
    {"id": 10, "title": "A Brief History of Time", "author": "Stephen Hawking", "genre": "Science", "rating": 4.4, "year": 1988, "description": "An accessible introduction to space, time and the universe.", "emoji": "🪐", "color": "blue"},
    {"id": 11, "title": "Cosmos", "author": "Carl Sagan", "genre": "Science", "rating": 4.6, "year": 1980, "description": "Explore the universe and humanity's place within it.", "emoji": "🌌", "color": "purple"},
    {"id": 12, "title": "Wings of Fire", "author": "A. P. J. Abdul Kalam", "genre": "Biography", "rating": 4.7, "year": 1999, "description": "The inspiring life story of India's scientist and former President.", "emoji": "🚀", "color": "orange"},
    {"id": 13, "title": "Atomic Habits", "author": "James Clear", "genre": "Self Help", "rating": 4.6, "year": 2018, "description": "Practical ideas for building good habits one small step at a time.", "emoji": "🌱", "color": "green"},
    {"id": 14, "title": "The Alchemist", "author": "Paulo Coelho", "genre": "Fiction", "rating": 4.5, "year": 1988, "description": "A shepherd travels in search of treasure and discovers his purpose.", "emoji": "✨", "color": "gold"},
    {"id": 15, "title": "To Kill a Mockingbird", "author": "Harper Lee", "genre": "Fiction", "rating": 4.6, "year": 1960, "description": "A powerful story about fairness, empathy and growing up.", "emoji": "📖", "color": "peach"},
    {"id": 16, "title": "The Little Prince", "author": "Antoine de Saint-Exupéry", "genre": "Fiction", "rating": 4.5, "year": 1943, "description": "A thoughtful tale about friendship, kindness and seeing with the heart.", "emoji": "⭐", "color": "blue"},
]

GENRES = ["All", "Romance", "Thriller", "Mystery", "Fantasy", "Science", "Biography", "Self Help", "Fiction"]


@app.route("/")
def home():
    search = request.args.get("search", "").strip()
    genre = request.args.get("genre", "All")
    sort = request.args.get("sort", "recommended")

    results = BOOKS[:]

    if genre != "All":
        results = [book for book in results if book["genre"] == genre]

    if search:
        term = search.casefold()
        results = [
            book for book in results
            if term in book["title"].casefold()
            or term in book["author"].casefold()
            or term in book["genre"].casefold()
        ]

    if sort == "rating":
        results.sort(key=lambda book: book["rating"], reverse=True)
    elif sort == "title":
        results.sort(key=lambda book: book["title"].casefold())

    top_books = sorted(BOOKS, key=lambda book: book["rating"], reverse=True)[:4]

    return render_template(
        "index.html",
        books=results,
        genres=GENRES,
        selected_genre=genre,
        selected_sort=sort,
        search=search,
        top_books=top_books,
        total_books=len(BOOKS)
    )


@app.route("/recommend", methods=["POST"])
def recommend():
    genre = request.form.get("genre", "All")
    if genre not in GENRES:
        genre = "All"
    return home_with_recommendation(genre)


def home_with_recommendation(genre):
    results = [book for book in BOOKS if genre == "All" or book["genre"] == genre]
    results.sort(key=lambda book: book["rating"], reverse=True)
    top_books = sorted(BOOKS, key=lambda book: book["rating"], reverse=True)[:4]
    return render_template(
        "index.html", books=results, genres=GENRES,
        selected_genre=genre, selected_sort="rating", search="",
        top_books=top_books, total_books=len(BOOKS),
        recommendation_message=f"Here are highly rated {genre if genre != 'All' else ''} books for you.".replace("  ", " ")
    )


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)
# Lesha-Gayen-