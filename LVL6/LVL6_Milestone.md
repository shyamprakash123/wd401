# Environment Variable Configuration for Node.js Application Deployment

## Purpose:

Environment variables are used to provide configuration parameters to Node.js applications. These variables can differ between development,
staging and production environments and help manage sensitive information securely.

## Instructions:

1.  **Create a `.env` file**: This file will store environment-specific configurations. Create it in the root directory of your Node.js application.

```env
####### production #######
NODE_ENV=production
PROD_USERNAME=simplenode
PROD_PASSWORD=12345678
PROD_DATABASE=todo
PROD_HOST=db
PROD_DIALECT=postgres

####### development #######
NODE_ENV=development
DEV_USERNAME=simplenode
DEV_PASSWORD=12345678
DEV_DATABASE=todo
DEV_HOST=db
DEV_DIALECT=postgres
```

2.  **Explanation of Environment Variables:**

- **`NODE_ENV`**: Indicates the environment in which the application is running (`production` or `development`).
- **`PROD_USERNAME`**: Username for the production database.
- **`PROD_PASSWORD`**: Password for the production database.
- **`PROD_DATABASE`**: Name of the production database.
- **`PROD_HOST`**: Hostname or IP address of the production database server.
- **`PROD_DIALECT`**: Dialect used by the production database (e.g., `postgres`, `mysql`).
- **`DEV_USERNAME`**: Username for the development database.
- **`DEV_PASSWORD`**: Password for the development database.
- **`DEV_DATABASE`**: Name of the development database.
- **`DEV_HOST`**: Hostname or IP address of the development database server.
- **`DEV_DIALECT`**: Dialect used by the development database (e.g., `postgres`, `mysql`).

3.  **Load Environment Variables**:

    - Use a package like `dotenv` to load environment variables from the `.env` file into your Node.js application.
    - Install `dotenv` package:

```bash
npm install dotenv
```

- Require and configure `dotenv` in your application's entry point (e.g., `index.js`):

```bash
require('dotenv').config();
```

**Access Environment Variables**:

- Access environment variables in your application using `process.env`.
- For example:

```js
const port = process.env.PORT || 3000;
```

## Why Managing Sensitive Information with Environment Variables Matters

Sensitive information, such as API keys, database credentials, or authentication tokens, needs to be handled securely when deploying applications. Here's why managing this information using environment variables is crucial:

### Security

Storing sensitive data directly in code or configuration files poses security risks, especially if the code is publicly accessible. Environment variables provide a safer alternative by keeping sensitive data separate from the codebase, reducing the risk of exposure to unauthorized parties.

### Access Control

Environment variables can be managed separately from the codebase, allowing administrators to restrict access to sensitive information only to authorized personnel. This helps enforce the principle of least privilege, ensuring that sensitive data is accessible only to those who need it.

### Ease of Configuration

Using environment variables makes it easier to configure applications for different environments (e.g., development, staging, production) without modifying the source code. This flexibility streamlines the deployment process and reduces the likelihood of errors caused by hardcoding environment-specific configurations.

### Scalability

As applications grow and evolve, the number of sensitive configuration parameters may increase. Managing these configurations using environment variables simplifies the scalability process, as it's easier to add or update variables as needed without making extensive changes to the codebase.

### Compliance

In regulated industries or environments where compliance with security standards is required, managing sensitive information using environment variables helps demonstrate adherence to security best practices. It allows organizations to maintain audit trails and ensure that sensitive data is handled in accordance with regulatory requirements.

## PM2 Clustering for Production

PM2 is a process manager for Node.js applications that provides powerful features for process management, monitoring, and deployment. One of its key features is clustering, which allows a Node.js application to utilize multiple CPU cores efficiently, improving performance and scalability, especially in production environments where high traffic and resource utilization are expected.

### What is Clustering?

Clustering is a technique used to distribute the workload of a single application across multiple processes, known as workers. Each worker runs as a separate instance of the application, allowing the application to handle more requests concurrently by utilizing the processing power of multiple CPU cores.

### Benefits of PM2 Clustering for Production

1.  **Improved Performance**: By distributing the workload across multiple CPU cores, PM2 clustering improves the application's performance, enabling it to handle a higher volume of requests simultaneously.
2.  **Enhanced Scalability**: PM2 clustering allows the application to scale horizontally by adding more worker processes as needed, providing seamless scalability to accommodate increasing traffic and workload demands.
3.  **Fault Tolerance**: In production environments, having multiple worker processes ensures fault tolerance. If one worker process crashes or becomes unresponsive, PM2 automatically restarts it, preventing downtime and maintaining the availability of the application.
4.  **Resource Utilization**: PM2 clustering optimizes resource utilization by efficiently utilizing all available CPU cores, ensuring that the server's hardware resources are fully utilized, leading to better overall system performance.

