# Report lab02

Author: Quentin Surdez

## Time Estimation

| Task | Estimated Time | Effective Time | Remarks |
| ------- | -------------- | ------------- | ------------- |
| Git | 5mn | 5mn |  |
| Docker | 30mn | 1h | It was a bit more challenging than estimated. I witnessed my own ignorance in terms of good practices when it comes to Dockerfile, so very interesting! |
| Docker Compose | 30mn | 15mn | Quite easy compared to what I have done in the past |
| Report | 1h | 1h | |


## Questions

### Choices justification for Dockerfiles

#### 1. Backend

1. Multi-stage Architecture:
    - I chose a multi-stage approach to reduce the size of the image
    - The `builder` stage installs dependencies with Poetry
    - The `runtime` stage contains only the code and virtual environment necessary for execution

2. Alpine Usage:
    - Alpine images are much lighter than usual Python images
    - This is a sure way to improve the size of the final image

3. Docker Cache Optimization:
    - Dependencies are installed before copying the source code
    - This allows reusing the Docker cache when only the source code changes, speeding up successive builds.

4. Poetry Environment Variables:
    - This allows to control the behavior of the `builder` stage at a granular level
    - We avoid interactive prompts 
    - We tell Poetry to create a virtual environment if necessary


#### 2. Frontend

1. Multi-stage Architecture:
    - The `build-stage` compiles the Vue.js application with Node.js
    - The `production-stage` uses nginx to serve the generated static files.
    - This approach significantly reduces the image size by eliminating Node.js and all build tools

2. Alpine Usage:
      - For the same reasons as the backend, we use the alpine images for their lightweight nature

3. Dependency Optimization:
      - `COPY package*.json ./` then `RUN npm ci` before copying the rest of the source code
      - `npm ci` is used instead of `npm install` to ensure deterministic installations based on package-lock.json
      - This strategy optimizes Docker cache

#### 3. `.dockerignore`

The whitelist strategy has been applied to both the frontend and backend `.dockerignore`. Only the necessary directories have been authorized as to optimize the passing of the context to Docker.

### Choices justification for compose and database

1. Explicitly Named Services:
      - Clear names (`frontend`, `backend`, `database`) to facilitate understanding
      - The `container_name` ensures consistent container names

2. Dependency Management:
      - `depends_on` defines the start-up order of services
      - The frontend depends on the backend, which depends on the database
      - This doesn't guarentee that services are ready, but defines a logical order 

3. Networking:
      - A dedicated `app-network` isolates the application containers
      - The `bridge` type is quite suitable for this use-case
      - Services can communicate using their names as hostnames

4. Database Configuration:
      - `postgres:alpine` image to reduce size
      - Using the credentials specified in the requirements
      - Using port 5432 to expose the database so that tools can access it, such as DataGrip or DBeaver

5. Data Persistance: 
      - Named volume `postgres-data` to store PostgreSQL data
      - The volume is mounted on `/var/lib/postgresql/data` where PostgreSQL data is stored
      - This configuration ensures that data persists even after container removal

6. Restart Policy:
      - `restart: unless-stopped` ensures containers restart automatically irrespective of the exit code
      - This policy doesn't restart containers that have been manually removed

7. Environment Variables:
      - Configuration of the database URL for the backend

8. Port Mapping:
      - Port mappings conform to requirements (80 for frontend, 8080 for backend, 5432 for database)
      - "host:container" format