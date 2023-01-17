# General Application Structuring

## Applications
An *Application* is what we in CIX call one of our API systems.
Each *Application* represents a set of functionality powered by a REST API.
Examples of *Applications* include *Membership*, *IAAS*, OTP etc.

## Services
The *Applications* are made up of multiple *Services*.
A *Service* is a set of *Methods* that are used to interact with a single type of model.
Typically, each model in an *Application* will have a *Service* associated with it.
Examples of *Services* include `Membership.user`, `IAAS.project`, etc

## Methods
*Services* are themselves made up of *Methods*.
A *Method* relates to a HTTP method (`GET`, `POST`) and is used to interact with the *Service* model in different ways.
The *Methods* commonly used in CloudCIX SaaS are `list`, `create`, `read`, `update` and `delete`.

When developing *Services*, we use Django's [Class Based Views](https://docs.djangoproject.com/en/2.0/topics/class-based-views/).
We create two classes, a `Collection` class and a `Resource` class (more details in the [views](https://github.com/CloudCIX/developers_manual/blob/main/style_guides/application_framework/layout/9.views.md) section).
`Collections` contain methods that do not need an object id (`list`, `create`), and `Resources` contain methods that do (`read`, `update`, `delete`).
When developing *Services*, please ensure that the layout of any file that references *Methods* stick to the order as they are described above; `list`, `create`, `read`, `update`, `delete`.

## Requirements
Requirements are split across the applications and the framework.
The requirements files are laid out below, using the typical paths within the Dockerfiles;

- `/application_framework/requirements.txt`: Base requirements for all environments for all applications
    - Examples are; `django`, `cloudcix`, `pyjwt`, etc
- `/application_framework/dev-requirements.txt`: Extra requirements for the testing environments for all applications
    - Examples are; `coverage`
- `/application_framework/app-requirements.txt`: Extra requirements for deployed environments for all applications
    - Examples are; `django-redis`, `gunicorn`, `raven`, etc
- `/application_framework/<application>/requirements.txt`: Extra base requirements for all environments for a specific application
    - Membership also has `ldap3` and `pytz`
    - Can be left empty, but the file should exist just to maintain patterns
- `/application_framework/<appliction>/dev-requirements.txt`: Extra requirements for the testing environment for a specific application
    - Can be left empty, but the file should exist just to maintain patterns

## Settings
Every application will likely have its own extra settings that it needs, and these settings will likely differ between environments also.s;

- `ALLOWED_HOSTS`: A tuple of strings representing the hostnames that the server should accept as allowed. If accessed through a different hostname, all responses will be 400s.
- `CACHES`: A dictionary mapping cache names to locations. We only use the `default` cache for now, and in the test environment it should be a dummy cache
- `CLOUDCIX_INFLUX_DATABASE`: The name of the influx database to report metrics to. Typically None in test environments.
- `DATABASES`: A mapping of database names to details. Typically an application will need a `default` and a database for itself.
- `DATABASE_ROUTERS`: A list of router files that are to be used to route database requests to the correct database
- `DEBUG`: Whether or not the system should be run in DEBUG mode. Usually True only for test env.
- `EMAIL_SUBJECT_PREFIX`: The subject prefix for any emails sent out by the django server. Usually None for test env.
- `INSTALLED_APPS`: A list of installed applications. Will usually just be the application itself.
- `TESTING`: A flag stating whether or not the system is in test mode. Always True in test env, false everywhere else.
- `TRACER_CONFIG`: Config for the Jaeger tracer. Put in local settings to customise config based on application and environment
- `TRACER_SERVICE_NAME`: The service name that will be attached to all spans for the application
- `RAVEN_CONFIG`: Config for the sentry client. Usually None in test env.

Other settings can be added as necessary. The application framework's main settings file runs a star import on settings local, so everything in the local file will be imported and defined.

## URLs
A `urls_local` file which is to be copied into `system_conf`.

Should only contain an `include` directive to import the urls from the application's `urls` file.

Membership's `urls_local` file looks like this;

```python
from django.urls import include, path

urlpatterns = [
    path('', include('membership.urls')),
]
```
