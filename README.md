# In-Depth Docker Guide: Frontend and Backend

## 1. Docker Core Concepts

### Images
An image is a read-only template containing a set of instructions for creating a Docker container. It's like a snapshot of a system, including the OS, software, and code.

Example: Building a custom Node.js image
```dockerfile
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "server.js"]
```

### Containers
A container is a runnable instance of an image. It's isolated from the host system and other containers, ensuring consistency across different environments.

Example: Running a Node.js container
```bash
docker run -d -p 3000:3000 --name my-node-app node-image
```

### Dockerfile
A Dockerfile is a script containing commands to assemble an image. Each instruction creates a new layer in the image.

Key Dockerfile instructions:
- `FROM`: Specifies the base image
- `WORKDIR`: Sets the working directory
- `COPY`: Copies files from host to container
- `RUN`: Executes commands during build
- `CMD`: Specifies the command to run when the container starts

## 2. Docker Networking

Docker creates isolated network environments for containers. Understanding networking is crucial for connecting frontend and backend services.

### Bridge Network
The default network for containers. Containers on the same bridge network can communicate using container names as hostnames.

Example: Creating a custom bridge network
```bash
docker network create my-network
docker run --network my-network --name backend backend-image
docker run --network my-network --name frontend frontend-image
```

### Port Mapping
Exposes container ports to the host system.

Example: Mapping container port 3000 to host port 8080
```bash
docker run -p 8080:3000 my-app
```

## 3. Docker Volumes

Volumes provide persistent storage for containers, essential for databases and file uploads.

Example: Creating and using a volume
```bash
docker volume create my-data
docker run -v my-data:/app/data my-app
```

## 4. Docker Compose

Docker Compose simplifies multi-container application management with a YAML file.

Example: Full-stack application with frontend, backend, and database
```yaml
version: '3'
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend
  backend:
    build: ./backend
    ports:
      - "5000:5000"
    depends_on:
      - db
    environment:
      - DATABASE_URL=mongodb://db:27017/myapp
  db:
    image: mongo:4.4
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

## 5. Practical Examples

### Frontend (React) with Hot Reloading
```dockerfile
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "start"]
```

Run with volume for live code updates:
```bash
docker run -v $(pwd):/app -p 3000:3000 react-app
```

### Backend (Express) with Database Connection
```dockerfile
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "server.js"]
```

Docker Compose setup:
```yaml
version: '3'
services:
  backend:
    build: .
    ports:
      - "5000:5000"
    environment:
      - MONGODB_URI=mongodb://db:27017/myapp
  db:
    image: mongo:4.4
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

## 6. Best Practices

1. Use multi-stage builds to reduce image size
2. Leverage layer caching for faster builds
3. Use .dockerignore to exclude unnecessary files
4. Never store secrets in images; use environment variables
5. Regularly update base images for security
6. Use health checks to ensure service availability

Example multi-stage build for a Node.js app:
```dockerfile
# Build stage
FROM node:14 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:14-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY package*.json ./
RUN npm install --only=production
CMD ["node", "dist/server.js"]
```

This setup results in a smaller production image without build dependencies.
