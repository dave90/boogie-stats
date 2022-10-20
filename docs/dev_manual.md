## Environment Setup
You will need Python 3.9+ in order to run BoogieStats because there's already some syntax in the codebase that's not
available in older versions.
I recommend using a [virtualenv](https://virtualenv.pypa.io/), Python's built-in `venv` or other means of separating
Python environments when working with BoogieStats.
You can use the snippets below to create and enter the environment.

You will need to run one of these commands only once:
```
$ virtualenv path/to/new/venv  # for virtualenv
$ python -m venv path/to/new/venv  # for venv
```

Run this command every time you want to enter the environment:
```
$ . path/to/new/venv/bin/activate
```

Next step is to install all the requirements and the application itself (in the editable mode).
You can do it by running:
```
$ pip install -e .[dev]
```
while you are in the root of the repository.

Once you have the package and requirements installed, create a development settings file with the following contents:
```
from boogiestats.boogiestats.settings import *  # noqa

DEBUG = True

# if you want to see song info in the UI, uncomment and properly set BS_CHART_DB_PATH
# BS_CHART_DB_PATH = "/path/to/stepmania-chart-db/db"  # see https://github.com/florczakraf/stepmania-chart-db

# you might need to uncomment and modify ALLOWED_HOSTS if you don't run your development server on localhost
# ALLOWED_HOSTS = ["localhost", "127.0.0.1", "any.extra.host.you.need"]
```
Adjust it as needed.
I usually put it next to the normal settings just for the ease of use (and I will assume that for the next commands):
`boogiestats/boogiestats/settings_dev.py`.

You are now ready to create the database:
```
$ DJANGO_SETTINGS_MODULE=boogiestats.boogiestats.settings_dev django-admin migrate
```
It will be created in `boogiestats/db.sqlite3` unless you modify `DATABASES` in your settings.
Migration should also be used when you already have a database but the application's code has changed.
You can safely run it on every repository update because the database tracks the migrations that have already been applied.

## Running the Application
You are now ready to run django's dev server. It has a built-in option to reload on changes, which works great with
packages that are installed in editable mode. Please notice that the dev server is not supposed to be used in a production
environment. In order to start dev server, run:
```
# start a server that listens on http://localhost:8000
$ DJANGO_SETTINGS_MODULE=boogiestats.boogiestats.settings_dev django-admin runserver 8000

# start a server that listens on all interfaces on port 8000; useful for access over the network, for example from a phone
$ DJANGO_SETTINGS_MODULE=boogiestats.boogiestats.settings_dev django-admin runserver 0.0.0.0:8000  
```

## Tests
BoogieStats uses [pytest](https://docs.pytest.org/) as a test framework. In order to run all tests, run:
```
$ pytest
```
If you need more information about pytest, just consult their docs.

## Coding Standards
BoogieStats enforce multiple standards when it comes to code style and quality. As long as the following commands exit
with 0 status, it's OK. If you need more info, consult the respective tool's docs.
```
$ black .  # reformats python code
$ flake8  # lints the python code, currently disabled because of issue #28
$ djlint --profile django --reformat .  # reformats django templates
$ djlint --profile django --check --lint .  # lints django templates (sometimes you have to do manual fixes because the one above can't fix everything)
$ bandit --configfile bandit.yml -r boogiestats/  # lints the python code for security issues
```

## Modifying Models
Until now, all [models](https://docs.djangoproject.com/en/stable/topics/db/models/) changes could be achieved using normal
django migrations. Unfortunately, some of them, such as the one that introduced collecting score judgment details, could
not be backwards compatible. As a result of that, we have to live with the legacy™.

In order to generate a migration after a change in model, run:
```
$ DJANGO_SETTINGS_MODULE=boogiestats.boogiestats.settings_dev django-admin makemigrations
```

## Useful Commands Summary
```
$ pip install -e .[dev]
$ pytest
$ black .
$ flake8
$ djlint --profile django --reformat .
$ djlint --profile django --check --lint .
$ bandit --configfile bandit.yml -r boogiestats/
$ DJANGO_SETTINGS_MODULE=boogiestats.boogiestats.settings_dev django-admin runserver 8000
$ DJANGO_SETTINGS_MODULE=boogiestats.boogiestats.settings_dev django-admin migrate
$ DJANGO_SETTINGS_MODULE=boogiestats.boogiestats.settings_dev django-admin makemigrations
$ DJANGO_SETTINGS_MODULE=prod.settings django-admin collectstatic
$ DJANGO_SETTINGS_MODULE=prod.settings gunicorn --bind localhost:55523 boogiestats.boogiestats.wsgi --log-level DEBUG --access-logfile access.log --error-logfile error.log --threads 2
```