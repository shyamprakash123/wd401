# Setting up a CI/CD Pipeline using GitHub Actions

In this guide, we'll walk through the process of setting up a Continuous Integration/Continuous Deployment (CI/CD) pipeline for a web application using GitHub Actions. The pipeline will automate the deployment process, ensuring seamless integration of code changes and swift error detection.

## 1. GitHub Actions Configuration

First, let's create a `.github/workflows` directory in the root of your project repository. Within this directory, we'll add a YML (`main.yml`) file to define our CI/CD pipeline.

### Example Workflow YML File:

```yaml
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

  # Deployment steps in CI-CD Pipeline
  deploy:
    name: Deploy to render (production)
    needs: run-tests
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

## 2. Explanation of Workflow Stages:

- **Validate Code**: This stage checks the syntax of the code to ensure it meets coding standards and doesn't contain any errors. You can customize the linting command according to your project's setup.
- **Build and Test**: This stage installs project dependencies, builds the application, and runs tests to ensure the application behaves as expected.
- **Deploy to Production**: This stage deploys the application to the production environment after successful validation, building, and testing.

![alt text](<Screenshot 2024-02-28 172513.png>)

![alt text](<Screenshot 2024-02-28 172545.png>)

![alt text](<Screenshot 2024-02-28 172631.png>)

![alt text](<Screenshot 2024-02-28 172757.png>)

![alt text](<Screenshot 2024-02-28 172815.png>)

# Automated Testing in CI/CD Pipeline

Automated testing is a crucial aspect of any CI/CD pipeline, ensuring the functionality, performance, and integrity of the application. In this guide, we'll integrate automated testing into our pipeline and define and implement test cases covering critical aspects of the application's features.

## 1. Setting up Automated Testing

First, let's add a testing stage to our CI/CD pipeline YAML file. We'll use popular testing frameworks like Jest for Node.js applications or React Testing Library for React.js applications.

### Example Testing Stage:

```yaml
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
```

## 2. Defining Test Cases

Next, let's define test cases that cover critical aspects of your application's features. These test cases should validate functionality, performance, and data integrity.

### Example Test Cases (for a Node.js application using Jest):

```js
const login = async (agent, username, password) => {
  let res = await agent.get("/login");
  let csrfToken = extractCsrfToken(res);
  res = await agent.post("/session").send({
    email: username,
    password: password,
    _csrf: csrfToken,
  });
};