### How to Enable PM2 Clustering for Production

To enable clustering for a Node.js application using PM2 in a production environment, follow these steps:

1.  **Install PM2**: If you haven't already installed PM2, you can install it globally using npm:

    bash code

    `npm install pm2 -g`

2.  **Start the Application in Cluster Mode**: Use the `pm2 start` command to start the application in cluster mode. Specify the number of instances (`-i`) or let PM2 manage the instances automatically (`max`).

    bash code

    `pm2 start app.js -i max`

3.  **Monitor the Application**: PM2 provides a built-in monitoring dashboard (`pm2 monit`) and command-line interface (`pm2 list`) to monitor the status and performance of the application and its worker processes.

![alt text](<Screenshot 2024-02-28 214113.png>)

4.  **Configure Logging**: Configure logging to capture application logs and errors. PM2 provides options to specify log file paths (`--output` and `--error`) where logs are written.

    bash code

    `pm2 start app.js -i max --output /path/to/out.log --error /path/to/err.log`

`/app/pm2.log`

```log
0|index    | server started on port 3000
0|index    | server started on port 3000
0|index    | server started on port 3000
0|index    | server started on port 3000
0|index    | server started on port 3000
0|index    | server started on port 3000
0|index    | server started on port 3000
0|index    | server started on port 3000
0|index    | server started on port 3000
0|index    | server started on port 3000
0|index    | server started on port 3000
0|index    | server started on port 3000
0|index    | server started on port 3000
0|index    | server started on port 3000
0|index    | server started on port 3000
```

`/app/pm2_error.log`

```txt
0|index    | Wed, 28 Feb 2024 14:32:51 GMT express-session deprecated undefined saveUninitialized option; provide saveUninitialized option at app.js:34:3
0|index    | Wed, 28 Feb 2024 14:32:51 GMT express-session deprecated undefined resave option; provide resave option at app.js:34:3
0|index    | Wed, 28 Feb 2024 14:32:51 GMT express-session deprecated undefined saveUninitialized option; provide saveUninitialized option at app.js:34:3
0|index    | Wed, 28 Feb 2024 14:32:51 GMT express-session deprecated undefined resave option; provide resave option at app.js:34:3
0|index    | Wed, 28 Feb 2024 14:32:51 GMT express-session deprecated undefined saveUninitialized option; provide saveUninitialized option at app.js:34:3
0|index    | Wed, 28 Feb 2024 14:32:51 GMT express-session deprecated undefined resave option; provide resave option at app.js:34:3
0|index    | Wed, 28 Feb 2024 14:32:51 GMT express-session deprecated undefined saveUninitialized option; provide saveUninitialized option at app.js:34:3
0|index    | Wed, 28 Feb 2024 14:32:51 GMT express-session deprecated undefined resave option; provide resave option at app.js:34:3
0|index    | Wed, 28 Feb 2024 14:32:51 GMT express-session deprecated undefined saveUninitialized option; provide saveUninitialized option at app.js:34:3
0|index    | Wed, 28 Feb 2024 14:32:51 GMT express-session deprecated undefined resave option; provide resave option at app.js:34:3
0|index    | Wed, 28 Feb 2024 14:32:51 GMT express-session deprecated undefined saveUninitialized option; provide saveUninitialized option at app.js:34:3
0|index    | Wed, 28 Feb 2024 14:32:51 GMT express-session deprecated undefined resave option; provide resave option at app.js:34:3
0|index    | Wed, 28 Feb 2024 14:32:51 GMT express-session deprecated undefined saveUninitialized option; provide saveUninitialized option at app.js:34:3
0|index    | Wed, 28 Feb 2024 14:32:51 GMT express-session deprecated undefined resave option; provide resave option at app.js:34:3
0|index    | Wed, 28 Feb 2024 14:32:51 GMT express-session deprecated undefined saveUninitialized option; provide saveUninitialized option at app.js:34:3
```

![alt text](<Screenshot 2024-02-28 214223.png>)

5.  **Ensure Proper Resource Allocation**: Configure PM2 to allocate resources appropriately based on the server's hardware specifications and workload requirements. Adjust the number of instances and other parameters accordingly to optimize performance.
6.  **Handle Graceful Shutdowns**: Implement graceful shutdown procedures in the application to ensure that worker processes can cleanly exit when necessary, such as during application updates or server maintenance.

