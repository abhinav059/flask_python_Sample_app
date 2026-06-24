
```markdown
# Deploying a Flask App with Docker 🐳

A hands-on walkthrough of containerizing an open-source Python Flask application using Docker — including the mistakes I made and how I fixed them.

---

## What This Project Is About

Instead of building a Flask app from scratch, I used the **official tutorial app** that ships with the [Flask repository](https://github.com/pallets/flask) itself. It's a small but complete web application — making it a perfect candidate for learning Docker deployment end-to-end.

I forked the repo here: [github.com/AnkitKumar017/flask](https://github.com/AnkitKumar017/flask)  
The specific app lives at: `examples/tutorial`

---

## Project Structure

flask/
└── examples/
    └── tutorial/        ← worked inside here
        ├── flaskr/
        ├── Dockerfile   ← created this
        └── ...


---

## The Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY . .

RUN pip install --upgrade pip
RUN pip install .

EXPOSE 5000

CMD sh -c "flask --app flaskr init-db && flask --app flaskr run --host=0.0.0.0"
```

**Why each line is there:**

| Instruction | Reason |
|---|---|
| `FROM python:3.11-slim` | Lightweight Python base image — smaller and faster than the full image |
| `WORKDIR /app` | Sets the working directory inside the container |
| `COPY . .` | Copies all app files into the container |
| `RUN pip install --upgrade pip` | Makes sure pip itself is up to date before installing |
| `RUN pip install .` | Installs the Flask app and its dependencies via `pyproject.toml` |
| `EXPOSE 5000` | Documents that the app runs on port 5000 |
| `CMD ...` | Initializes the SQLite database first, then starts the dev server |

> The `init-db` step in the CMD is important — without it, Flask has no database schema and every page returns a 500 error. (Learned this the hard way — more below.)

---

## Steps to Reproduce

**1. Clone the forked repo**
```bash
git clone https://github.com/AnkitKumar017/flask.git
```

**2. Navigate into the tutorial app**
```bash
cd flask/examples/tutorial
```

**3. Build the Docker image**
```bash
docker build -t flask-tutorial .
```

**4. Verify the image was created**
```bash
docker images
```

**5. Run the container**
```bash
docker run -d -p 5000:5000 --name flask-container flask-tutorial
```

**6. Check it's running**
```bash
docker ps
```

**7. View logs (useful for debugging)**
```bash
docker logs flask-container
```

**8. Open in browser**
```
http://localhost:5000
```

---

## Problems I Hit (And How I Fixed Them)

### Problem 1 — Dockerfile in the wrong directory

I initially placed the Dockerfile at the root of the cloned repo (`flask/`), but the app code lives inside `flask/examples/tutorial/`. Docker's build context looks for files relative to where the Dockerfile sits, so it couldn't find anything it needed.

**Fix:** Moved the Dockerfile into `flask/examples/tutorial/` and ran the build command from that directory.

---

### Problem 2 — Internal Server Error on every page

After the container started successfully, visiting `http://localhost:5000` just gave:

```
Internal Server Error
```

Checked the logs with `docker logs flask-container` and saw database-related errors. The issue was that the SQLite schema had never been created — Flask's tutorial app requires running `flask init-db` before the app can do anything useful.

**Fix:** Updated the `CMD` in the Dockerfile to initialize the database before starting the server:

```dockerfile
# Before (broken)
CMD ["flask", "--app", "flaskr", "run", "--host=0.0.0.0"]

# After (working)
CMD sh -c "flask --app flaskr init-db && flask --app flaskr run --host=0.0.0.0"
```

The `&&` ensures the server only starts if the database initialization succeeds.

---

## Result

Successfully containerized and ran a real Flask application using Docker. The app was accessible on `http://localhost:5000` with full functionality — register, login, and create posts all worked.

---

## Key Takeaways

- Always place your Dockerfile in the same directory as your app's root (or set the build context carefully)
- If your app needs a database, make sure the schema is initialized **before** the server starts — do it in the same `CMD` using `&&`
- `docker logs <container>` is your best friend when something silently fails
- `python:3.11-slim` is almost always the right base image for Flask apps — the full image is much larger for no real benefit

---

## Tech Used

- Python 3.11
- Flask (Pallets Project)
- Docker
- SQLite
```

---

Clean, tells the story, explains the *why* behind every decision, and the problems section reads like something you actually went through — not a dry checklist. Paste this directly into your `README.md`.