describe("My-Sports-Schedular test suite", () => {
  beforeAll(async () => {
    await db.sequelize.sync({ force: true });
    server = app.listen(3000, () => {});
    agent = request.agent(server);
  });
  afterAll(async () => {
    await db.sequelize.close();
    server.close();
  });

  test("Sign up", async () => {
    // signup user A
    let res = await agent.get("/signup");
    let csrfToken = extractCsrfToken(res);
    res = await agent.post("/users").send({
      firstName: "Test",
      lastName: "User A",
      email: "userA@gmail.com",
      password: "1234",
      _csrf: csrfToken,
    });
    expect(res.statusCode).toBe(302);

    // signup user A
    res = await agent.get("/signup");
    csrfToken = extractCsrfToken(res);
    res = await agent.post("/users").send({
      firstName: "Test",
      lastName: "User B",
      email: "userB@gmail.com",
      password: "1234",
      _csrf: csrfToken,
    });
    expect(res.statusCode).toBe(302);
  });

  test("sign out", async () => {
    let res = await agent.get("/sports");
    expect(res.statusCode).toBe(200);
    res = await agent.get("/signout");
    expect(res.statusCode).toBe(302);
    res = await agent.get("/sports");
    expect(res.statusCode).toBe(302);
  });

  test("creating a new sport", async () => {
    const agent = request.agent(server);
    await login(agent, "userA@gmail.com", "1234");
    const res = await agent.get("/User/changePassword");
    const csrfToken = extractCsrfToken(res);
    let sportName = "Cricket";
    const response = await agent.post("/sports/new-sport").send({
      sportName: sportName,
      _csrf: csrfToken,
    });
    expect(response.statusCode).toBe(302);

    // validating sport
    const groupedTodosResponse = await agent
      .get(`/sports/${sportName}`)
      .set("Accept", "application/json");
    const parsedGroupedResponse = JSON.parse(groupedTodosResponse.text);
    const sportId = parsedGroupedResponse.sportId;
    const isAdmin = parsedGroupedResponse.isAdmin;
    const returnedSportName = parsedGroupedResponse.sportName;
    const sessionsLength = parsedGroupedResponse.session.length;
    expect(sportId).toBe(1);
    expect(isAdmin).toBe(true);
    expect(returnedSportName).toBe(sportName);
    expect(sessionsLength).toBe(0);
  });

  test("creating a new session", async () => {
    const agent = request.agent(server);
    await login(agent, "userA@gmail.com", "1234");
    let res = await agent.get("/User/changePassword");
    let csrfToken = extractCsrfToken(res);
    let sportName = "Cricket";
    let date = new Date();
    let place = "CBIT";
    let members = "Shyam, Ram, Kiran";
    let players = 5;

    //Getting sportId using sportName
    const groupedTodosResponse = await agent
      .get(`/sports/${sportName}`)
      .set("Accept", "application/json");
    const parsedGroupedResponse = JSON.parse(groupedTodosResponse.text);
    const sportId = parsedGroupedResponse.sportId;

    const response = await agent.post(`/sports/${sportName}`).send({
      dateTime: date,
      place: place,
      members: members,
      players: players,
      sportId: sportId,
      _csrf: csrfToken,
    });
    expect(response.statusCode).toBe(302);
  });
```

## 3. Explanation

- **Automated Testing Stage**: This stage executes automated tests as part of the CI/CD pipeline. It ensures that the application behaves as expected and meets specified requirements.
- **Test Cases**: These are example test cases written in Jest, a popular testing framework for Node.js applications. The first test case validates the functionality of creating a user, while the second test case tests the retrieval of a user by ID.

![alt text](<Screenshot 2024-02-28 174656.png>)

![alt text](<Screenshot 2024-02-28 174757.png>)

# Environment Variable Configuration in CI/CD Pipeline

Proper configuration of environment variables is crucial for managing sensitive or environment-specific information required for the application to function correctly across various stages of the CI/CD pipeline. In this guide, we'll cover how to configure environment variables within the CI/CD pipeline.

## 1. Using Secrets in GitHub Actions

GitHub Actions allows you to store sensitive information, such as API keys, access tokens, or database credentials, securely as secrets. These secrets can be used in your workflow to configure environment variables.

### Example Secrets Configuration:

1.  Navigate to your GitHub repository.
2.  Go to "Settings" > "Secrets" > "New repository secret".
3.  Add the sensitive information as a secret (e.g., `DB_PASSWORD`, `API_KEY`).

## 2. Injecting Secrets into Workflow

Once you've added the secrets to your repository, you can inject them into your CI/CD pipeline workflow as environment variables.

### Example Workflow YAML with Secrets:

```yaml
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
```

## 3. Explanation

- **Secrets Configuration**: GitHub Actions allows you to securely store sensitive information as secrets at the repository level.
- **Injecting Secrets into Workflow**: You can inject secrets into your workflow as environment variables using the `${{ secrets.SECRET_NAME }}` syntax. These environment variables can then be accessed within your workflow steps or scripts.
-

### Currently Configured Secrets

The pipeline relies on specific environment variables crucial for the application's functionality across different stages. These variables include:

> `PG_USER`: Name of the PostgreSQL user. `PG_PASSWORD`: Password for the PostgreSQL user. `SERVICE_ID`: Service of to deploy to render.com `RENDER_API_KEY`: API Key to deploy to render.com

![alt text](<Screenshot 2024-02-28 174916.png>)

## 4. Best Practices

- **Limit Access**: Only grant access to secrets to users or services that require them for the CI/CD pipeline.
- **Rotate Secrets**: Regularly rotate sensitive information, such as API keys or access tokens, to enhance security.
- **Use Encrypted Secrets**: Avoid exposing secrets directly in workflow files or logs by using encrypted secrets stored in GitHub.

# Error Reporting Integration in CI/CD Pipeline

Integrating error reporting mechanisms into your CI/CD pipeline is essential for promptly identifying and resolving issues that may arise during pipeline execution. In this guide, we'll cover how to integrate error reporting and notification mechanisms, ensuring that the development team receives detailed error messages and logs in case of any pipeline failures.

## 1. Setting Up Error Reporting

To integrate error reporting into your pipeline, you can use error monitoring and logging services like Sentry, Rollbar, or AWS CloudWatch Logs. These services capture errors, exceptions, and logs generated during pipeline execution.

### Example Error Reporting Setup:

1.  **Choose an Error Reporting Service**: Select a suitable error monitoring and logging service based on your requirements and preferences.
2.  **Set Up Error Reporting Service**: Follow the documentation provided by the chosen service to set up error reporting for your application.
3.  **Generate Integration Keys**: Obtain integration keys or tokens required to authenticate your pipeline with the error reporting service.

## 2. Error Notification Mechanism

Once error reporting is set up, you can configure your pipeline to notify the development team in case of any pipeline failures. This notification should include detailed error messages, logs, and relevant information for quick diagnosis and resolution.

### Example Error Notification Setup:

1.  **Select Notification Channel**: Choose a communication platform for error notifications, such as Slack or Discord.
2.  **Set Up Integration with Communication Platform**: Use integrations provided by the communication platform to post notifications automatically.
3.  **Configure Notification Message**: Customize the notification message to include essential information, such as pipeline status, error message, affected stage, and relevant logs.

## 3. Implementing Error Reporting and Notification in Pipeline using SLACK

Finally, integrate error reporting and notification mechanisms into your CI/CD pipeline workflow. Capture errors and logs generated during pipeline execution and send notifications to the designated communication channel in case of any failures.

### Example Workflow YAML with Error Reporting and Notification using SLACK:

```yaml
# Deployment steps in CI-CD Pipeline
deploy:
  name: Deploy to render (production)
  needs: run-tests
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

## 4. Explanation

- **Error Reporting Setup**: Set up error reporting using a monitoring and logging service of your choice.
- **Error Notification Mechanism**: Choose a communication platform and configure integration to post notifications automatically.
- **Implementation in Pipeline**: Integrate error reporting and notification mechanisms into your CI/CD pipeline workflow. Capture errors, logs, and relevant information and send notifications to the designated communication channel in case of pipeline failures.

![alt text](<Screenshot 2024-02-28 175030.png>)

![alt text](<Screenshot 2024-02-28 175056.png>)

### Screen Recorded Link For CI/CD Integration:

[Watch the Screen Recording](https://www.loom.com/share/fe0dad6926b149f9b76d0854fb0f2bb1?sid=d6ebf39d-16ba-44ce-945f-ba77e8b30b3a)

[Continuation video...](https://www.loom.com/share/28b7fc6d6c0c4c66bbdcaa541707d253?sid=d97260c8-f0b8-40cc-92bf-238a66ff997d)
