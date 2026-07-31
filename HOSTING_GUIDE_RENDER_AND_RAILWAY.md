# Kafubu Block Secondary School Portal Hosting Guide

## Recommended choice

Use a paid Render web service with the persistent disk defined in `render.yaml`.
The school portal stores its SQLite database, uploaded background pictures,
documents, videos and profile pictures under `/var/data`. A free Render web
service must not be used for permanent school records because its local files
are temporary.

## Before uploading to GitHub

Upload the contents inside `kafubu_security_backup_upgrade` directly to the
repository root. At the GitHub root you must see:

- `app.py`
- `requirements.txt`
- `render.yaml`
- `.python-version`
- `wsgi.py`
- `templates`
- `static`

Do not upload the ZIP itself as the application source. Do not upload
`school.db`, `.env`, `.secret_key`, `uploads` or `backups`.

## New Render service using the Blueprint

1. Push all package files to the GitHub `main` branch.
2. In Render, choose **New > Blueprint**.
3. Connect the `Parker173894/Kafubublock` repository.
4. Render reads `render.yaml`.
5. Enter `INITIAL_ADMIN_PASSWORD` when requested. Use at least 12 characters
   with uppercase, lowercase and a number.
6. Enter `RECOVERY_ALLOWED_EMAIL`.
7. Apply the Blueprint and wait for the deployment to become live.
8. Login with username `head` and the value supplied for
   `INITIAL_ADMIN_PASSWORD`.
9. Change the initial password when prompted.

Render generates `FLASK_SECRET_KEY` automatically. Never publish the actual
password or secret in GitHub.

## Existing Render service

If you keep the existing service, configure these settings:

- Build command: `pip install -r requirements.txt`
- Start command:
  `gunicorn --workers 1 --threads 4 --timeout 120 --bind 0.0.0.0:$PORT wsgi:app`
- Health check path: `/health`
- Python version: `3.12.8`

Add a persistent disk:

- Disk name: `kafubu-school-data`
- Mount path: `/var/data`
- Size: `1 GB` initially

Set these environment variables:

- `APP_ENV=production`
- `APP_DEMO_MODE=0`
- `TRUST_PROXY=1`
- `DATA_DIR=/var/data`
- `BACKUP_DIR=/var/data/backups`
- `AUTO_BACKUP_ENABLED=1`
- `FLASK_SECRET_KEY=<random value of at least 32 characters>`
- `INITIAL_ADMIN_PASSWORD=<strong password of at least 12 characters>`
- `RECOVERY_ALLOWED_EMAIL=<official school email>`

After saving, choose **Manual Deploy > Clear build cache & deploy**.

## Railway alternative

Railway is a practical alternative because it supports persistent volumes.

1. Create a Railway project from the GitHub repository.
2. Railway reads `railway.toml`.
3. Add a volume and mount it at `/data`.
4. Set `DATA_DIR=/data` and `BACKUP_DIR=/data/backups`.
5. Add the same security environment variables listed above.
6. Generate a public domain and deploy.

Do not deploy without a volume because the database and uploaded files would
not be permanent.

## Important operational limits

- Keep one Gunicorn worker because SQLite is a single-file database.
- Download backups regularly from the portal Backup and Restore Centre.
- For a larger school or several simultaneous users, migrate from SQLite to
  PostgreSQL and move uploaded files to object storage.
- Never commit passwords, secret keys, pupil data or uploaded private documents
  to GitHub.
