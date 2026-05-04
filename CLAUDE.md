# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

A Docker Compose stack that boots a Drupal site whose sole job is to parse Drupal core's source tree with the `drupal/api` module and expose the parsed docblocks as a Views REST endpoint. The export is consumed downstream for vector databases / LLM training datasets — the bundled `unsloth` GPU container is intended to be the consumer of that data.

The Drupal site itself is disposable infrastructure for producing the export. Treat it as a build artifact, not a long-lived application.

## Common commands

```bash
# Bring the stack up (first run installs Drupal, clones core, parses the API — takes a long time)
docker-compose up -d

# Tail initialization (parsing the queue is the slow part)
docker-compose logs -f drupal

# Re-run the full parse pipeline against an already-installed site
docker-compose exec drupal /usr/local/bin/drush-site-precreate

# Apply changes to drupal/config/*.yml without a rebuild
docker-compose exec drupal drush config:import --partial --source=/opt/drupal/config

# Hit the export
curl http://localhost:8383/export/api/drupal/11.x
```

Service ports: drupal `8383`, mariadb `3306`, elasticsearch `9200`, unsloth jupyter `8888` / app `8000` / ssh `2222`.

## Architecture

Four services in [docker-compose.yml](docker-compose.yml):

- **drupal** — built from [drupal/Dockerfile](drupal/Dockerfile) (drupal:10.5-php8.3 + composer-installed `drupal/api` and `drush/drush`). The container's ENTRYPOINT is the `drush-site-precreate` script, *not* `apache2-foreground` — Apache is `exec`'d at the end of the script.
- **mariadb** — backing store. The drupal service `depends_on` it with `condition: service_healthy`.
- **elasticsearch** — single-node, security disabled, 10G mem limit. Provisioned for downstream indexing of the export; the Drupal site does not talk to it directly in the current scripts.
- **unsloth** — GPU (nvidia runtime) container for fine-tuning. Mounts `./unsloth/work` as `/workspace/work`. Independent of the Drupal pipeline.

### The drush-site-precreate lifecycle

[drupal/drush-site-precreate](drupal/drush-site-precreate) is the heart of the project. It is **idempotent by design** — it branches on `drush core:status --field=bootstrap`:

1. **Cold start (not bootstrapped):** `drush site:install standard` → install `api` and `rest` modules → `git clone --depth=1 --branch 11.x` Drupal core into `web/sites/default/files/api_git_repositories/drupal` → run `process_api_queue` → `config:import --partial` from `/opt/drupal/config`.
2. **Warm start (already bootstrapped):** `drush updatedb` only. The API is *not* re-parsed on warm boot — to refresh the data you must invoke the script manually (see commands above).

`process_api_queue` is the parsing loop: it `api:upsert-branch` + `api:re-parse`, runs cron to enqueue, then drains `api_parse_queue` with `NUM_PROCESSES=8` parallel `drush queue:run` workers in a loop until the queue size is 0. Tunables (`NUM_PROCESSES`, `TIME_LIMIT`, `LEASE_TIME`, `UPDATE_FREQUENCY`) are constants at the top of the script.

The site root is bind-mounted as an anonymous volume (`/var/www/html/sites`), but `/opt/drupal/config` is a host bind from `./drupal/config`. Edits to YAML there are visible inside the container immediately and applied via `drush config:import`.

### The Views export

[drupal/config/views.view.drupal_api_export.yml](drupal/config/views.view.drupal_api_export.yml) defines the `drupal_api_export` view over the `api_branch_docblock` base table. This is the source of truth for the export's shape — adding/removing fields here and re-importing config is the normal way to change the API output. The path is `/export/api/drupal/11.x` (REST display).

## Conventions

- Config is the single source of truth for the export shape — change YAML, don't poke the DB.
- Drupal core branch / version is hard-coded in `drush-site-precreate` (`BRANCH=11.x`, `CORE_COMPATIBILITY=11.x`). Changing target version requires editing both the script and likely the view's path.
- The Dockerfile pins Drupal 10.5 (the host site) but parses Drupal **11.x** source. Don't conflate the two.
