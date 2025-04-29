# Ignition Docker Container with Custom Modules

This project allows you to build a custom Ignition Gateway Docker container with pre-installed modules from a local directory. Instead of downloading modules during the build process, this container uses modules you've placed in a local directory.

This project was inspired by and modifies [thirdgen88/ignition-derived-example](https://github.com/thirdgen88/ignition-derived-example), adapting it to support local module installation rather than downloading modules during the build process.

## Project Structure

```
.
├── docker-compose.yml         # Docker Compose configuration
├── gw-build/                  # Build context directory
│   ├── base.gwbk              # Base Ignition gateway backup
│   ├── Dockerfile             # Docker build instructions
│   ├── docker-entrypoint-shim.sh  # Entrypoint script
│   ├── modules/               # Directory for your .modl files
│   │   └── *.modl             # Your module files go here
│   ├── register-module.sh     # Module registration script
│   └── register-password.sh   # Password configuration script
└── gw-secrets/                # Secrets directory
    └── GATEWAY_ADMIN_PASSWORD # Admin password file
```

## How to Use

### 1. Prepare Your Modules

Place your Ignition module files (*.modl) in the `gw-build/modules/` directory.

### 2. Set Up Admin Password

Create a password file:

```bash
mkdir -p gw-secrets
echo "your_admin_password" > gw-secrets/GATEWAY_ADMIN_PASSWORD
```

### 3. Build and Run

Build and start the container using Docker Compose:

```bash
docker-compose up --build
```

By default, the Ignition Gateway will be accessible at http://localhost:8088.

## Configuration Options

### Environment Variables

Edit the `docker-compose.yml` file to customize:

- `IGNITION_VERSION`: The version of Ignition to use (default: 8.1.47)
- `TZ`: Time zone (default: America/Chicago)
- `GATEWAY_MODULES_ENABLED`: Comma-separated list of modules to enable

### Persistence

Uncomment the volumes section in `docker-compose.yml` to enable data persistence:

```yaml
volumes:
  - gateway-data:/usr/local/bin/ignition/data
```

## Customization

### Admin Username

To change the admin username, modify the `GATEWAY_ADMIN_USERNAME` argument in the Dockerfile or set the environment variable in docker-compose.yml.

### Custom Entrypoint Logic

The `docker-entrypoint-shim.sh` script runs before the Ignition Gateway starts. You can modify this script to add custom initialization logic.

## Troubleshooting

### Module Registration Issues

If you encounter issues with module registration:

1. Check the module files for corruption
2. Try processing problematic modules individually
3. Check the logs for specific error messages

### Container Fails to Start

If the container fails to start:

1. Verify that the `base.gwbk` file exists and is valid
2. Ensure all module files are properly formatted
3. Check that the admin password file exists and is readable

## Advanced Usage

### Building a Custom Image

To create a reusable Docker image:

1. Uncomment the `image` line in `docker-compose.yml`
2. Run `docker-compose build`
3. Push the image to your registry with `docker push huntermatuse:base-${IGNITION_VERSION:-8.1.47}`