## Detailed Explanation of PM2 Cluster Mode Deployment with Dockerfile.

This Dockerfile is designed to facilitate the deployment of a Node.js application using PM2 in both production and development environments. It follows a multi-stage build approach to optimize the Docker image size and includes separate stages for production and development environments.

`Docker Desktop`

### Stage: base

Dockerfile code

```Dockerfile
FROM --platform=$BUILDPLATFORM node:lts-alpine as base
WORKDIR /app
COPY package.json /
EXPOSE 3000
```

- This stage sets up the base image using the `node:lts-alpine` image, which is based on the Alpine Linux distribution and includes Node.js.
- It sets the working directory to `/app` within the container.
- The `COPY` command copies the `package.json` file from the local directory to the `/` directory in the container.
- The `EXPOSE` command exposes port `3000`, indicating that the application running inside the container will listen on this port.

### Stage: production

Dockerfile code

```Dockerfile
FROM base as production
ENV NODE_ENV=production
RUN npm install
RUN npm install pm2 -g
COPY . /app
CMD ["pm2-runtime", "start", "index.js", "--instances", "max", "--output", "/app/pm2.log", "--error", "/app/pm2_error.log"]
```

- This stage extends the `base` stage and sets it as the `production` stage.
- It sets the `NODE_ENV` environment variable to `production` to indicate the production environment.
- The `RUN npm install` command installs the dependencies specified in the `package.json` file.
- The `RUN npm install pm2 -g` command installs PM2 globally using npm.
- The `COPY` command copies the application files from the local directory to the `/app` directory in the container.
- The `CMD` command specifies the command to start the application using PM2 in cluster mode (`pm2-runtime start index.js --instances max --output /app/pm2.log --error /app/pm2_error.log`). This command starts the application, utilizing all available CPU cores (`--instances max`), and directs the output and error logs to `/app/pm2.log` and `/app/pm2_error.log`, respectively.

### Stage: dev

Dockerfile code

```Dockerfile
FROM base as dev
ENV NODE_ENV=development
RUN npm install -g npm@10.4.0
RUN npm install -g nodemon && npm install
COPY . /app
CMD npm run start
```

- This stage extends the `base` stage and sets it as the `dev` stage.
- It sets the `NODE_ENV` environment variable to `development` to indicate the development environment.
- The `RUN npm install -g npm@10.4.0` command installs a specific version of npm globally.
- The `RUN npm install -g nodemon && npm install` command installs nodemon globally and other dependencies specified in the `package.json` file.
- The `COPY` command copies the application files from the local directory to the `/app` directory in the container.
- The `CMD` command specifies the command to start the application in development mode (`npm run start`), which typically involves using nodemon for automatic reloading during development.

`Docker Logs`

![alt text](<Screenshot 2024-02-28 214456.png>)

### Summary

- This Dockerfile provides a flexible and efficient way to deploy a Node.js application using PM2 in both production and development environments.
- It optimizes the Docker image size by utilizing multi-stage builds and ensures proper dependency installation and application startup commands for each environment.
- For production, it sets up PM2 to run the application in cluster mode and manages logs using the `--output` and `--error` options.
- For development, it provides a convenient setup for using nodemon for automatic reloading during code changes.

### Docker Compose Configuration Explanation

This Docker Compose configuration is used to define and orchestrate two services, `app` and `db`, for a Node.js application and a PostgreSQL database, respectively. Let's break down each section:

#### Version and Services

docker-compose.yaml code

```yml
version: "3.8"
services:
```

- Specifies the version of Docker Compose being used (`3.8` in this case).
- Defines a list of services that compose the application.

#### App Service

docker-compose.yaml code

```yml
app:
  build:
    context: .
    target: dev
  image: todo-app:development
  volumes:
    - .:/app
  ports:
    - "3000:3000"
  env_file:
    - .env
  depends_on:
    - db
```

- Defines the `app` service, which represents the Node.js application.
- Specifies how the application should be built and run:
  - `build`: Specifies the build configuration, with the context set to the current directory (`.`) and the target set to `dev`.
  - `image`: Specifies the name of the Docker image to be built (`todo-app:development`).
  - `volumes`: Mounts the current directory (`.`) as a volume inside the container at `/app`, allowing live code changes without rebuilding the image.
  - `ports`: Maps port `3000` on the host to port `3000` on the container, allowing access to the application from outside.
  - `env_file`: Specifies the path to an environment file (`.env`) containing environment variables for the application.
  - `depends_on`: Specifies that the `app` service depends on the `db` service, ensuring that the database is started before the application.

