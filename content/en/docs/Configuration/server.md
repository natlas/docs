---
title: "Configuring Natlas Server"
linkTitle: "Configuring Natlas Server"
date: 2020-08-16
weight: 1
description: >
  Learn more about configuring the Natlas Server
---

There are a number of config options that you can specify in the application environment or in a file called `.env` before initializing the database and launching the application. There are also a number of configuration options available for the server which change certain authentication-based behaviors as well as agent work configurations.

## Environment Config

Environment configs are loaded from the environment or a `.env` file and require an application restart to change. Bind mounting a `.env` file to `/opt/natlas/natlas-server/.env` (rather than providing it as a docker env file) is encouraged so that passwords are not visible to the entire container.

### Core

| Variable | Default | Explanation |
|---|---|---|
| `SECRET_KEY` | Randomly generated | Used for CSRF tokens and sessions. You should generate a unique value for this in `.env`, otherwise sessions will be invalidated whenever the app restarts. |
| `SERVER_NAME` | `None` | This should be set to the domain and optional port that your service will be accessed on. Do **NOT** include the scheme here. E.g. `example.com` or `10.0.0.15:5000` |
| `PREFERRED_URL_SCHEME` | `https` | You can optionally set this value to `http` if you're not using ssl. This should be avoided for any production environments. |
| `CONSISTENT_SCAN_CYCLE` | `False` | Setting this to `True` will cause the random scan order to persist between scan cycles. This will produce more consistent deltas between an individual host being scanned. **Note**: Changes to the scope will still change the scan order, resulting in one cycle of less consistent timing. |

### Data Storage

| Variable | Default | Explanation |
|---|---|---|
| `DATA_DIR` | `/data` | Path to store any data that should be persisted. Sqlite database, any log files, and media files all go in subdirectories of this directory. |
| `SQLALCHEMY_DATABASE_URI` | `sqlite:///$DATA_DIR/db/metadata.db` | A [SQLALCHEMY URI](https://flask-sqlalchemy.palletsprojects.com/en/2.x/config/) that points to the database to store natlas metadata in. Supported types by natlas-server are: `sqlite:`, `mysql:` |
| `ELASTICSEARCH_URL` | `http://localhost:9200` | A URL that points to the elasticsearch cluster to store natlas scan data in |
| `MEDIA_DIRECTORY` | `$DATA_DIR/media/` | If you want to store media (screenshots) in a larger mounted storage volume, set this value to an absolute path. If you change this value, be sure to copy the contents of the previous media directory to the new location, otherwise old media will not render.|

### Instrumentation

| Variable | Default | Explanation |
|---|---|---|
| `SENTRY_DSN` | `""` | Enables automatic reporting of all Flask exceptions to a [Sentry.io instance](https://sentry.io/). Example: `http://mytoken@mysentry.example.com/1` |
| `SENTRY_ENVIRONMENT` | `None` | Specifies the value to provided for the `environment` tag in Sentry. Use it to differentiate between different stages or stacks. e.g. `Beta` or `Prod` |
| `SENTRY_JS_DSN` | `""` | Enables automatic reporting of all JS errors to a [Sentry.io instance](https://sentry.io/). This is separate from `SENTRY_DSN` so you can report client-side errors separately from server-side. |
| `OPENCENSUS_ENABLE` | `false` | Enables OpenCensus instrumentation to help identify performance bottlenecks. |
| `OPENCENSUS_SAMPLE_RATE` | `1.0` | Specifies the percentage of requests that are traced with OpenCensus. A number from 0 to 1. |
| `OPENCENSUS_AGENT` | `127.0.0.1:55678` | An OpenCensus agent or collector that this instance will emit traffic to. |

### Mail Settings

| Variable | Default | Explanation |
|---|---|---|
| `MAIL_SERVER` | `None` | Mail server to use for invitations, registrations, and password resets |
| `MAIL_PORT` | `587` | Port that `MAIL_SERVER` is listening on |
| `MAIL_USE_TLS` | `True` | Whether or not to connect to `MAIL_SERVER` with STARTTLS |
| `MAIL_USE_SSL` | `False` | Whether or not to connect to `MAIL_SERVER` with SSL (E.g. Port 465) |
| `MAIL_USERNAME` | `None` | Username (if required) to connect to `MAIL_SERVER` |
| `MAIL_PASSWORD` | `None` | Password (if required) to connect to `MAIL_SERVER` |
| `MAIL_FROM` | `None` | Address to be used as the "From" address for outgoing mail. This is required if `MAIL_SERVER` is set. |

### Development Settings

These settings are mostly only used for development purposes, and can have adverse effects if you modify them in your production environment.

| Variable | Default | Explanation |
|---|---|---|
| `FLASK_ENV` | `production` | Used to tell flask which environment to run. Only change this if you are debugging or developing, and never leave your server running in anything but `production`.  |
| `FLASK_APP` | `natlas-server.py` | The file name that launches the flask application. This should not be changed as it allows commands like `flask run`, `flask db upgrade`, and `flask shell` to run.|
| `NATLAS_VERSION_OVERRIDE` | `None` | **Danger**: This can be optionally set for development purposes to override the version string that natlas thinks it's running. Doing this can have adverse affects and should only be done with caution. The only reason to really do this is if you're developing changes to the way host data is stored and presented. |

### Example ENV

The following is an example ENV file that assumes your elasticsearch cluster is accessible at `172.17.0.2:9200` and that your mail server is accessible with authentication at `172.17.0.4`. If you do not have a mail server, remove `MAIL_` settings.

```bash
#####
# Flask Settings
#####
SECRET_KEY=im-a-really-long-secret-please-dont-share-me
FLASK_ENV=production

#####
# Data Stores
#####
ELASTICSEARCH_URL=http://172.17.0.2:9200

# A mysql database via the mysqlclient driver
#SQLALCHEMY_DATABASE_URI=mysql://natlas:password@172.18.0.5/natlas

# A sqlite database with a custom name vs the default metadata.db
#SQLALCHEMY_DATABASE_URI=sqlite:////data/db/test.db

#####
# Mail settings
#####
MAIL_SERVER=172.17.0.4
MAIL_USERNAME=dade.murphy
MAIL_PASSWORD=this-is-an-invalid-password
MAIL_FROM=noreply@example.com

#####
# Natlas Specific Settings
#####
CONSISTENT_SCAN_CYCLE=True
```

## Web Config

Web configs are loaded from the SQL database and changeable from the web interface without requiring an application restart.

| Variable | Default | Explanation |
|---|---|---|
| `LOGIN_REQUIRED` | `True` | Require login to browse results |
| `REGISTER_ALLOWED` | `False` | Permit open registration for new users |
| `AGENT_AUTHENTICATION` | `True` | Optionally require agents to authenticate before being allowed to get or submit work |
| `CUSTOM_BRAND` | `""` | Custom branding for the navigation bar to help distinguish different natlas installations from one another |
