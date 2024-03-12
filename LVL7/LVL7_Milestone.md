## Dockerizing and Deploying a Node.js Application with CICD Pipeline

### Dockerizing the Application

#### Dockerfile Explanation

DockerfileCopy code

```Dockerfile
FROM --platform=$BUILDPLATFORM node:lts-alpine as base
WORKDIR /app
COPY package.json /
EXPOSE 3000
```

# Production Stage

```Dockerfile
FROM base as production
ENV NODE_ENV=production
RUN npm install
RUN npm install pm2 -g
COPY . /app
CMD ["pm2-runtime", "start", "index.js", "--instances", "max", "--output", "/app/pm2.log", "--error", "/app/pm2_error.log"]
```

# Development Stage

```Dockerfile
FROM base as dev
ENV NODE_ENV=development
RUN npm install -g npm@10.5.0
RUN npm install -g nodemon && npm install
COPY . /app
CMD npm run start
```

### Explanation:

1.  **Base Stage**:

    - Utilizes the Node.js LTS Alpine image as the base image.
    - Sets the working directory to `/app`.
    - Copies `package.json` to the root directory.

2.  **Production Stage**:

    - Inherits from the base stage.
    - Sets the environment variable `NODE_ENV` to `production`.
    - Installs dependencies using `npm install`.
    - Installs `pm2` globally for process management.
    - Copies the entire application directory to `/app`.
    - Executes the application using `pm2-runtime`, which manages the application lifecycle and provides features like process monitoring and logging.

3.  **Development Stage**:

    - Inherits from the base stage.
    - Sets the environment variable `NODE_ENV` to `development`.
    - Upgrades npm version to 10.5.0 globally.
    - Installs `nodemon` globally for automatic server restarts during development.
    - Installs dependencies using `npm install`.
    - Copies the entire application directory to `/app`.
    - Starts the application using `npm run start`.

##

Configuring Environment Variables in Docker

### Purpose and Expected Values

`.env` file:

```.env
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

#### For Production Environment:

- **NODE_ENV**: Specifies the environment mode. Expected value: `production`.
- **PROD_USERNAME**: Database username for production. Expected value: A secure username.
- **PROD_PASSWORD**: Database password for production. Expected value: A secure password.
- **PROD_DATABASE**: Name of the production database. Expected value: A valid database name.
- **PROD_HOST**: Hostname for the production database server. Expected value: Hostname or IP address.
- **PROD_DIALECT**: Database dialect for production. Expected value: The type of database (e.g., `postgres`, `mysql`, `mongodb`).

#### For Development Environment:

- **NODE_ENV**: Specifies the environment mode. Expected value: `development`.
- **DEV_USERNAME**: Database username for development. Expected value: A secure username.
- **DEV_PASSWORD**: Database password for development. Expected value: A secure password.
- **DEV_DATABASE**: Name of the development database. Expected value: A valid database name.
- **DEV_HOST**: Hostname for the development database server. Expected value: Hostname or IP address.
- **DEV_DIALECT**: Database dialect for development. Expected value: The type of database (e.g., `postgres`, `mysql`, `mongodb`).

### Secure Practices for Managing Sensitive Information

1.  **Use of .env File**: Storing sensitive information like database credentials in a separate `.env` file ensures that these details are not hardcoded within the Dockerfile or application code.
2.  **Gitignore**: Ensure that the `.env` file is included in the `.gitignore` to prevent accidental exposure of sensitive information.
3.  **Encryption**: For additional security, consider encrypting the `.env` file or using a secret management tool to store and retrieve sensitive data securely.
4.  **Access Control**: Restrict access to the `.env` file to authorized users only.
5.  **Avoid Hardcoding**: Refrain from hardcoding sensitive information directly into Dockerfiles or application code to minimize the risk of exposure.

## Docker Compose Configuration for Multiple Services

### `docker-compose.yml` - Development Configuration

```yml
version: "3.8"
services:
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

volumes:
  pg-dev-data:
