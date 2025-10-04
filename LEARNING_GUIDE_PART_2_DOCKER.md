# Docker Guide for Learning Endpoints API

This guide explains how to use Docker with the Learning Endpoints API project, including building Docker images, running containers, and best practices.

## Overview

The Learning Endpoints API is a TypeScript-based Express.js application that provides various API endpoints for fetching data from external services. This Docker setup allows you to containerize the application for consistent deployment across different environments.

## Docker Setup

### Files Created

1. **Dockerfile** - Multi-stage Docker configuration
2. **.dockerignore** - Excludes unnecessary files from build context
3. **DOCKER.md** - This documentation file

## Dockerfile Explanation

Our Dockerfile uses a **multi-stage build** approach for optimal image size and security:

### Stage 1: Builder
```dockerfile
FROM node:20-alpine AS builder
```
- Uses Node.js 20 Alpine Linux (lightweight base image)
- Installs all dependencies (including dev dependencies)
- Compiles TypeScript to JavaScript
- Outputs built files to `dist/` directory


#### Existing Images
*In 99% of cases we start off with an existing image to base our image off of. We don't need to reinvent the wheel when someone has already done the work for us.*

*Most images are just a basic linux (usually ubuntu) build with a couple bits installed on top.*

*This one we're using is exactly that. A basic ubuntu computer with node and a few other usful bits*

#### Repos
*The FROM command downloads/pulls all images from the docker hub repo. This is the default repo by default.*

*https://hub.docker.com/search - Check out this website and see what images are availible.*

*Another good read is this: https://learn.g2.com/best-docker-containers-repository which explains how there are countless repos availible to us. The article also shows popular docker images being used, which should give you an idea of how people use Docker.*




### Stage 2: Production
```dockerfile
FROM node:20-alpine AS production
```
- Creates a fresh, minimal production image
- Installs only production dependencies
- Copies compiled JavaScript from builder stage
- Sets up non-root user for security
- Includes health check for container monitoring

### Key Features

- **Multi-stage build**: Reduces final image size by excluding build tools
- **Security**: Runs as non-root user (`nextjs:nodejs`)
- **Health check**: Monitors application health
- **Layer optimization**: Copies package files first for better caching
- **Alpine Linux**: Minimal base image for smaller footprint

## Building the Docker Image

### Basic Build
```bash
docker build -t learning-endpoints .
```

### Build with Custom Tag
```bash
docker build -t learning-endpoints:v1.0.0 .
```

### Build with Custom Name
```bash
docker build -t my-api-app .
```

## Running the Container

### Basic Run
```bash
docker run -p 3000:9999 learning-endpoints
```
This maps container port 9999 to host port 3000.

### Run with Environment Variables
```bash
docker run -p 3000:9999 -e PORT=9999 -e NODE_ENV=production learning-endpoints
```

### Run in Detached Mode
```bash
docker run -d -p 3000:9999 --name my-api learning-endpoints
```

### Run with Volume Mount (for development)
```bash
docker run -p 3000:9999 -v "$(pwd)/.env:/app/.env:ro" learning-endpoints
```

## Environment Variables

The application supports these environment variables:

- `PORT` - Server port (default: 9999)
- `NODE_ENV` - Environment (development/production)
- Additional API keys for external services (check your route implementations)

## Docker Compose (Optional)

Create a `docker-compose.yml` for easier management:

```yaml
version: '3.8'
services:
  api:
    build: .
    ports:
      - "3000:9999"
    environment:
      - PORT=9999
      - NODE_ENV=production
    restart: unless-stopped
```

Run with:
```bash
docker-compose up -d
```

## Testing the Container

After starting the container, test the API:

### Check Health
```bash
curl http://localhost:3000/
```

### Test Specific Endpoints
```bash
# API documentation
curl http://localhost:3000/

# Weather endpoint (example)
curl http://localhost:3000/api/weather/london

# Quotes endpoint (example)
curl http://localhost:3000/api/quotes
```

## Docker Commands Reference

