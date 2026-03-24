 **Full project layout:**

```
otp_auth/
├── otp_auth/
│   ├── settings.py        ← PostgreSQL config via env vars
│   ├── settings_test.py   ← SQLite in-memory (for local test runs)
│   └── urls.py
└── otp/
    ├── migrations/
    │   └── 0001_initial.py
    ├── models.py     ← OTPToken + OTPAuditLog
    ├── services.py   ← issue_token(), verify_token()
    ├── views.py      ← GenerateTokenView, VerifyTokenView
    ├── urls.py
    └── tests.py      ← 15 tests covering all paths
```

---

## Setup commands (your PostgreSQL environment)

```bash
# 1. Create the database
psql -U postgres -c "CREATE USER otp_user WITH PASSWORD 'yourpassword';"
psql -U postgres -c "CREATE DATABASE otp_db OWNER otp_user;"

# 2. Install dependencies
pip install django psycopg2-binary argon2-cffi

# 3. Set environment variables (or put in .env)
export DB_NAME=otp_db
export DB_USER=otp_user
export DB_PASSWORD=yourpassword
export DJANGO_SECRET_KEY=$(python -c "import secrets; print(secrets.token_hex(50))")

# 4. Run migrations
python manage.py migrate

# 5. Start
python manage.py runserver
```

## API usage

```bash
# Generate a password
curl -X POST http://localhost:8000/otp/generate/ \
  -H "Content-Type: application/json" \
  -d '{"user_id": "evano@email.com"}'
# → {"passphrase": "violet-anchor-47", "expires_in_seconds": 120}

# Log in
curl -X POST http://localhost:8000/otp/verify/ \
  -H "Content-Type: application/json" \
  -d '{"user_id": "evano@email.com", "passphrase": "violet-anchor-47"}'
# → {"ok": true, "message": "Login successful."}
```

## What each file handles

`services.py` has all the security logic — `secrets.choice()` for cryptographically secure passphrase generation, Argon2id hashing (never stores the raw phrase), and the atomic `UPDATE ... WHERE used=0` that prevents the race condition where two requests try to use the same token simultaneously.

`views.py` stays thin — it only parses the request and calls the service. All error responses use the same generic message regardless of whether the token was expired, already used, or wrong. That prevents an attacker from learning which condition triggered the failure.

One thing to add before production: wrap the `GenerateTokenView` with your authentication middleware so users can only request their own token. Right now the endpoint trusts whatever `user_id` is in the request body.