```

#### Explanation:

- **app Service**:

  - Builds the Docker image for the Node.js application using the development target.
  - Maps the local directory to `/app` in the container for live code reloading.
  - Exposes port `3000` for accessing the application.
  - Loads environment variables from the `.env` file.
  - Depends on the `db` service.

- **db Service**:

  - Uses the PostgreSQL image version 15.
  - Mounts a volume for persisting database data.
  - Loads environment variables from the `.env` file.
  - Specifies environment variables for PostgreSQL configuration.

### `docker-compose-prod.yml` - Production Configuration

```yml
version: "3.8"
services:
  app:
    build:
      context: .
      target: production
    image: todo-app:production
    ports:
      - "3020:3000"
    env_file:
      - .env
    depends_on:
      - db

  db:
    image: postgres:15
    volumes:
      - pg-prod-data:/var/lib/postgresql/data
    env_file:
      - .env
    environment:
      POSTGRES_USER: $PROD_USERNAME
      POSTGRES_PASSWORD: $PROD_PASSWORD
      POSTGRES_DB: $PROD_DATABASE

volumes:
  pg-prod-data:
```

#### Explanation:

- **app Service**:

  - Builds the Docker image for the Node.js application using the production target.
  - Maps port `3020` on the host to port `3000` in the container.
  - Loads environment variables from the `.env` file.
  - Depends on the `db` service.

- **db Service**:

  - Uses the PostgreSQL image version 15.
  - Mounts a volume for persisting database data.
  - Loads environment variables from the `.env` file.
  - Specifies environment variables for PostgreSQL configuration.

![alt text](<Screenshot 2024-02-28 214456.png>)

![alt text](<Screenshot 2024-02-28 214603.png>)

![alt text](<Screenshot 2024-02-28 214636.png>)

![alt text](<Screenshot 2024-02-28 214714.png>)

## Setting Up CI/CD Pipeline for Docker Hub

### Overview

This CI/CD pipeline automates the deployment process of a Dockerized Node.js application using GitHub Actions. It consists of three stages: testing, building Docker image, and deployment to a server. Proper error handling and reporting are integrated to notify stakeholders about the pipeline's status.

### Pipeline Configuration

```yml
name: CI-CD Pipeline
on: push
env:
  PG_DATABASE: ${{ secrets.PG_DATABASE }}
  PG_USER: ${{ secrets.PG_USER }}
  PG_PASSWORD: ${{ secrets.PG_PASSWORD }}
  SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