### Image Management
```bash
# List images
docker images

# Remove image
docker rmi learning-endpoints

# Build without cache
docker build --no-cache -t learning-endpoints .
```

### Container Management
```bash
# List running containers
docker ps

# List all containers
docker ps -a

# Stop container
docker stop my-api

# Remove container
docker rm my-api

# View logs
docker logs my-api

# Execute commands in running container
docker exec -it my-api sh
```

### Cleanup
```bash
# Remove unused images
docker image prune

# Remove unused containers
docker container prune

# Remove all unused resources
docker system prune -a
```

## Production Considerations

### Environment Variables
- Create a `.env` file for production secrets
- Never include sensitive data in the image
- Use Docker secrets or environment variable injection

### Security
- The container runs as non-root user for security
- Only necessary files are included (see `.dockerignore`)
- Production dependencies only in final image

### Monitoring
- Health check is included for container orchestration
- Monitor logs with `docker logs <container-name>`
- Use monitoring tools for production deployments

### Resource Limits
Set resource limits in production:
```bash
docker run -p 3000:9999 --memory=512m --cpus=1 learning-endpoints
```

## Troubleshooting

### Build Issues
- Ensure all dependencies are listed in `package.json`
- Check TypeScript compilation succeeds locally
- Verify `.dockerignore` isn't excluding needed files

### Runtime Issues
- Check container logs: `docker logs <container-name>`
- Verify port mapping matches application port
- Ensure environment variables are properly set

### Connection Issues
- Confirm port mapping: `-p host_port:container_port`
- Check firewall settings
- Verify the application starts correctly in logs


## Example Workflow

```bash
# 1. Build the image
docker build -t learning-endpoints:latest .

# 2. Run the container
docker run -d -p 3000:9999 --name api-server learning-endpoints:latest

# 3. Test the API
curl http://localhost:3000/

# 4. View logs
docker logs api-server

# 5. Stop and cleanup
docker stop api-server
docker rm api-server
```

This Docker setup provides a robust, secure, and efficient way to containerize your Learning Endpoints API for development, testing, and production environments.

---

# Docker Learning Assignments

Complete these assignments to gain hands-on experience with Docker and prepare for integrating databases into your project.

## Week 1: Docker Fundamentals

### Assignment 1: Build and Run Your API Container
**Objective**: Create a Docker image from this project and verify it works correctly.

**Tasks**:
1. **Build the Docker image** using the provided Dockerfile:
   ```bash
   docker build -t learning-endpoints .
   ```
2. **Run the container** with proper port mapping:
   ```bash
   docker run -p 3000:9999 --name my-api learning-endpoints
   ```
3. **Test all endpoints** to ensure they work:
   - Visit `http://localhost:3000/` for API documentation
   - Test weather endpoint: `curl http://localhost:3000/api/weather/london`
   - Test quotes endpoint: `curl http://localhost:3000/api/quotes`
   - Test users endpoint: `curl http://localhost:3000/api/users/1`
   - Test crypto endpoint: `curl http://localhost:3000/api/crypto/bitcoin`

**Success Criteria**: All endpoints return valid responses and the container runs without errors.

### Assignment 2.1: Understand Dockerfile Commands
**Objective**: Research and understand each command used in the project's Dockerfile.

**Tasks**:
1. **Study each Dockerfile command** for:
   - `FROM node:20-alpine AS builder`
   - `WORKDIR /app`
   - `COPY package*.json ./`
   - `RUN npm ci --only=production=false`
   - `COPY . .`
   - `RUN npm run build`
   - `FROM node:20-alpine AS production`
   - `RUN addgroup -g 1001 -S nodejs && adduser -S nextjs -u 1001`
   - `COPY --from=builder /app/dist ./dist`
   - `USER nextjs`
   - `EXPOSE 9999`
   - `HEALTHCHECK`
   - `CMD ["node", "dist/app.js"]`

2. **Research multi-stage builds**: Understand why we use two stages and the benefits.

**Deliverable**: Explain to Paddy each command and the overall build strategy.

### Assignment 2.2: Master Common Docker Commands
**Objective**: Learn essential Docker commands for day-to-day container management.

