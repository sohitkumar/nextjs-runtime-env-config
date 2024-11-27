# Next.js Deployment with Runtime Environment Variables

Next.js applications cannot fetch environment variables at runtime after being built. This guide provides a solution using Docker and a custom `start.sh` script to load `.env` variables at runtime. This document includes necessary configurations and instructions for deployment.

---

## Instructions

### 1. Create a `Dockerfile` in your project root folder

Use the following `Dockerfile` to containerize your Next.js application. This configuration ensures that the app is built and runtime environment variables are properly handled.

```dockerfile
# Use an official Node.js runtime as the base image
FROM node:18-alpine

# Set the working directory
WORKDIR /app

# Copy package.json and package-lock.json files
COPY package*.json ./

# Install dependencies
RUN npm install --legacy-peer-deps

# Copy the entire project into the container
COPY . .

# Expose the port that Next.js runs on
EXPOSE 3000

# Copy the start.sh script into the container
COPY start.sh /start.sh

# Make the script executable
RUN chmod +x /start.sh

# Use the script to start the application
CMD ["/start.sh"]
```

---

### 2. Create a `start.sh` Script in your project root folder

The `start.sh` script loads `.env` variables at runtime and ensures the application is built before starting.

```bash
#!/bin/sh

# Ensure .env file exists and export variables for runtime
if [ -f /app/.env ]; then
  echo "Loading environment variables from .env file..."

  # Export environment variables from the .env file for runtime
  set -a  # Automatically export all variables
  source /app/.env
  set +a  # Stop automatically exporting variables

  # Debug: Verify that the environment variables are loaded correctly
  echo "Loaded environment variables:"
  env | grep -i "NEXT_PUBLIC"  # Display client-side variables for debugging
  env | grep -i "DB_"         # Display server-side variables for debugging
fi

# Build Next.js app if it's not already built
if [ ! -d "/app/.next" ]; then
  echo "Building the Next.js app..."
  npm run build  # Run the build process
fi

# Start the Next.js application
echo "Starting the Next.js app..."
npm run start
```

---

### 3. Create a `docker-compose.yml` File (it can be anywhere, its only needs env file and the image name)

The `docker-compose.yml` file defines the service and ensures the `.env` file is mounted correctly for runtime access.

```yaml
version: "3.9"
services:
  app:
    image: nextjs-ui-app
    container_name: nextjs_app
    ports:
      - "3000:3000"
    env_file:
      - .env
    volumes:
      - ./.env:/app/.env
```

---

## Notes

- Use `NEXT_PUBLIC_*` for environment variables that need to be accessible on the client side.
- Private variables (e.g., database credentials) remain secure in the server-side environment.
- The `start.sh` script ensures the app is built and runtime environment variables are exported before starting the server.

---

This guide should help you streamline your Next.js deployment process with runtime environment variable support!
