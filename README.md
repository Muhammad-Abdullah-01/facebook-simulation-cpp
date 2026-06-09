# 📘 Facebook Social Media Simulation — C++

A console-based **social media platform simulation** built in C++, inspired by Facebook. Loads real data from text files and simulates core social networking features — friends, pages, posts, comments, likes, memories, and timelines. Built entirely with **OOP principles**, **raw pointers**, **dynamic memory management**, and **file I/O** — no STL containers used.

---

## 🖥️ Preview

```
        READING AND ASSOCIATING DATA

u1 liked the page Coca-Cola
u2 added a new friend: Ali Hassan
post01 added to the timeline of Sara Khan
...

SOCIAL MEDIA MENU
1.  View Friend List
2.  View Liked Pages
3.  View Home Page
4.  View Timeline
5.  View Liked Lists
6.  Like a Post
7.  View Liked List
8.  View a Specific Post
9.  View Posts
10. See a Memory
11. Share a Memory
12. View Timeline
13. View Page
    exit (enter -1)

Enter your choice:
```

---

## ⚙️ Features

- [x] Load Users, Pages, Posts, Comments from `.txt` files
- [x] Friend list management
- [x] Page likes
- [x] Post timeline (per user and per page)
- [x] Like a post
- [x] Comment on a post
- [x] View home feed (posts by friends + liked pages)
- [x] View full timeline
- [x] See Memories (posts from same day in past years)
- [x] Share a Memory (re-share an old post with new caption)
- [x] Activity support (feeling, thinking about, celebrating, making)
- [x] Sign up — adds new user to file
- [x] View liked list of a post
- [x] View a specific post or page timeline

---

## 🧠 Object-Oriented Design — Core Focus

This project is fundamentally an **OOP design exercise**. Every concept from object-oriented programming is applied deliberately.

---

### 1. 🔷 Inheritance

```
Object (abstract base)
├── User        → a person with friends, timeline, liked pages
└── Page        → a public page with its own timeline
```

`Object` is the **parent class** for both `User` and `Page`. This allows polymorphic behavior — both can be stored as `Object*` pointers and used interchangeably wherever an author or liker is needed.

```
Post (base)
└── Memory      → a shared memory post referencing an original post
```

`Memory` extends `Post` with a pointer to the original post and overrides `ViewPost()` to display "X Years Ago" formatting.

---

### 2. 🔷 Polymorphism

**Runtime polymorphism** via virtual functions:

| Virtual Function | In `Object` | Overridden In |
|---|---|---|
| `GetID()` | returns 0 | `User`, `Page` |
| `GetName()` | returns 0 | `User` (full name), `Page` (title) |
| `AddToTimeline()` | empty | `User`, `Page` |
| `PrintDetailView()` | empty | `User`, `Page` |
| `ViewPost()` | base display | `Memory` (adds "X Years Ago") |

This means `Post`, `Comment`, and `Facebook` classes work with `Object*` pointers without needing to know whether it's a `User` or `Page` at compile time.

---

### 3. 🔷 Encapsulation

Every class hides its data as `private` and exposes only what's needed through well-defined getters and setters:

- `User` hides `friendList`, `likedPages`, `Timeline`
- `Post` hides `Comments[]`, `LikedBy[]`, `SharedBy`
- `Comment` hides `Text`, `CommentedBy`
- `Facebook` is the **controller class (Singleton-style)** — all data arrays live here and are accessible only through its public interface

---

### 4. 🔷 Association & Composition

Carefully designed object relationships:

| Relationship | Type | Description |
|---|---|---|
| `User` ↔ `User` (friends) | Association | Users point to each other; neither owns the other |
| `User` → `Page` (liked pages) | Association | User holds pointer to page; page lives in Facebook |
| `Post` → `Object` (SharedBy) | Association | Post references who made it; doesn't own them |
| `Post` → `Object[]` (LikedBy) | Association | Post references likers; doesn't delete them |
| `Post` owns `Comment[]` | Composition | Post creates and deletes its comments |
| `Post` owns `Activity` | Composition | Post creates and deletes its activity |
| `Memory` → `Post` (ActualPost) | Association | Memory references original post; doesn't delete it |
| `Facebook` owns everything | Composition | Facebook creates and deletes all Users, Pages, Posts, Comments |

---

### 5. 🔷 Operator Overloading