**Tasks**:
1. **Practice these commands** and understand their purpose:
   ```bash
   # Image commands
   docker images
   docker build -t <name> .
   docker rmi <image>
   docker pull <image>
   docker push <image>
   
   # Container commands
   docker run <options> <image>
   docker ps
   docker ps -a
   docker stop <container>
   docker start <container>
   docker rm <container>
   docker logs <container>
   docker exec -it <container> <command>
   
   # System commands
   docker system df
   docker system prune
   docker volume ls
   docker network ls
   ```

2. OPTIONAL: **Create a cheat sheet** with the most important commands and their common options.

**Deliverable**: Personal Docker command reference sheet with examples.

## Week 2: Database Integration with Docker

### Assignment 3.1: Research MongoDB Docker Images
**Objective**: Find and choose an appropriate MongoDB image on Docker Hub that your node project will interact with.

**Tasks**:
1. **Browse Docker Hub** at https://hub.docker.com/_/mongo
2. **Compare different MongoDB tags**:
   - `mongo:latest`
   - `mongo:7.0`
   - `mongo:7.0-jammy`
   - `mongo:6.0-focal`
   - etc
3. **Read the documentation** for the official MongoDB image
4. **Understand environment variables** like:
   - `MONGO_INITDB_ROOT_USERNAME`
   - `MONGO_INITDB_ROOT_PASSWORD`
   - `MONGO_INITDB_DATABASE`

### Assignment 3.2: Run MongoDB Container
**Objective**: Successfully run a MongoDB instance using Docker.

**Tasks**:
1. **Pull the MongoDB image**
2. **Run MongoDB container** with proper configuration
3. **Verify the container is running**

**Success Criteria**: MongoDB container runs successfully and accepts connections on port 27017.

### Assignment 3.3: Practice MongoDB Commands
**Objective**: Read the MongoDB docs and learn the CLI operations you can call on a running container.

**Tasks**:
1. **Connect to MongoDB** using the MongoDB shell:
   ```bash
   docker exec -it mongodb mongosh -u admin -p password123
   ```
2. **Practice basic CRUD operations from the terminal**:

  Rough Example:
   ```javascript
   // Create database and collection
   db.createCollection("users")
   
   // Create (Insert) documents
   db.users.insertOne({name: "John Doe", email: "john@example.com", age: 30})
   db.users.insertMany([
     {name: "Jane Smith", email: "jane@example.com", age: 25},
     {name: "Bob Johnson", email: "bob@example.com", age: 35}
   ])
   
   // Read (Find) documents
   db.users.find()
   db.users.findOne({name: "John Doe"})
   db.users.find({age: {$gte: 30}})
   
   // Update documents
   db.users.updateOne({name: "John Doe"}, {$set: {age: 31}})
   db.users.updateMany({age: {$lt: 30}}, {$set: {status: "young"}})
   
   // Delete documents
   db.users.deleteOne({name: "Bob Johnson"})
   db.users.deleteMany({age: {$lt: 25}})
   ```

**Deliverable**: Show Paddy your MongoDB skills at your next session.

## Week 3: Full Integration

### Assignment 1: Add Database HTTP Functions with Mongoose
**Objective**: Extend your node APP with Mongoose to interact with your running MongoDB database container.

Look at the mongo + mongoose docs to understand how this can be done. 

**Success Criteria**: Your API can perform full CRUD operations on MongoDB data using Mongoose with proper schema validation and error handling.

Your CRUD operations can be whatever you want. Users, products, books, todos, whatever you feel like! 

### Assignment 2: Multi-Container Application Flow
**Objective**: Run both your Node project and MongoDB as separate containers and demonstrate the complete flow. Do this by hitting your node projects endpoints, the node project should then call out and hit the running mongo instance.

**Tasks**:
**Deliverable**: Working multi-container setup with flow diagram documentation.

### Assignment 3: Docker Compose Orchestration
**Objective**: Create a Docker Compose file to manage both containers together.

Remember to document your progress, take screenshots of working applications, and ask questions when you encounter issues!