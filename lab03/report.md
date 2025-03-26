# Report lab03

Author: Quentin Surdez

Link to MR [here](https://gitlab.com/Sinaf19/dop-python-quentin/-/merge_requests/3)


# Estimation

| Task | Estimated Time | Effective Time | Remarks |
| ------- | -------------- | ------------- | ------------- |
| Makefile | 15mn | 5mn | Was easier than planned |
| Traefik | 30mn | 1h | It was a bit more challenging than estimated. I witnessed my own ignorance in terms of good practices when it comes to Dockerfile, so very interesting! |
| Connecting backend to database | 1h | 1h30min | Encountered some issues with path prefix stripping that took time to debug |
| Frontend implementation | 1h | 1h15min | Integrating SQLAlchemy was straightforward, but configuring environment variables took longer than expected |
| Docker Registry Setup | 15min | 10min | Was easier than planned
| Report | 1h | 1h | |
| Total | 5h | 5h | Good estimate |

## Makefile creation
For the Makefile implementation, I followed these principles:

1. Simplicity and Readability:
  - Used clear target names that directly reflect their purpose (install, dev-backend, etc.)
  - Added comments to explain non-obvious commands
  - Grouped related commands under single targets

2. Dependency Management:
  - Created a hierarchical structure where targets depend on prerequisites

3. Environment Handling:
  - Included loading of environment variables from .env files where needed
  - Made dev targets load appropriate environment configurations

## Traefik configuration
To implement Traefik as a reverse proxy, I took the following approach:

1. Service Configuration:
  - Added Traefik as a service in the compose.yml file
  - Configured it to listen on port 80
  - Set up labels for dashboard access (for development purposes)


2. Routing Rules:
  - Created path-based routing using PathPrefix rules:
    - Requests to / are routed to the frontend
    - Requests to /api are routed to the backend
  - Used the StripPrefix middleware for the backend to remove the /api prefix before forwarding requests



## Backend database connection and twelve-factor app
For connecting the backend to the database and applying twelve-factor app principles, I made the following modifications:

1. Environment-Based Configuration:

  - Created a .env file at the root of the project to store all environment variables
  - Updated the database.py file to read connection parameters from environment variables
  - Used default values as fallbacks for development


2. Database Connection:

  - Installed SQLAlchemy and psycopg2 using Poetry
  - Implemented the database connection with a connection string built from environment variables
  - Created models for data representation and schemas for validation


3. Healthcheck Implementation:

  - Added a healthcheck to the database service in compose.yml
  - Configured the backend to depend on the database with condition service_healthy
  - This ensures the backend only starts when the database is actually ready to accept connections




