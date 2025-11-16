# Docker Deployment Guide

This directory contains Docker-related configuration files for the webroot project. The project uses Docker Compose to orchestrate multiple services in a development and production environment.

## Table of Contents

- [Quick Start](#quick-start)
- [Architecture Overview](#architecture-overview)
- [Current Services](#current-services)
- [Directory Structure](#directory-structure)
- [Adding New Services](#adding-new-services)
- [Database Management](#database-management)
- [Environment Variables](#environment-variables)
- [Networking](#networking)
- [Volumes and Data Persistence](#volumes-and-data-persistence)
- [Common Commands](#common-commands)
- [Troubleshooting](#troubleshooting)

## Quick Start

**Prerequisites:** Create a `.env` file at the project root (required for PostgreSQL credentials):

```bash
# Create .env file with PostgreSQL credentials
cat > .env << 'EOF'
POSTGRES_DEFAULT_USER=postgres
POSTGRES_DEFAULT_PASSWORD=<safepassword>
POSTGRES_DEFAULT_DB=postgres
EOF
```

Then start the services:

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down

# Stop all services and remove volumes (⚠️ destroys data)
docker-compose down -v
```

## Architecture Overview

The Docker Compose setup orchestrates three main services:

1. **web** - Python HTTP server for serving static files
2. **postgres** - PostgreSQL database with multiple databases
3. **api** - Rust-based API server

All services communicate via a shared Docker network (`webroot-network`).

## Current Services

### Web Service

- **Image**: `python:3.11-slim`
- **Port**: 8887
- **Purpose**: Serves static files via Python's built-in HTTP server
- **Access**: http://localhost:8887

### PostgreSQL Service

- **Image**: `postgres:15-alpine`
- **Port**: 5432
- **Databases**: membercommons, exiobase, suitecrm
- **Credentials** (configured in root `.env`):
  - User: `${POSTGRES_DEFAULT_USER}` (default: `postgres`)
  - Password: `${POSTGRES_DEFAULT_PASSWORD}` (set in `.env`)
  - Database: `${POSTGRES_DEFAULT_DB}` (default: `postgres`)
- **Features**:
  - Health checks enabled
  - Automatic multiple database creation
  - SQL schema initialization on first run
  - Data persistence via named volume

**⚠️ Important:** The PostgreSQL service requires a root `.env` file with credentials. See [Environment Variables](#environment-variables) for setup instructions.

### API Service

- **Base Image**: `rust:1.91-slim-bookworm`
- **Port**: 8081
- **Purpose**: Rust-based API server (partner_tools)
- **Build Context**: `./team`
- **Features**:
  - Hot-reload support via volume mounting
  - Cargo cache persistence for faster rebuilds
  - Depends on postgres service

## Directory Structure

```
docker/
├── README.md                    # This file
└── postgres/
    └── init-multiple-databases.sh  # Database initialization script
```

### PostgreSQL Initialization

The postgres service uses an initialization script ([init-multiple-databases.sh](postgres/init-multiple-databases.sh)) that:

1. Creates multiple databases from the `POSTGRES_MULTIPLE_DATABASES` environment variable
2. Grants privileges to the postgres user
3. Applies SQL schemas from `team/admin/sql/` directory

## Adding New Services

To add a new service to the Docker Compose setup, follow these steps:

### 1. Define the Service in docker-compose.yml

Add your service definition to the root [docker-compose.yml](../docker-compose.yml):

```yaml
services:
  your-service-name:
    # Option A: Use a pre-built image
    image: node:18-alpine

    # Option B: Build from a Dockerfile
    build:
      context: ./path/to/service
      dockerfile: Dockerfile

    container_name: webroot-your-service
    working_dir: /app

    # Mount source code for development
    volumes:
      - ./path/to/service:/app
      - node_modules:/app/node_modules  # Optional: cache dependencies

    # Expose ports (host:container)
    ports:
      - "3000:3000"

    # Environment variables
    environment:
      - NODE_ENV=development

    # Or use env file
    env_file:
      - ./path/to/service/.env

    # Service dependencies
    depends_on:
      - postgres

    # Connect to network
    networks:
      - webroot-network

    # Restart policy
    restart: unless-stopped

    # Health check (optional but recommended)
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

    # Start command
    command: npm start
```

### 2. Create Service-Specific Configuration (if needed)

If your service requires initialization scripts or configuration:

```bash
mkdir -p docker/your-service-name
# Add configuration files, scripts, etc.
```

### 3. Add Volume Definitions (if needed)

Add named volumes for data persistence to the `volumes` section:

```yaml
volumes:
  postgres-data:
  cargo-cache:
  cargo-git:
  target-cache:
  your-service-data:  # Add your volume here
```

### 4. Update .dockerignore (if needed)

Ensure unnecessary files aren't copied into your Docker build context by updating [.dockerignore](../.dockerignore).

### 5. Test Your Service

```bash
# Build and start only your service
docker-compose up -d your-service-name

# View logs
docker-compose logs -f your-service-name

# Verify service is running
docker-compose ps
```

## Database Management

### Accessing PostgreSQL

```bash
# Connect via docker exec
docker exec -it webroot-postgres psql -U postgres -d membercommons

# Connect from host (requires psql installed)
psql -h localhost -p 5432 -U postgres -d membercommons
```

### Adding New Databases

To create additional databases:

1. Update the `POSTGRES_MULTIPLE_DATABASES` environment variable in [docker-compose.yml](../docker-compose.yml):

```yaml
environment:
  POSTGRES_MULTIPLE_DATABASES: membercommons,exiobase,suitecrm,newdb
```

2. (Optional) Add initialization SQL schema:

```bash
# Place schema file in team/admin/sql/
cp your-schema.sql team/admin/sql/
```

3. Update [init-multiple-databases.sh](postgres/init-multiple-databases.sh) if custom logic is needed

4. Recreate the postgres container:

```bash
docker-compose down
docker volume rm webroot_postgres-data  # ⚠️ Destroys existing data
docker-compose up -d postgres
```

## Environment Variables

The project uses **two separate `.env` files** for different purposes:

### 1. Root `.env` File (Docker Compose Substitution)

Located at the project root (`.env`), this file contains variables used for **variable substitution** in `docker-compose.yml`:

```bash
# Environment variables for Docker Compose
# These are used for variable substitution in docker-compose.yml

# PostgreSQL credentials (used for POSTGRES_USER and POSTGRES_PASSWORD)
POSTGRES_DEFAULT_USER=postgres
POSTGRES_DEFAULT_PASSWORD=<safepassword>
POSTGRES_DEFAULT_DB=postgres
```

**Important:** Docker Compose reads this file automatically for substituting `${VARIABLE_NAME}` placeholders in the docker-compose.yml file. This file is:
- ✅ Already in `.gitignore` - will not be committed to version control
- ✅ Already in `.dockerignore` - will not be included in Docker images

### 2. Service-Specific `.env` File (Container Environment)

Located at `team/.env`, this file contains environment variables that are **loaded inside containers**:

```yaml
# In docker-compose.yml
env_file:
  - ./team/.env
```

This file contains application-specific configuration like database connection strings, API keys, and service settings.

### Variable Substitution vs Container Environment

**Key Difference:**
- **Variable substitution** (`${VAR}` in docker-compose.yml): Uses root `.env` or host environment
- **Container environment** (`env_file` directive): Makes variables available inside the running container

Example in docker-compose.yml:
```yaml
postgres:
  environment:
    # These ${} variables are substituted from ROOT .env
    POSTGRES_USER: ${POSTGRES_DEFAULT_USER}
    POSTGRES_PASSWORD: ${POSTGRES_DEFAULT_PASSWORD}
  env_file:
    # This file's variables are available INSIDE the container
    - ./team/.env
```

### Setting Up Environment Variables

1. **First time setup**: Copy the root `.env.example` (if exists) or create `.env` at project root:
   ```bash
   # Create root .env file
   cat > .env << 'EOF'
   POSTGRES_DEFAULT_USER=postgres
   POSTGRES_DEFAULT_PASSWORD=YourSecurePasswordHere
   POSTGRES_DEFAULT_DB=postgres
   EOF
   ```

2. **Service configuration**: Copy and configure `team/.env`:
   ```bash
   cp team/.env.example team/.env
   # Edit team/.env with your configuration
   ```

## Networking

All services communicate via the `webroot-network` bridge network. Services can reference each other by their service name (e.g., `postgres`, `api`, `web`).

### Connecting from one service to another

```bash
# From api service to postgres
postgresql://postgres:postgres@postgres:5432/membercommons

# From api to web service
http://web:8887
```

### Accessing from host

Use `localhost` with the exposed port:

```bash
# PostgreSQL
postgresql://postgres:postgres@localhost:5432/membercommons

# API
http://localhost:8081

# Web
http://localhost:8887
```

## Volumes and Data Persistence

The project uses both bind mounts and named volumes:

### Named Volumes (Managed by Docker)

- `postgres-data` - PostgreSQL database files
- `cargo-cache` - Rust dependency cache
- `cargo-git` - Git dependencies for Cargo
- `target-cache` - Rust build artifacts

Named volumes persist data across container restarts and are managed by Docker. **Note:** These volumes are stored in Docker's internal storage (not as folders in your project directory), which is why you won't see them locally. This approach provides better performance (especially on macOS/Windows) and keeps your project directory clean. To inspect these volumes, use `docker volume ls` or `docker volume inspect webroot_volume-name`.

### Bind Mounts (Host filesystem)

- `.:/app` (web service) - Entire project directory
- `./team:/app/team` (api service) - Source code for hot-reload
- `./docker/postgres/init-multiple-databases.sh` - Database init script

Bind mounts allow real-time code changes without rebuilding containers.

## Common Commands

```bash
# Start all services in background
docker-compose up -d

# Start specific service
docker-compose up -d postgres

# View logs for all services
docker-compose logs -f

# View logs for specific service
docker-compose logs -f api

# Restart a service
docker-compose restart api

# Rebuild a service
docker-compose up -d --build api

# Stop all services
docker-compose down

# Stop and remove volumes (⚠️ destroys data)
docker-compose down -v

# Execute command in running container
docker-compose exec api bash
docker-compose exec postgres psql -U postgres

# View running containers
docker-compose ps

# View resource usage
docker stats
```

## Troubleshooting

### Missing Environment Variables Error

**Symptoms:**
```
WARN[0000] The "POSTGRES_DEFAULT_USER" variable is not set. Defaulting to a blank string.
WARN[0000] The "POSTGRES_DEFAULT_PASSWORD" variable is not set. Defaulting to a blank string.
```

Or PostgreSQL container fails with:
```
Error: Database is uninitialized and superuser password is not specified.
You must specify POSTGRES_PASSWORD to a non-empty value for the superuser.
```

**Solution:**

1. Create a `.env` file at the project root (same directory as `docker-compose.yml`):
   ```bash
   cat > .env << 'EOF'
   POSTGRES_DEFAULT_USER=postgres
   POSTGRES_DEFAULT_PASSWORD=<safepassword>
   POSTGRES_DEFAULT_DB=postgres
   EOF
   ```

2. Restart Docker Compose:
   ```bash
   docker-compose down
   docker-compose up -d
   ```

**Why this happens:** Docker Compose uses variable substitution (`${VAR_NAME}`) in the docker-compose.yml file. These variables must be defined in either:
- A `.env` file at the project root (recommended)
- The host system's environment variables

The `env_file` directive in docker-compose.yml only makes variables available **inside** containers, not for docker-compose.yml substitution.

### Service won't start

```bash
# Check logs
docker-compose logs -f service-name

# Check container status
docker-compose ps

# Inspect container
docker inspect webroot-service-name
```

### Port already in use

```bash
# Find process using port
lsof -i :8081

# Kill process or change port in docker-compose.yml
```

### Database connection refused

```bash
# Ensure postgres is running and healthy
docker-compose ps postgres

# Check health status
docker inspect webroot-postgres | grep -A 10 Health

# Wait for postgres to be ready
docker-compose logs -f postgres
```

### Clear all data and restart fresh

```bash
# Stop everything
docker-compose down -v

# Remove all containers and volumes
docker-compose rm -f
docker volume prune

# Start fresh
docker-compose up -d
```

### Rebuild service from scratch

```bash
# Remove container and rebuild
docker-compose rm -f service-name
docker-compose build --no-cache service-name
docker-compose up -d service-name
```

### Check service connectivity

```bash
# From inside a container
docker-compose exec api ping postgres
docker-compose exec api curl http://web:8887

# Check network
docker network inspect webroot_webroot-network
```

## Best Practices

1. **Always create root `.env` file** - Required for Docker Compose variable substitution. Never commit this file to version control.
2. **Use health checks** - Add health checks to ensure services are ready before dependents start
3. **Separate environment files** - Use root `.env` for Docker Compose substitution, service-specific `.env` files for container environment variables
4. **Use .env files** - Keep secrets out of docker-compose.yml
5. **Version your images** - Use specific version tags instead of `latest`
6. **Document environment variables** - Create `.env.example` files
7. **Use named volumes** - For data that should persist across container recreations
8. **Use bind mounts** - For development code that needs hot-reload
9. **Set restart policies** - Use `restart: unless-stopped` for production services
10. **Optimize build context** - Use `.dockerignore` to exclude unnecessary files
11. **Multi-stage builds** - For production, consider multi-stage Dockerfiles to reduce image size
12. **Resource limits** - Consider adding CPU/memory limits for production deployments

## Additional Resources

- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Dockerfile Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [PostgreSQL Docker Image](https://hub.docker.com/_/postgres)
- [Rust Docker Image](https://hub.docker.com/_/rust)
