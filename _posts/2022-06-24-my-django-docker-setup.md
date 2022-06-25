# My Django Docker Setup

This is my template

**requirements.txt**
```shell
Django
psycopg2-binary
gunicorn
python-dateutil
whitenoise
```
**scripts/init-django.sh**
```shell
#!/bin/bash
set -e

if [ "$ENV" = "local" ]; then

    echo "Giving postgres some startup time ..."
    sleep 5
    
    PGPASSWORD="$DB_PASSWORD"
    
fi;

>&2 echo "Migrating database1 ..."

python manage.py makemigrations --merge --noinput

>&2 echo "Migrating database2 ..."

python manage.py makemigrations --noinput

>&2 echo "Migrating database3 ..."

python manage.py migrate --noinput

>&2 echo "Creating some data ...."

export PYTHONPATH=/app:/usr/local/lib/python3.9/site-packages/
export DJANGO_SETTINGS_MODULE=core.settings

python scripts/init_db.py


if [ "$ENV" = "local" ]; then
    python manage.py runserver 0.0.0.0:8000

else
    python manage.py collectstatic --noinput
    cd /app
    exec gunicorn core.wsgi:application \
        --name webserver \
        --bind 0.0.0.0:443 \
        --workers 10 \
        --log-level=info \
        --certfile=cert.pem \
        --keyfile=privkey.pem
fi;
```

**scripts/init-db.py**
```python
import django
django.setup()

from django.contrib.auth.models import User

"""
Create some initial data 
"""

users = User.objects.all()

if not users:
    print ("Creating superuser admin/admin...")
    User.objects.create_superuser('admin', 'admin@example.com', 'admin')

    # pre-populate some data
    # ...
else:
    print ("Already data in db. Skipping ...")
```
