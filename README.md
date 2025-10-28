# Drupal API Export Project

This project provides a Docker-based setup for building a Views REST export of the Drupal core API, specifically designed for creating vector databases or generating training datasets for machine learning models.

## Prerequisites

- Docker
- Docker Compose

## Setup Instructions

1. Clone the repository:

```bash
git clone git@github.com:npacker/drupal-api-docker.git
cd drupal-api-docker
```

2. Configure the `.env` file with appropriate values for your environment:

```env
# Database configuration
DRUPAL_DB_HOST=mariadb
DRUPAL_DB_PORT=3306
DRUPAL_DB_NAME=drupal
DRUPAL_DB_USER=drupal
DRUPAL_DB_PASS=drupal

# Drupal site configuration
DRUPAL_SITE_NAME="Drupal API Export"
DRUPAL_ADMIN_USER=admin
DRUPAL_ADMIN_PASS=admin

3. Start the services:

```bash
docker-compose up -d
```

4. Wait for the services to initialize (this may take a few minutes). You can check the logs with:

```bash
docker-compose logs -f
```

## Project Structure

- `.env`: Environment variables for the Docker Compose setup
- `docker-compose.yml`: Configuration for Docker services (Drupal and MariaDB)
- `drupal/Dockerfile`: Custom Dockerfile for the Drupal service
- `drupal/drush-site-precreate`: Script for initializing the Drupal site and processing the API
- `drupal/config/views.view.drupal_api_export.yml`: Drupal view configuration for the REST export

## Using the Views REST Export

The project includes a pre-configured Views REST export at `/export/api/drupal/11.x`. To access the API:

1. Ensure the Drupal service is running
2. Navigate to `http://localhost:8383/export/api/drupal/11.x` in your browser or use a tool like `curl`:

```bash
curl http://localhost:8383/export/api/drupal/11.x
```

## Customizing the Views Configuration

The Views configuration is stored in `drupal/config/views.view.drupal_api_export.yml`. To modify the API output:

1. Edit the configuration file
2. Import the configuration:

```bash
docker-compose exec drupal drush config:import --partial --source=/opt/drupal/config
```

## Updating the API

To update the API data:

1. Ensure the Drupal service is running
2. Run the following command to reprocess the API:

```bash
docker-compose exec drupal /opt/drupal/web/drush-site-precreate
```

## Notes

- The project is designed to work with version 11.x of Drupal core
- The `drush-site-precreate` script handles site installation, module configuration, and API processing
- For development, you can modify files in the `drupal/config` directory and re-import them using `drush config:import`
