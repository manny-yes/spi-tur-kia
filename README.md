# Spina CMS Dockerized Project

This project provides a complete setup for building, deploying, and running Spina CMS (v2.6.2) in a Dockerized environment. The repository includes options for deploying the pre-built Docker image from GHCR or building the image locally.

The deployment is based on Docker Compose with two services defined: `spina` and `spinadb`. The first has the application itself, the second is a PostgreSQL database based on the `postgres:15` image. There's a healthcheck included in the `spinadb` definition, which is based on the `pg_isready` command. The service `spina` waits until `spinadb` is healthy, then proceeds to start. Spina requires a database to be available beforehand, otherwise it fails during initialization.

## Quickstart
Clone the repo, go to the `deploy/` directory and run `docker-compose -f docker-compose.yml --env-file custom.env up -d`. App will be available at http://localhost:3000.

## Workflows
[![Docker CI - Compose](https://github.com/manny-yes/spi-tur-kia/actions/workflows/docker-integration-tests.yml/badge.svg?event=pull_request)](https://github.com/manny-yes/spi-tur-kia/actions/workflows/docker-integration-tests.yml)

[![Docker CI - Publish](https://github.com/manny-yes/spi-tur-kia/actions/workflows/docker-build-publish.yml/badge.svg?event=push)](https://github.com/manny-yes/spi-tur-kia/actions/workflows/docker-build-publish.yml)

## Repository Structure

```
.
├── README.md
├── deploy
│   ├── Dockerfile
│   ├── custom.env
│   ├── docker-compose-build.yml
│   └── docker-compose.yml
├── src
│   ├── (contents of SpinaCMS v2.6.2)
└── .github
    └── workflows
        ├── docker-build-publish.yml
        └── docker-integration-tests.yml
```

## Deployment Instructions

### Preparations
After cloning the repository, edit the contents of `custom.env`:
```bash
https://github.com/manny-yes/spi-tur-kia.git
cd spi-tur-kia
vim deploy/custom.env
```

### Option 1: Pull Pre-Built Image from GHCR
This option pulls the published image from `ghcr.io/manny-yes/spi-tur-kia/spina:latest` and starts the services with docker compose:
```bash
cd deploy
docker-compose -f docker-compose.yml --env-file custom.env up -d
```

### Option 2. Build and Deploy Locally
This option leverages the option for building and running the application with docker compose file in one step. The built image is stored locally at `localhost/manny-yes/spi-tur-kia/spina:latest`:
```bash
cd deploy
docker-compose -f docker-compose-build.yml --env-file custom.env up -d
```

### Local Image Build Only
To build the image locally without running it, go to the root path of the repository and run:
```bash
docker build -t localhost/spina:v2.6.2-localbuild -f deploy/Dockerfile .
```
The entrypoint of all built is `bash`, which allows for code inspection but don't start any services.

## Developer Experience

### Bundle Update
To perform a `bundle update` and update dependencies:
1. Deploy the app with compose using one of the two options provided.
2. Open a shel inside the container and run the update:
```bash
docker exec -it deploy-spina-1 bash
root@e67f16671b06:/app# bundle update
# Fetching gem metadata from https://rubygems.org/...........
# Resolving dependencies...
# Fetching rake 13.2.1 (was 13.0.6)
# (...)
# Fetching rails 6.1.7.10 (was 6.1.4.4)
# Installing rails 6.1.7.10 (was 6.1.4.4)
# Bundle updated!
root@e67f16671b06:/# exit
```
3. Go to the `src` directory and copy the updated `Gemfile.lock` file from the container
```bash
cd ../src
docker cp deploy-spina-1:/app/Gemfile.lock Gemfile.lock 
```

[Pull request #4](https://github.com/manny-yes/spi-tur-kia/pull/4) shows successfully updated dependencies and passed integration tests. There's also a published image with the `:v2.6.2-update` tag.

### Docker CI Workflow for Integration Tests
- **Purpose**: Validates image builds, service availability, and HTTP responses for Spina. Provides instant feedback for proposed changes.
- **Key Features**:
  - Build the image and deploy it using Docker Compose.
  - Run TCP availability and HTTP validation tests.
  - Blocks merging changes if tests fail.
- **Workflow File**: `.github/workflows/docker-integration-tests.yml`

### Docker CI Workflow for Publishing Images
- **Purpose**: Publishes images to GHCR when new tags are added.
- **Key Features**:
  - Triggered on `v*` tag creation.
  - Publishes the image to `ghcr.io`.
- **Workflow File**: `.github/workflows/docker-build-publish.yml`

## Source Code and Images
- The `src/` directory contains the unchanged contents of [SpinaCMS v2.6.2](https://github.com/SpinaCMS/Spina/archive/refs/tags/v2.6.2.tar.gz).
- Images
    - Image `ghcr.io/manny-yes/spi-tur-kia/spina:v2.6.2` has the SpinaCMS v2.6.2 source code as is.
    - Image `ghcr.io/manny-yes/spi-tur-kia/spina:v2.6.2-update` has an updated `Gemfile.lock` after a `bundle update`.
- To deploy a different published image, just edit the image on the `docker-compose.yml` file.

## Problems Found

### PG Gem Build Error
- **Error**:
  ```bash
  pg_result.c:1590:65: error: ‘rb_cData’ undeclared (first use in this function)
  ```
- **Solution**: Added `RUN bundle update pg` to the `Dockerfile` to resolve the issue at build time.