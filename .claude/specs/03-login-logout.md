# Spec: Login and Logout

## Overview
This step implements real authentication for Spendly. Currently `GET /login` only renders a static template and `GET /logout` is an unimplemented stub. This feature wires the login form to the `users` table (validating email/password against the hashed password stored by `database/db.py`), establishes a server-side session on success, and implements logout by clearing that session. This is the foundation every later "logged-in" page (profile, expenses) will depend on for knowing who the current user is.

## Depends on
- Step 01 — Database Setup (`users` table, `get_db()`, password hashing via werkzeug) must be complete.

## Routes
- `GET /login` — renders the login form — public (already implemented, unchanged)
- `POST /login` — validates email + password against `users`, starts a session on success, re-renders `login.html` with an error on failure — public
- `GET /logout` — clears the session and redirects to `landing` — logged-in

## Database changes
No database changes. `database/db.py` already has everything needed (`users` table with `email` and `password_hash`). This step only adds a read helper for looking up a user by email — no schema changes.

## Templates
- **Create:** none
- **Modify:** `templates/login.html` — no structural changes required; it already posts to `/login` and already renders `{{ error }}` if present, so the existing markup works as-is with the new POST handler.

## Files to change
- `app.py` — add `methods=["GET", "POST"]` to `/login`, implement credential-check logic, set `app.secret_key`, implement `session` usage, implement `/logout`
- `database/db.py` — add a `get_user_by_email(email)` helper (parameterized query) used by the login route

## Files to create
None.

## New dependencies
No new dependencies. Use `flask.session` (built into Flask) and `werkzeug.security.check_password_hash` (already installed).

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterised queries only
- Passwords hashed with werkzeug — use `check_password_hash` to verify, never compare plaintext
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- `app.secret_key` must be set for `session` to work — read from an environment variable with a hardcoded local-dev fallback, never commit a production secret
- DB logic (the email lookup) belongs in `database/db.py`, not inline in the `/login` route
- On failed login, re-render `login.html` with a generic error (e.g. "Invalid email or password") — never reveal whether the email exists
- `/logout` must actually clear the session (`session.clear()`), not just redirect

## Definition of done
- [ ] Logging in with the seeded demo user (`demo@spendly.com` / `demo123`) succeeds and redirects away from the login page
- [ ] Logging in with a wrong password re-renders `login.html` with an error and does not create a session
- [ ] Logging in with a non-existent email re-renders `login.html` with the same generic error
- [ ] After a successful login, `session` contains the user's id
- [ ] Visiting `/logout` while logged in clears the session and redirects to the landing page
- [ ] `app.py` still has no inline SQL — all queries live in `database/db.py`
- [ ] App starts without errors on port 5001
