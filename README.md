# CinRes — Cinema Reservation (PHP + MySQL)

> Lightweight, legacy PHP web app for reserving cinema tickets (circa 2003).

This repository contains a small procedural PHP application that demonstrates a simple cinema reservation workflow. It was written for older PHP versions and uses the deprecated `mysql_*` API. See the Compatibility section below before running on modern systems.

## Table of contents

- Project overview
- Quick start (recommended: Docker)
- Prerequisites
- Installation
- Database setup
- Configuration
- Run the app
- Admin / usage
- Troubleshooting
- Security notes
- Contributing
- License & credits

## Project overview

Files of interest:

- `index.php` — public landing page (lists films, links to ordering)
- `index_admin.php` — admin UI for managing films and shows
- `includes/` — helper files and DB connectors (`mysql.filme.php`, `mysql.kino.php`) and CSS
- `kino.sql` — SQL dump to create the database schema and seed data

The app is small and procedural. It uses `mysql_query`, `mysql_connect`, and related functions that were removed in PHP 7+. You can either run it in a legacy PHP 5.x environment, or run it inside a Docker container that provides an older PHP image, or port the code to `mysqli`/`PDO` (recommended).

## Quick start (recommended)

The easiest, most reproducible way to run this project on a modern machine is with Docker Compose. This example runs an Apache+PHP image compatible with `mysql_*` and a MySQL server.

1. Ensure Docker and Docker Compose are installed.
2. Create a `docker-compose.yml` (example below), then run `docker-compose up -d`.

Example `docker-compose.yml` (pick a PHP 5.6 Apache image if you need `mysql_*` support):

```yaml
version: '3.8'
services:
  web:
    image: php:5.6-apache
    ports:
      - "8080:80"
    volumes:
      - ./:/var/www/html
    depends_on:
      - db
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: kino
    volumes:
      - db_data:/var/lib/mysql
volumes:
  db_data:

```

Start the stack:

```bash
docker-compose up -d
```

Import the database (from host):

```bash
docker exec -i $(docker-compose ps -q db) mysql -u root -pexample kino < kino.sql
```

Open a browser at http://localhost:8080/

## Prerequisites

- PHP (legacy behavior): if you want to run this app without containerization you'll need PHP 5.x (the app uses `mysql_*`). Modern PHP (7.0+) removed `mysql_*`.
- MySQL or MariaDB (for the database)
- A web server (Apache/Nginx) or the PHP built-in server for quick testing (but note compatibility concerns)

If you prefer to update the code to run on current PHP versions, convert `mysql_*` calls to `mysqli` or `PDO` first.

## Installation (manual / non-Docker)

1. Place the project folder into your web server document root (or serve it using `php -S`).
2. Create a MySQL database and import `kino.sql`.
3. Edit the DB connection files in `includes/` to match your DB credentials (see Configuration below).

## Database setup

Import the included SQL dump:

```bash
mysql -u root -p kino < kino.sql
```

If the `kino` database does not exist yet, create it first:

```sql
CREATE DATABASE kino;
```

## Configuration

The two connection helper files you will likely need to edit are in `includes/`:

- `includes/mysql.kino.php`
- `includes/mysql.filme.php`

Open these files and update host/user/password/dbname values to match your MySQL server. Example (conceptual):

```php
<?php
$db_host = 'localhost';
$db_user = 'root';
$db_pass = 'example';
$db_name = 'kino';
// legacy code may use mysql_connect/mysql_select_db — consider migrating to mysqli or PDO
?>
```

Note: The exact variables and code in those files may differ. Search for `mysql_connect`, `mysql_select_db`, or `$dbhost` to find the lines to change.

## Run the app

- With Docker Compose (recommended):

```bash
docker-compose up -d
# import kino.sql as shown above
```

- Quick local test (if you have a compatible PHP installed):

```bash
cd /path/to/project
php -S 0.0.0.0:8000
# visit http://localhost:8000/
```

Be aware: on a modern host with PHP 7+, the app will fail at runtime due to removed `mysql_*` functions.

## Admin / usage

- Manage films and shows through `index_admin.php`.
- From the public site `index.php` users can view available films and start a reservation flow.

## Troubleshooting

- "Call to undefined function mysql_connect()": you are running PHP 7+. Either run the app in an older PHP environment, or port the code to `mysqli`/`PDO`.
- Database connection errors: verify host, user, password in the `includes/mysql*.php` files and ensure the MySQL service is running.
- Port conflicts with Docker: change the host port mapping (e.g., `8081:80`) in `docker-compose.yml`.

## Security notes

This project is a legacy educational example and is NOT production-ready:

- It uses deprecated `mysql_*` functions and does not follow modern best practices.
- Input handling likely lacks prepared statements — SQL injection risk.
- Do not expose this app to the public internet without a security audit and updates.

Recommended actions before real-world use:

1. Migrate DB code to `mysqli` or `PDO` with prepared statements.
2. Add input validation and output escaping.
3. Harden authentication and session handling.

## Contributing

This repository appears to be a personal/educational project. If you want to modernize it, consider the following roadmap:

1. Replace deprecated `mysql_*` calls with `mysqli` or `PDO`.
2. Add a database migration script and use parameterized queries.
3. Add automated tests for core flows (create film, add show, reserve ticket).

If you'd like, I can help convert the DB layer to `mysqli`/`PDO` step-by-step.

## License & credits

This project includes an old GPL notice in `index.php`. Treat the project as GPL-licensed (original author: Georg Knoerr, 2003). See the in-file header for details.

---

If you want, I can also:

- Create a `docker-compose.yml` for you and start the stack locally.
- Port the connection code to `mysqli` or `PDO` so it runs on modern PHP.

Tell me which you'd prefer and I'll proceed.
