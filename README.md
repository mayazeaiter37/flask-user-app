# Lab 5: Postman and APIs (Flask user app)

Maya Zeaiter, ID 202401204
Software Tools Lab, Fall 2026-2027

A Flask REST API that adds, reads, updates and deletes users in a SQLite database, tested with a Postman collection.

## Endpoints

| # | Endpoint | Method | Description |
|---|----------|--------|-------------|
| 1 | `http://localhost:5000/api/users` | GET | Get the list of all users |
| 2 | `http://localhost:5000/api/users/<user_id>` | GET | Get one user by id |
| 3 | `http://localhost:5000/api/users/add` | POST | Create a new user |
| 4 | `http://localhost:5000/api/users/update` | PUT | Update a user |
| 5 | `http://localhost:5000/api/users/delete/<user_id>` | DELETE | Delete a user |

## Project files

```
flask-user-app/
  app.py                     final app (database functions + REST endpoints)
  requirements.txt           flask, flask-cors
  .gitignore                 venv, __pycache__, database.db
  postman/
    Flask_user_app.postman_collection.json    graded collection
    Flask_user_app.postman_environment.json   environment with base_url
```

## Step by step

### 1. Project setup (VS Code terminal)

```bash
mkdir flask-user-app && cd flask-user-app
python -m venv venv
# Windows:      venv\Scripts\activate
# macOS/Linux:  source venv/bin/activate
pip install flask flask-cors
pip freeze > requirements.txt     # or use the requirements.txt provided
```

Note: `sqlite3` is part of the Python standard library, so it needs no install. The handout's `pip install db-sqlite3` is harmless but not needed. `flask-cors` is needed because the REST code imports `flask_cors`.

### 2. Database code, first commit on main

Create a repo on GitHub named `flask-user-app` (empty, no README). Then:

```bash
git init
git branch -M main
# copy app_step1_db_only.py into the project and rename it app.py
python app.py                     # prints "User table created successfully", database.db appears
git add app.py requirements.txt .gitignore
git commit -m "Add SQLite database layer for users"
git remote add origin https://github.com/<your-username>/flask-user-app.git
git push -u origin main
```

### 3. REST API on a new branch

```bash
git checkout -b rest-api
# replace app.py with the final app.py from this folder
python app.py                     # server runs on http://localhost:5000
```

Test the GET endpoints in the browser:
* `http://localhost:5000/api/users` returns `[]` on an empty database
* after adding a user (Postman, step 5) `http://localhost:5000/api/users/1` returns that user

```bash
git add app.py
git commit -m "Add Flask REST API endpoints for user CRUD"
git push -u origin rest-api
```

### 4. Merge into main

```bash
git checkout main
git merge rest-api
git push origin main
```

### 5. Postman graded exercise

Option A, import (fastest):
1. Postman > Import > select both files in `postman/`.
2. Top right environment dropdown > select **Flask user app - local**.
3. With `python app.py` running, open each request and click Send, in order: Add user, Get all users, Get user by ID, Update user, Delete user. Or right click the collection > Run collection.

Option B, build it by hand (what the lab walks you through):
1. Collections > **+** > name it `Flask user app`.
2. Environments > **+** > name `Flask user app - local` > variable `base_url` = `http://localhost:5000` > Save, then select it in the dropdown.
3. Add five requests using `{{base_url}}` in every URL, for example `{{base_url}}/api/users/add`.
4. For POST and PUT: Body > raw > JSON, and paste the body below.
5. Send each request, then in the response pane click **Save Response > Save as example**.

Add user body (POST):
```json
{
    "name": "John Doe",
    "email": "jondoe@gamil.com",
    "phone": "067765434567",
    "address": "John Doe Street, Innsbruck",
    "country": "Austria"
}
```

Update user body (PUT, `user_id` is required):
```json
{
    "user_id": 1,
    "name": "John Doe",
    "email": "john.doe@gmail.com",
    "phone": "067765434567",
    "address": "Maria-Theresien-Strasse 1, Innsbruck",
    "country": "Austria"
}
```

How the provided collection covers each graded item:
* **Collection named "Flask user app"**: yes.
* **Requests for every endpoint**: Add user, Get all users, Get user by ID, Update user, Delete user.
* **Example for each request**: each request has a saved "200 OK" example taken from a real run of this app.
* **URL saved as an environment variable**: `base_url` in the environment, used as `{{base_url}}` in all five requests.
* Extra: the Add user test script saves the new id into `user_id`, so Get by ID, Update and Delete always hit the user just created. Each request also has simple tests (status 200 plus one content check).

If you built the collection by hand, export it (collection > ... > Export, v2.1) into `postman/`. Then commit it:

```bash
git add postman README.md
git commit -m "Add Postman collection and environment"
git push origin main
```

## Fixes made to the handout code

1. `insert_user`: `conn().rollback()` changed to `conn.rollback()` (conn is an object, calling it raises an error).
2. `get_users` and `get_user_by_id`: added `finally: conn.close()` so connections are not left open.
3. `get_user_by_id`: fixed the `except` indentation from the handout.
4. `__main__`: calls `create_db_table()` before `app.run()`, so the table exists when the API starts. On later runs it prints "User table creation failed - Maybe table already exists", which is expected.

## Troubleshooting

* **macOS, port 5000 already in use or Postman returns 403**: macOS AirPlay Receiver uses port 5000. Turn it off (System Settings > General > AirDrop & Handoff > AirPlay Receiver) or run on another port with `app.run(port=5001)` and set `base_url` to `http://localhost:5001`.
* **Get user by ID returns `{}`**: no user with that id exists. Run Add user first.
* **Update returns `{}`**: the body is missing `user_id` or a field.
