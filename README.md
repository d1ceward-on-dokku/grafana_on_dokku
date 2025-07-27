![](.github/images/repo_header.png)

[![Grafana](https://img.shields.io/badge/Grafana-12.1.0-blue.svg)](https://github.com/grafana/grafana/releases/tag/v12.1.0)
[![Dokku](https://img.shields.io/badge/Dokku-Repo-blue.svg)](https://github.com/dokku/dokku)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://github.com/d1ceward-on-dokku/grafana_on_dokku/graphs/commit-activity)

# Run Grafana on Dokku

## Overview

This guide explains how to deploy [Grafana](https://grafana.com/), an open-source metrics dashboard and graph editor, on a [Dokku](http://dokku.viewdocs.io/dokku/) host. Dokku is a lightweight PaaS that simplifies deploying and managing applications using Docker.

## Prerequisites

Before proceeding, ensure you have the following:

- A working [Dokku host](http://dokku.viewdocs.io/dokku/getting-started/installation/).
- The [PostgreSQL plugin](https://github.com/dokku/dokku-postgres) installed on Dokku.
- (Optional) The [Let's Encrypt plugin](https://github.com/dokku/dokku-letsencrypt) for SSL certificates.

## Setup Instructions

### 1. Create the App

Log into your Dokku host and create the `grafana` app:

```bash
dokku apps:create grafana
```

### 2. Configure the App

#### Install, Create, and Link PostgreSQL Plugin

1. Install the PostgreSQL plugin:

    ```bash
    dokku plugin:install https://github.com/dokku/dokku-postgres.git postgres
    ```

2. Create a PostgreSQL service:

    ```bash
    dokku postgres:create grafana
    ```

3. Link the PostgreSQL service to the app:

    ```bash
    dokku postgres:link grafana grafana
    ```

#### Set Environment Variables

1. Retrieve the `DATABASE_URL` from the app configuration:

    ```bash
    dokku config grafana
    ```

2. Set the `GF_DATABASE_URL` environment variable:

    ```bash
    dokku config:set grafana GF_DATABASE_URL='previously_copied_database_url'
    ```

3. Set the HTTP port for Grafana:

    ```bash
    dokku config:set grafana GF_SERVER_HTTP_PORT=5000
    ```

4. Generate and set a secret key for Grafana:

    ```bash
    dokku config:set grafana GF_SECURITY_SECRET_KEY=$(echo `openssl rand -base64 45` | tr -d \=+ | cut -c 1-32)
    ```

### 3. Configure the Domain and Ports

Set the domain for your app to enable routing:

```bash
dokku domains:set grafana grafana.example.com
```

Map the internal port `5000` to the external port `80`:

```bash
dokku ports:set grafana http:80:5000
```

### 4. Deploy the App

You can deploy the app to your Dokku server using one of the following methods:

#### Option 1: Deploy Using `dokku git:sync`

If your repository is hosted on a remote Git server with an HTTPS URL, you can deploy the app directly to your Dokku server using `dokku git:sync`. This method also triggers a build process automatically. Run the following command:

```bash
dokku git:sync --build grafana https://github.com/d1ceward-on-dokku/grafana_on_dokku.git
```

This will fetch the code from the specified repository, build the app, and deploy it to your Dokku server.

#### Option 2: Clone the Repository and Push Manually

If you prefer to work with the repository locally, you can clone it to your machine and push it to your Dokku server manually:

1. Clone the repository:

    ```bash
    # Via HTTPS
    git clone https://github.com/d1ceward-on-dokku/grafana_on_dokku.git
    ```

2. Add your Dokku server as a Git remote:

    ```bash
    git remote add dokku dokku@example.com:grafana
    ```

3. Push the app to your Dokku server:

    ```bash
    git push dokku master
    ```

Choose the method that best suits your workflow.

### 5. Enable SSL (Optional)

Secure your app with an SSL certificate from Let's Encrypt:

1. Add the HTTPS port:

    ```bash
    dokku ports:add grafana https:443:5000
    ```

2. Install the Let's Encrypt plugin:

    ```bash
    dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git
    ```

3. Set the contact email for Let's Encrypt:

    ```bash
    dokku letsencrypt:set grafana email you@example.com
    ```

4. Enable Let's Encrypt for the app:

    ```bash
    dokku letsencrypt:enable grafana
    ```

## Wrapping Up

Congratulations! Your Grafana instance is now up and running. You can access it at [https://grafana.example.com](https://grafana.example.com).

To add Grafana plugins, simply set the environment variable named `GF_INSTALL_PLUGINS`:

```bash
dokku config:set grafana GF_INSTALL_PLUGINS=grafana-piechart-panel,grafana-github-datasource
```
