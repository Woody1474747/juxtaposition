# Development Environment Setup

This guide will help you set up a local development environment for Juxtaposition, the Pretendo Network's Miiverse replacement.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Quick Start](#quick-start)
3. [Detailed Setup](#detailed-setup)
4. [Configuration](#configuration)
5. [Running the Services](#running-the-services)
6. [Testing with Console/Emulator](#testing-with-consoleemulator)
7. [Troubleshooting](#troubleshooting)
8. [Additional Resources](#additional-resources)

## Prerequisites

Before you begin, ensure you have the following installed:

- **Node.js 20 or higher** - [Download](https://nodejs.org/)
- **npm** (comes with Node.js)
- **Docker** (highly recommended) - [Download](https://www.docker.com/)
- **Git** - [Download](https://git-scm.com/)

### Required External Services

Juxtaposition requires two other Pretendo Network services to function:
- [Account Server](https://github.com/PretendoNetwork/account) - Handles user authentication
- [Friends Server](https://github.com/PretendoNetwork/friends) - Manages friend relationships

These services are automatically provided via Docker in the development setup.

## Quick Start

For the fastest setup, use the Docker preset:

```bash
# 1. Clone the repository
git clone https://github.com/Woody1474747/juxtaposition.git
cd juxtaposition

# 2. Start Docker services (MongoDB, Redis, MinIO, Account, Friends servers)
cd .docker
docker compose up -d
cd ..

# 3. Set up and run the miiverse-api service
cd apps/miiverse-api
npm install
PN_MIIVERSE_API_USE_PRESETS=docker npm run dev
```

Open a new terminal:

```bash
# 4. Set up and run the juxtaposition-ui service
cd apps/juxtaposition-ui
npm install
PN_JUXTAPOSITION_UI_USE_PRESETS=docker npm run dev
```

The services will be available at:
- **Miiverse API**: http://localhost:8080
- **Juxtaposition UI**: http://localhost:5173

## Detailed Setup

### Step 1: Clone the Repository

```bash
git clone https://github.com/Woody1474747/juxtaposition.git
cd juxtaposition
```

### Step 2: Install Dependencies

This project uses npm workspaces. You can install all dependencies from the root:

```bash
npm install
```

Or install dependencies for each service individually:

```bash
# Install miiverse-api dependencies
cd apps/miiverse-api
npm install
cd ../..

# Install juxtaposition-ui dependencies
cd apps/juxtaposition-ui
npm install
cd ../..
```

### Step 3: Start Docker Services

The `.docker` directory contains a Docker Compose configuration that sets up all required services:

- **MongoDB** (port 27017) - Database for storing posts, users, communities, etc.
- **Redis** (port 6379) - Session storage and caching
- **MinIO** (ports 9000, 9001) - S3-compatible object storage for images
- **PostgreSQL** (port 5432) - Database for the Friends service
- **Account Server** (port 8123) - User authentication via gRPC
- **Friends Server** (port 8124) - Friend management via gRPC
- **HTTP Proxy** (ports 8888, 8081) - For testing with real consoles

Start all services:

```bash
cd .docker
docker compose up -d
cd ..
```

To check the status of services:

```bash
cd .docker
docker compose ps
```

To view logs:

```bash
cd .docker
docker compose logs -f
```

### Step 4: Initialize the Database

After starting Docker services, you need to initialize some data in MongoDB:

```bash
# Configure the Miiverse service endpoint
docker exec -it juxtaposition-dev-mongo-1 mongosh account --eval '
db.servers.insertOne({
  ip: "127.0.0.1",
  port: 80,
  service_name: "miiverse",
  service_type: "service",
  game_server_id: "",
  title_ids: [],
  access_mode: "prod",
  maintenance_mode: false,
  device: 1,
  aes_key: "1234567812345678123456781234567812345678123456781234567812345678",
  client_id: "87cd32617f1985439ea608c2746e4610"
})'

# Configure the Miiverse endpoints
docker exec -it juxtaposition-dev-mongo-1 mongosh miiverse --eval '
db.endpoints.insertOne({
  status: 0,
  server_access_level: "prod",
  topics: true,
  guest_access: true,
  host: "api.olv.pretendo.cc",
  api_host: "api.olv.pretendo.cc",
  portal_host: "portal.olv.pretendo.cc",
  n3ds_host: "ctr.olv.pretendo.cc"
})'
```

## Configuration

Both services (`miiverse-api` and `juxtaposition-ui`) can be configured using environment variables or configuration files.

### Using Docker Presets (Recommended for Development)

The easiest way to configure the services is to use the Docker preset, which automatically configures everything to work with the Docker Compose setup:

```bash
# For miiverse-api
PN_MIIVERSE_API_USE_PRESETS=docker npm run dev

# For juxtaposition-ui
PN_JUXTAPOSITION_UI_USE_PRESETS=docker npm run dev
```

### Manual Configuration

If you're not using Docker or need custom configuration, create `.env` files in each service directory.

#### apps/miiverse-api/.env

```bash
PN_MIIVERSE_API_AES_KEY=1234567812345678123456781234567812345678123456781234567812345678
PN_MIIVERSE_API_CDN_URL=http://cdn.pretendo.cc/miiverse
PN_MIIVERSE_API_MONGOOSE_URI=mongodb://localhost:27017/miiverse?directConnection=true
PN_MIIVERSE_API_S3_ENDPOINT=http://localhost:9000
PN_MIIVERSE_API_S3_KEY=minioadmin
PN_MIIVERSE_API_S3_SECRET=minioadmin
PN_MIIVERSE_API_S3_BUCKET=miiverse
PN_MIIVERSE_API_S3_REGION=us-east-1
PN_MIIVERSE_API_GRPC_FRIENDS_HOST=localhost
PN_MIIVERSE_API_GRPC_FRIENDS_PORT=8124
PN_MIIVERSE_API_GRPC_FRIENDS_API_KEY=12345678123456781234567812345678
PN_MIIVERSE_API_GRPC_ACCOUNT_HOST=localhost
PN_MIIVERSE_API_GRPC_ACCOUNT_PORT=8123
PN_MIIVERSE_API_GRPC_ACCOUNT_API_KEY=12345678123456781234567812345678
PN_MIIVERSE_API_GRPC_SERVER_PORT=8125
PN_MIIVERSE_API_GRPC_SERVER_API_KEY=12345678123456781234567812345678
```

#### apps/juxtaposition-ui/.env

```bash
PN_JUXTAPOSITION_UI_HTTP_PORT=5173
PN_JUXTAPOSITION_UI_CDN_DOMAIN=https://cdn.pretendo.cc/miiverse
PN_JUXTAPOSITION_UI_WHITELIST=
PN_JUXTAPOSITION_UI_SERVER_ENVIRONMENT=prod
PN_JUXTAPOSITION_UI_DISABLE_CONSOLE_CHECKS=true
PN_JUXTAPOSITION_UI_AES_KEY=1234567812345678123456781234567812345678123456781234567812345678
PN_JUXTAPOSITION_UI_MONGOOSE_URI=mongodb://localhost:27017/miiverse?directConnection=true
PN_JUXTAPOSITION_UI_S3_ENDPOINT=http://localhost:9000
PN_JUXTAPOSITION_UI_S3_KEY=minioadmin
PN_JUXTAPOSITION_UI_S3_SECRET=minioadmin
PN_JUXTAPOSITION_UI_S3_BUCKET=miiverse
PN_JUXTAPOSITION_UI_S3_REGION=us-east-1
PN_JUXTAPOSITION_UI_GRPC_FRIENDS_HOST=localhost
PN_JUXTAPOSITION_UI_GRPC_FRIENDS_PORT=8124
PN_JUXTAPOSITION_UI_GRPC_FRIENDS_API_KEY=12345678123456781234567812345678
PN_JUXTAPOSITION_UI_GRPC_ACCOUNT_HOST=localhost
PN_JUXTAPOSITION_UI_GRPC_ACCOUNT_PORT=8123
PN_JUXTAPOSITION_UI_GRPC_ACCOUNT_API_KEY=12345678123456781234567812345678
PN_JUXTAPOSITION_UI_GRPC_MIIVERSE_HOST=localhost
PN_JUXTAPOSITION_UI_GRPC_MIIVERSE_PORT=8125
PN_JUXTAPOSITION_UI_GRPC_MIIVERSE_API_KEY=12345678123456781234567812345678
PN_JUXTAPOSITION_UI_REDIS_HOST=localhost
```

For a complete list of configuration options, check the `src/config.ts` file in each service directory.

## Running the Services

### Development Mode (with hot reload)

#### miiverse-api

```bash
cd apps/miiverse-api
PN_MIIVERSE_API_USE_PRESETS=docker npm run dev
```

This will:
- Watch for file changes
- Automatically rebuild and restart on changes
- Run on port 8080 by default

#### juxtaposition-ui

```bash
cd apps/juxtaposition-ui
PN_JUXTAPOSITION_UI_USE_PRESETS=docker npm run dev
```

This will:
- Watch for file changes
- Automatically rebuild and restart on changes
- Run on port 5173 by default

### Production Build

To build for production:

```bash
# Build miiverse-api
cd apps/miiverse-api
npm run build

# Build juxtaposition-ui
cd apps/juxtaposition-ui
npm run build
```

To run production builds:

```bash
# Run miiverse-api
cd apps/miiverse-api
PN_MIIVERSE_API_USE_PRESETS=docker npm start

# Run juxtaposition-ui
cd apps/juxtaposition-ui
PN_JUXTAPOSITION_UI_USE_PRESETS=docker npm start
```

## Testing with Console/Emulator

### Creating a Test Account

You can create a test PNID (Pretendo Network ID) using the HTTP proxy:

```bash
# Register a new account
https_proxy=http://localhost:8888 curl -k -v -X POST \
  -H "Content-Type: application/json" \
  --data '{"email":"test@example.com", "username":"TestUser", "mii_name":"TestMii", "password":"password123", "password_confirm":"password123"}' \
  https://api.pretendo.cc/v1/register/

# Login to get tokens
https_proxy=http://localhost:8888 curl -k -v -X POST \
  -H 'Content-Type: application/json' \
  --data '{"grant_type": "password", "username":"TestUser", "password":"password123"}' \
  https://api.pretendo.cc/v1/login

# Get user info (use the access_token from login response)
https_proxy=http://localhost:8888 curl -k -v -X GET \
  -H "Authorization: Bearer <access_token_here>" \
  https://api.pretendo.cc/v1/user
```

### Making Yourself a Developer Account

Developer accounts (access level 3) have additional privileges:

```bash
docker exec -it juxtaposition-dev-mongo-1 mongosh account --eval '
db.pnids.updateOne({username: "TestUser"}, {$set: {access_level: 3}})
'
```

### Testing with Cemu (Wii U Emulator)

1. Install the [Miiverse_Proxy graphics pack](./.docker/Miiverse_Proxy) in Cemu
2. Configure your account in `mlc/usr/save/system/act/80000001/account.dat`
3. Run Cemu through the proxy:

```bash
http_proxy=http://localhost:8888 https_proxy=http://localhost:8888 \
  cemu -mlc /path/to/mlc
```

### Testing with Web Browser

To access the Juxtaposition web UI through the proxy:

```bash
chromium-browser --proxy-server="http=localhost:8888;https=localhost:8888"
```

Then navigate to the configured domain (e.g., `https://juxt.pretendo.cc`)

### Proxy Web Interface

The mitmproxy web interface is available at:
- **URL**: http://localhost:8081
- **Password**: letmein

This allows you to inspect all HTTP/HTTPS traffic going through the proxy.

## Troubleshooting

### Docker Services Won't Start

**Issue**: Docker containers fail to start or are unhealthy.

**Solution**:
- Check Docker is running: `docker ps`
- Check logs: `cd .docker && docker compose logs`
- Try recreating containers: `docker compose down && docker compose up -d`
- On SELinux systems, try: `setenforce permissive`

### MongoDB Connection Issues

**Issue**: Services can't connect to MongoDB.

**Solution**:
- Verify MongoDB is running: `docker ps | grep mongo`
- Check MongoDB logs: `docker logs juxtaposition-dev-mongo-1`
- Ensure MongoDB replica set is initialized (should happen automatically via healthcheck)
- Try: `docker exec -it juxtaposition-dev-mongo-1 mongosh --eval "rs.status()"`

### Port Already in Use

**Issue**: Error like "Port 8080 is already in use"

**Solution**:
- Check what's using the port: `lsof -i :8080` (on Linux/Mac) or `netstat -ano | findstr :8080` (on Windows)
- Stop the conflicting service or change the port in configuration
- For Juxtaposition services, set the port in `.env`:
  - `PN_MIIVERSE_API_HTTP_PORT=8081` for miiverse-api
  - `PN_JUXTAPOSITION_UI_HTTP_PORT=5174` for juxtaposition-ui

### Environment Variables Not Working

**Issue**: Configuration changes don't take effect.

**Solution**:
- Ensure you're using the correct prefix: `PN_MIIVERSE_API_` or `PN_JUXTAPOSITION_UI_`
- Environment variables take precedence over `.env` files
- Restart the service after changing configuration
- Use `PN_*_USE_PRESETS=docker` to override with Docker defaults

### MinIO/S3 Connection Issues

**Issue**: Can't upload images or access S3 storage.

**Solution**:
- Verify MinIO is running: `docker ps | grep minio`
- Check MinIO web UI: http://localhost:9001 (login: minioadmin/minioadmin)
- Verify bucket exists: `docker exec -it juxtaposition-dev-minio-1 mc ls local/`
- Recreate bucket if needed: Run the minio-setup container again

### Services Start but Don't Respond

**Issue**: Services are running but HTTP requests timeout or fail.

**Solution**:
- Check service logs for errors
- Verify all dependencies (MongoDB, Redis, Account, Friends) are healthy: `cd .docker && docker compose ps`
- Ensure account and friends servers are properly configured in MongoDB (see Step 4)
- Test MongoDB connection: `docker exec -it juxtaposition-dev-mongo-1 mongosh miiverse --eval "db.posts.count()"`

### Build Errors

**Issue**: `npm install` or `npm run build` fails.

**Solution**:
- Ensure you're using Node.js 20 or higher: `node --version`
- Clear npm cache: `npm cache clean --force`
- Delete `node_modules` and `package-lock.json`, then reinstall:
  ```bash
  rm -rf node_modules package-lock.json
  npm install
  ```
- If using workspaces, try installing from the root directory

## Additional Resources

### Project Structure

```
juxtaposition/
├── .docker/              # Docker Compose configuration and setup scripts
├── apps/
│   ├── miiverse-api/     # Backend API service
│   └── juxtaposition-ui/ # Frontend web interface
├── packages/             # Shared packages
│   ├── grpc-client/      # gRPC client utilities
│   └── esbuild-plugin-*  # Custom build plugins
├── protobufs/            # Protocol Buffer definitions
├── migrations/           # Database migrations
└── README.md             # Project overview
```

### Related Repositories

- [Account Server](https://github.com/PretendoNetwork/account) - User authentication and PNID management
- [Friends Server](https://github.com/PretendoNetwork/friends) - Friend list and relationships
- [Pretendo Network](https://github.com/PretendoNetwork) - Main organization

### Useful Commands

```bash
# Lint code
npm run lint              # Check for issues
npm run lint:fix          # Auto-fix issues

# Build services
npm run build             # Build for production

# Run tests (if available)
npm test

# View service logs
cd .docker
docker compose logs -f [service_name]

# Stop all Docker services
cd .docker
docker compose down

# Remove all data and start fresh
cd .docker
docker compose down -v    # Warning: This deletes all data!
docker compose up -d
```

### Getting Help

- **Issues**: Report bugs or request features on [GitHub Issues](https://github.com/Woody1474747/juxtaposition/issues)
- **Discussions**: Ask questions on [GitHub Discussions](https://github.com/Woody1474747/juxtaposition/discussions)
- **Discord**: Join the Pretendo Network Discord server for community support

## Contributing

Once you have your development environment set up, you're ready to contribute! Please check the main README.md for contribution guidelines and the project philosophy.

### Translation

Help localize Pretendo Network by contributing translations on [Weblate](https://hosted.weblate.org/engage/pretendonetwork/).
