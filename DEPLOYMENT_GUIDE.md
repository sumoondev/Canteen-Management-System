# Railway Deployment Guide

This guide is written specifically for this project in `/home/suhan/CMS`.

## 1. Before You Push To GitHub

Confirm the repository contains these files at the root:

- `manage.py`
- `requirements.txt`
- `Procfile`
- `runtime.txt`
- `data-migration.json`

Confirm these files are not committed:

- `.env`
- local media files
- SQLite database files

Run the final local checks:

```bash
source myenv/bin/activate
python manage.py check
python manage.py test
python manage.py collectstatic --noinput
```

## 2. Push The Code To GitHub

If the repo is not already connected:

```bash
git init
git branch -M main
git add .
git commit -m "Prepare Railway deployment"
git remote add origin https://github.com/<your-username>/<your-repository>.git
git push -u origin main
```

If the repo already exists:

```bash
git add .
git commit -m "Final Railway deployment prep"
git push origin main
```

## 3. Create The Railway Project

1. Open Railway.
2. Click `New Project`.
3. Choose `Deploy from GitHub repo`.
4. Select your Django repository.
5. Railway will create a web service from the repository root.

## 4. Provision PostgreSQL

1. Inside the same Railway project, click `New`.
2. Choose `Database`.
3. Choose `PostgreSQL`.
4. Wait until Railway creates the database service.
5. Open the PostgreSQL service and confirm it exposes `DATABASE_URL`.

## 5. Add The Persistent Volume

Your uploaded menu item images must survive redeployments, so attach a Railway Volume.

1. Open the Django web service.
2. Open `Settings`.
3. Open `Volumes`.
4. Click `Add Volume`.
5. Set the mount path exactly to:

```text
/app/media
```

1. Save the volume configuration.
1. Redeploy after attaching the volume.

## 6. Add Railway Environment Variables

Open the Django web service and add these variables.

```env
DJANGO_ENV=production
DJANGO_SECRET_KEY=<paste-a-new-random-secret>
DJANGO_ALLOWED_HOSTS=<your-service>.up.railway.app,.railway.app
DJANGO_CSRF_TRUSTED_ORIGINS=https://<your-service>.up.railway.app
DATABASE_URL=${{Postgres.DATABASE_URL}}
DB_CONN_MAX_AGE=600
DB_SSL_REQUIRE=True
DJANGO_MEDIA_URL=/media/
DJANGO_MEDIA_ROOT=/app/media
DJANGO_SECURE_SSL_REDIRECT=True
DJANGO_SECURE_HSTS_SECONDS=31536000
DJANGO_SECURE_HSTS_INCLUDE_SUBDOMAINS=True
DJANGO_SECURE_HSTS_PRELOAD=True
```

Optional but useful:

```env
RAILWAY_PUBLIC_DOMAIN=<your-service>.up.railway.app
```

### Secret Key Setup

Generate the Django secret locally:

```bash
source myenv/bin/activate
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

Copy that output into Railway as `DJANGO_SECRET_KEY`.

Important:

- Keep the real secret key out of Git
- Do not reuse your local development key in production
- Do not store production secrets inside `.env.example`

## 7. What Railway Will Run Automatically

Railway uses these project files:

`requirements.txt`

```txt
Django==5.2.11
Pillow==12.1.0
dj-database-url==2.3.0
gunicorn==23.0.0
psycopg2-binary==2.9.10
python-dotenv==1.0.1
whitenoise==6.9.0
```

`Procfile`

```Procfile
web: python manage.py collectstatic --noinput && gunicorn canteen_management.wsgi:application --bind 0.0.0.0:$PORT --log-file -
```

`runtime.txt`

```txt
python-3.12.3
```

## 8. First Deployment Check

After the first Railway deploy, open the deployment logs and confirm:

- dependencies installed successfully
- `collectstatic` completed successfully
- Gunicorn started without import errors
- there is no `ImproperlyConfigured` error for `DATABASE_URL` or `DJANGO_SECRET_KEY`

## 9. Run Migrations And Seed The Live Database

After the app has deployed, run migrations and load the fixture inside the Railway web service environment.

### Option A: Railway Dashboard Shell

1. Open the web service.
2. Open the latest deployment.
3. Open the service shell.
4. Run:

```bash
python manage.py migrate
python manage.py loaddata data-migration.json
python manage.py check --deploy
```

### Option B: Railway CLI

Install Railway CLI, log in, then run:

```bash
railway login
railway link
railway shell
python manage.py migrate
python manage.py loaddata data-migration.json
python manage.py check --deploy
exit
```

## 10. Manual Media Warning

This is important for your viva and for production correctness.

- Your local `media/` data is ignored by Git
- `public/media/` is also ignored
- `data-migration.json` does not embed the actual binary image files
- Railway starts with a fresh empty volume

That means your 4 existing menu item images will not automatically exist on Railway even if the database rows are seeded correctly.

After deployment, you must manually re-upload those 4 images using the live admin inventory edit form so the files are saved into the Railway volume mounted at `/app/media`.

If you skip this step:

- the database rows may still reference image paths
- but the actual image files will not exist on the Railway filesystem
- so image URLs may return missing-file responses

## 11. Post-Deploy Verification Checklist

Verify all of the following in production:

1. The home page loads on the Railway public domain.
2. Login works for admin and normal users.
3. Menu items appear from PostgreSQL data.
4. Admin inventory, analytics, and orders pages load without route errors.
5. AJAX refreshes continue working silently.
6. You can upload a new item image.
7. Uploaded images still exist after a redeploy.
8. Static assets load correctly.
9. `receipt/<order_id>/` works for paid orders.

## 12. Common Problems And Fixes

### App crashes on boot

Check:

- `DJANGO_ENV=production`
- `DJANGO_SECRET_KEY` is set
- `DATABASE_URL` is mapped from the PostgreSQL service
- `DJANGO_ALLOWED_HOSTS` matches the Railway hostname

### CSRF failure on forms

Check:

- `DJANGO_CSRF_TRUSTED_ORIGINS` contains the full `https://...` origin
- you are using the final Railway public URL, not an old preview URL

### Uploaded images disappear after redeploy

Check:

- the volume is mounted at `/app/media`
- `DJANGO_MEDIA_ROOT=/app/media`
- the images were uploaded after the volume was attached

### Static files return 404

Check:

- `whitenoise` is installed
- `collectstatic` succeeded in the logs
- there is no manifest generation error during startup