#### Database Service

docker-compose.yaml code

```yml
db:
  image: postgres:15
  volumes:
    - pg-dev-data:/var/lib/postgresql/data
  env_file:
    - .env
  environment:
    POSTGRES_USER: $DEV_USERNAME
    POSTGRES_PASSWORD: $DEV_PASSWORD
    POSTGRES_DB: $DEV_DATABASE
```

- Defines the `db` service, representing the PostgreSQL database.
- Specifies the PostgreSQL image to use (`postgres:15`).
- Defines volumes to persist database data (`pg-dev-data`).
- Specifies environment variables for the database:
  - `POSTGRES_USER`, `POSTGRES_PASSWORD`, and `POSTGRES_DB` are loaded from the `.env` file and provided to the PostgreSQL container.

#### Volumes

docker-compose.yaml code

```yml
volumes:
  pg-dev-data:
```

- Defines a named volume (`pg-dev-data`) to persist data for the PostgreSQL database.

`docker-compose-prod.yml` file:

This `docker-compose-prod.yml` file defines two services, `app` and `db`, for a Node.js application and a PostgreSQL database, respectively, configured for production deployment:

- The `app` service:

  - Builds the application using the `production` target.
  - Sets the image name to `todo-app:production`.
  - Maps port `3010` on the host to port `3000` on the container.
  - Loads environment variables from the `.env` file.
  - Depends on the `db` service.

- The `db` service:

  - Uses the PostgreSQL version 15 image.
  - Defines a volume (`pg-prod-data`) to persist PostgreSQL data.
  - Loads environment variables from the `.env` file.
  - Sets environment variables for PostgreSQL user, password, and database name.

- Defines a volume named `pg-prod-data` to persist PostgreSQL data.

This configuration ensures that the Node.js application and PostgreSQL database are properly configured and connected for production deployment using Docker Compose.

`Docker Images`

![alt text](<Screenshot 2024-02-28 214603.png>)

`Docker Containers`

![alt text](<Screenshot 2024-02-28 214636.png>)

`Docker Volumes`

![alt text](<Screenshot 2024-02-28 214714.png>)

## Implementing Security Measures for Node.js Application

To fortify your Node.js application, it's crucial to implement security measures to safeguard sensitive information and ensure data integrity. Here are some best practices you can apply:

# Securing Environment Variables

## Best Practices:

1.  **Avoid Hardcoding Secrets**: Never hardcode sensitive information like API keys, database credentials, or tokens directly into your codebase. Instead, use environment variables to store and access these secrets.
2.  **Use .env Files**: Store environment-specific configurations, including sensitive data, in a `.env` file. This file should be excluded from version control systems to prevent accidental exposure.

3.  **Do Not Push .env to Git**: Ensure that the `.env` file is not pushed to a public or shared Git repository to prevent unauthorized access to sensitive information.
4.  **Use .gitignore**: Exclude the `.env` file from version control by adding it to your `.gitignore` file. This prevents Git from tracking changes to the `.env` file and ensures it remains local to your development environment.

5.  **Restrict Access**: Ensure that access to environment variables containing sensitive information is restricted to authorized users and processes only.

`sample .gitignore file`

```txt
# Logs
logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
lerna-debug.log*

node_modules
dist
dist-ssr
*.local

# Editor directories and files
.vscode/*
!.vscode/extensions.json
.idea
.DS_Store
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw?

# production
/build
dist
dev-dist
build
.env
.env.development
.env.production
.vercel
```

#### Example:

`# .env file`

# Database Configuration

```env
DB_HOST=localhost
DB_PORT=5432
DB_NAME=mydatabase
DB_USER=myuser
DB_PASSWORD=mypassword
```

# API Key

`API_KEY=mysecreatapikey`

### Employing HTTPS for Data Encryption

#### Best Practices:

1.  **Encrypt Data in Transit**: Utilize HTTPS to encrypt data transmitted between the client and server, protecting it from interception and tampering by malicious actors.
2.  **Obtain Valid SSL Certificate**: Acquire a valid SSL certificate from a trusted certificate authority (CA) to enable HTTPS encryption for your domain. This certificate verifies the identity of your server and establishes a secure connection with client browsers.

### Watch Demonstration Video:

[Video Link](https://www.loom.com/share/a4d1ecb239654fcbaf0f70a3e7e0bb02?sid=3b075ea6-ad30-4158-b264-5cc3200d289c)