jobs:
  # Label of the container job
  run-tests:
    # Containers must run in Linux based operating systems
    runs-on: ubuntu-latest

    # Service containers to run with `container-job`
    services:
      # Label used to access the service container
      postgres:
        # Docker Hub image
        image: postgres:11.7
        # Provide the password for postgres
        env:
          POSTGRES_USER: ${{ secrets.PG_USER }}
          POSTGRES_PASSWORD: ${{ secrets.PG_PASSWORD }}
          POSTGRES_DB: ${{ secrets.PG_DATABASE }}
        # Set health checks to wait until postgres has started
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      # Downloads a copy of the code in your repository before running CI tests
      - name: Check out repository code
        uses: actions/checkout@v3

      # Performs a clean installation of all dependencies in the `package.json` file
      - name: Install dependencies
        run: cd My-sports-schedular && npm ci

      # Runs the unit tests
      - name: Run unit tests
        run: cd My-sports-schedular && npm test

      - name: Notify Slack
        uses: rtCamp/action-slack-notify@v2
        with:
          status: ${{ job.status }}
          author_name: GitHub Actions
          steps: ${{toJson(steps)}}
          title: ${{ job.status }} ${{ github.repository }}
          fields: Repo ${{ github.repository }}
          color: ${{ job.status == 'success' && 'good' || 'danger' }}
          channel: "My Sports Schedular Github Actions"
        if: always()
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}

      - name: Notify Success
        if: success()
        run: |
          MESSAGE='🎉 Great news! Your tests was successful! 🚀'
          PAYLOAD="{\"text\":\"$MESSAGE\"}"
          curl -X POST -H 'Content-type: application/json' --data "$PAYLOAD" "${{ secrets.SLACK_WEBHOOK }}"

      - name: Notify Failure
        if: failure()
        run: |
          FAILURE_MESSAGE='⚠️ Oh no! The tests has failed! ❌'
          FAILURE_PAYLOAD="{\"text\":\"$FAILURE_MESSAGE\"}"
          curl -X POST -H 'Content-type: application/json' --data "$FAILURE_PAYLOAD" "${{ secrets.SLACK_WEBHOOK }}"

  build-docker-image:
    name: Build Docker Image
    needs: run-tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Docker Image && Push to Docker Hub
        run: |
          cd todo-app
          docker build -t ${{ secrets.DOCKER_USERNAME }}/todo-app .
          docker push ${{ secrets.DOCKER_USERNAME }}/todo-app

  # Deployment steps in CI-CD Pipeline
  deploy:
    name: Deploy to render (production)
    needs: build-docker-image
    runs-on: ubuntu-latest

    steps:
      - name: Deploy to render (production)
        uses: johnbeynon/render-deploy-action@v0.0.8
        with:
          service-id: ${{ secrets.SERVICE_ID }}
          api-key: ${{ secrets.RENDER_API_KEY }}

      - name: Notify Slack
        uses: rtCamp/action-slack-notify@v2
        with:
          status: ${{ job.status }}
          author_name: GitHub Actions
          steps: ${{toJson(steps)}}
          title: ${{ job.status }} ${{ github.repository }}
          fields: Repo ${{ github.repository }}
          color: ${{ job.status == 'success' && 'good' || 'danger' }}
          channel: "My Sports Schedular Github Actions"
        if: always()
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}

      - name: Notify Success
        if: success()
        run: |
          MESSAGE='🎉 Great news! Your deploy was successful! 🚀'
          PAYLOAD="{\"text\":\"$MESSAGE\"}"
          curl -X POST -H 'Content-type: application/json' --data "$PAYLOAD" "${{ secrets.SLACK_WEBHOOK }}"

      - name: Notify Failure
        if: failure()
        run: |
          FAILURE_MESSAGE='⚠️ Oh no! The deploy has failed! ❌'
          FAILURE_PAYLOAD="{\"text\":\"$FAILURE_MESSAGE\"}"
          curl -X POST -H 'Content-type: application/json' --data "$FAILURE_PAYLOAD" "${{ secrets.SLACK_WEBHOOK }}"
```

#### Stage 1: Testing (`run-tests` job)

- **Purpose**: Validates code changes by running unit tests.
- **Steps**:
  1.  Checks out the repository code.
  2.  Installs dependencies and runs unit tests.
  3.  Notifies Slack channel about the test status (success or failure).
  4.  Sends a Slack notification on test success or failure.

#### Stage 2: Building Docker Image (`build-docker-image` job)

- **Purpose**: Builds a Docker image for the application and pushes it to Docker Hub.
- **Steps**:
  1.  Checks out the code.
  2.  Logs in to Docker Hub using provided credentials.
  3.  Builds the Docker image from the `todo-app` directory.
  4.  Pushes the built image to Docker Hub.

#### Stage 3: Deployment (`deploy` job)

- **Purpose**: Deploys the Dockerized application to a server using Render.
- **Steps**:
  1.  Deploys the application to Render production environment using Render Deploy Action.
  2.  Notifies Slack channel about the deployment status (success or failure).
  3.  Sends a Slack notification on deployment success or failure.

![alt text](<Screenshot 2024-03-12 210211.png>)

![alt text](<Screenshot 2024-03-12 210227.png>)

![alt text](<Screenshot 2024-03-12 210239.png>)

![alt text](<Screenshot 2024-03-12 210251.png>)

![alt text](<Screenshot 2024-03-12 210309.png>)

![alt text](<Screenshot 2024-03-12 212749.png>)

![alt text](<Screenshot 2024-03-12 212239.png>)

### Watch Demonstration Video:

[Video Link](https://www.loom.com/share/b7dc5dd339fd4a8190b119f3f88ed29c?sid=1f057a01-54f3-490e-97f7-8c6a43d70829)

[Continuation video...](https://www.loom.com/share/360fa5d4d0644e998601f306a23d77ba?sid=5071d8a0-5027-4d5f-970b-acf783dbbdb5)