| Operator | Class | Purpose |
|---|---|---|
| `<<` | `Date` | Print date as `dd/mm/yyyy` |
| `>>` | `Date` | Read date from input stream |
| `==` | `Date` | Compare dates (allows ±1 day tolerance for home feed) |
| `<<` | `Comment` | Print comment as `"Name wrote: text"` |

---

### 6. 🔷 Static Members

Used for shared counters across all instances:

| Static Variable | Class | Purpose |
|---|---|---|
| `totalUsers` | `User` | Tracks how many users exist |
| `totalPages` | `Page` | Tracks how many pages exist |
| `totalPosts` | `Post` | Tracks how many posts exist (also used for ID generation) |
| `totalComments` | `Comment` | Tracks total comments (used for ID generation) |
| `friendCounter` | `User` | Helper for associating friends during file load |
| `likeCounter` | `Post` | Helper for associating likes during file load |
| `CurrentDate` | `Date` | Global current date shared across the system |

---

### 7. 🔷 Dynamic Memory Management

No STL — all arrays are manually managed with `new` and `delete`:

- All timelines are `Post**` — arrays of post pointers, max size 10
- Friend lists are `User**`, liked pages are `Page**`
- Triple pointer `char***` used in `LoadUsers()` and `LoadPosts()` to temporarily hold IDs read from file before association
- Every class has a destructor that explicitly frees its own memory
- Associated objects are **not deleted** by the class that holds the pointer — only the owning class deletes

---

### 8. 🔷 File I/O

Data is loaded from 4 text files at startup:

| File | Contains |
|---|---|
| `Pages.txt` | Page IDs and titles |
| `Users.txt` | User IDs, names, friend IDs, liked page IDs |
| `Posts.txt` | Post IDs, dates, text, activity, shared-by, liked-by |
| `Comments.txt` | Comment IDs, post ID, commenter ID, text |

Custom `>>` operator on `Date` and `getline()` used for multi-word text parsing. A two-pass loading strategy is used — first read all entities, then associate them (to avoid dangling pointer issues).

---

## 🏗️ Class Structure

```
Facebook.cpp
│
├── class Helper         → Static utilities: string copy, concatenate, int-to-char
├── class Date           → Day/month/year with overloaded >>, <<, ==
├── class Object         → Abstract base: ID, virtual GetID/GetName/AddToTimeline
│
├── class Activity       → Type + value for post activities (feeling, celebrating...)
├── class Comment        → ID, text, pointer to commenter (Object*)
│
├── class Post           → ID, date, text, activity, SharedBy, Comments[], LikedBy[]
│   └── class Memory     → Inherits Post; adds reference to original post
│
├── class User           → Inherits Object; friends, liked pages, timeline
├── class Page           → Inherits Object; title, timeline
│
└── class Facebook       → Controller; owns all arrays; loads data; search functions
```

---

## ⚙️ How to Build & Run

### Prerequisites
- g++ compiler (MinGW on Windows, GCC on Linux/Mac)
- 4 data text files in the same folder: `Pages.txt`, `Users.txt`, `Posts.txt`, `Comments.txt`

### Windows (VS Code)

```bash
g++ Facebook.cpp -o Facebook
./Facebook.exe
```

### Linux / macOS

```bash
g++ Facebook.cpp -o Facebook
./Facebook
```

---

## 📁 Required File Format

### Pages.txt
```
3
p1
Coca-Cola
p2
National Geographic
```

### Users.txt
```
3
u1 Ali Hassan u2 u3 -1 p1 p2 -1
u2 Sara Khan u1 -1 p1 -1
```

### Posts.txt
```
5
1 post01 15 11 2023
Hello World!
u1 u2 -1
```

### Comments.txt
```
2
c01 post01 u2
Great post!
```

---

## 🕹️ Menu Options

| Option | Action |
|---|---|
| 1 | View your friend list |
| 2 | View pages you liked |
| 3 | View home feed (friends + liked pages posts) |
| 4 | View your timeline |
| 5 | View liked list of a post |
| 6 | Like a post |
| 7 | View liked list |
| 8 | Comment on a post |
| 9 | View a specific post |
| 10 | See memories (same day, past years) |
| 11 | Share a memory with new caption |
| 12 | View full timeline |
| 13 | View a page's timeline |
| -1 | Exit |

---

## 👨‍💻 Built With

- **Language:** C++
- **Compiler:** g++ (MinGW / GCC)
- **IDE:** Visual Studio Code
- **Libraries:** `iostream`, `fstream`, `cstring`, `cstdlib` only

---

## 📄 License

This project is open source and free to use for educational purposes.
