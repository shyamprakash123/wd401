# Configuration of Testing Framework

### Jest

Jest is a popular testing framework for JavaScript projects, offering a delightful developer experience with features like snapshot testing, mocking, and built-in code coverage reports. Below, I'll outline the steps to configure Jest to run a test suite in a Node.js project:

### 1. Installation

First, ensure you have Node.js installed on your machine. Then, initialize a new Node.js project (if you haven't already) and install Jest as a development dependency:

```bash
npm init -y  # Initializes a new Node.js project
npm install --save-dev jest
```

### 2. Configuration

Jest can be configured via a `jest.config.js` file or through the `"jest"` field in your `package.json`. Let's create a `jest.config.js` file at the root of your project:

### 3. Writing Tests

Create test files within your project, following a naming convention that Jest recognizes. Jest will look for test files with names ending in `.test.js` or `.spec.js` by default. For example, we might have a file named `todos.js`:

```js
const request = require("supertest");
var cheerio = require("cheerio");
const db = require("../models/index");
const app = require("../app");

let server, agent, latestTodo;
function extractCsrfToken(res) {
  var $ = cheerio.load(res.text);
  return $("[name=_csrf]").val();
}

const login = async (agent, username, password) => {
  let res = await agent.get("/login");
  let csrfToken = extractCsrfToken(res);
  res = await agent.post("/session").send({
    email: username,
    password: password,
    _csrf: csrfToken,
  });
};

describe("Todo test suite", () => {
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

    // signup user B
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
    let res = await agent.get("/todos");
    expect(res.statusCode).toBe(200);
    res = await agent.get("/signout");
    expect(res.statusCode).toBe(302);
    res = await agent.get("/todos");
    expect(res.statusCode).toBe(302);
  });

  test("creating a new todo", async () => {
    const agent = request.agent(server);
    await login(agent, "userA@gmail.com", "1234");
    const res = await agent.get("/todos");
    const csrfToken = extractCsrfToken(res);
    const response = await agent.post("/todos").send({
      title: "Buy milk",
      dueDate: new Date().toISOString(),
      completed: false,
      _csrf: csrfToken,
    });

    expect(response.statusCode).toBe(302);
  });
});
```

### 4. Running Tests

You can now run your tests using the `jest` command in your terminal:

```bash
npx jest
```

# Test Suite Coverage

### 1. Unit Tests

Unit tests, which are usually performed at the function or module level, concentrate on testing discrete application modules or components. These tests make sure every unit operates according to specification and acts as intended. The following is an approach to unit testing:

#### a. Identify Units:

Identify the various units within our application, such as functions, classes, or modules.

#### b. Write Tests:

Write tests for each unit, covering different scenarios and edge cases. Use mocking to isolate dependencies and ensure that each unit is tested in isolation.

#### c. Example (Using Jest):

```js
// app.js
app.post(
  "/todos",
  connectEnsureLogin.ensureLoggedIn(),
  async (request, response) => {
    console.log("Creating a todo", request.body, request.user.id);
    try {
      if (request.body.title.length == 0) {
        request.flash("error", "Please enter Todo-name");
      } else if (request.body.title.length < 5) {
        request.flash("error", "Todo-name length must be minimum 5 characters");
      }
      if (request.body.dueDate == 0) {
        request.flash("error", "Please enter dueDate");
      }
      if (request.body.title.length >= 5 && request.body.dueDate.length > 0) {
        const todo = await Todo.addTodo({
          title: request.body.title,
          dueDate: request.body.dueDate,
          completed: false,
          userId: request.user.id,
        });
        request.flash("success", "New Todo is Added");
      }
      return response.redirect("/todos");
    } catch (error) {
      console.error(error);
      return response.status(422).json(error);
    }
  }
);
module.exports = app;
```

```js
// todo.js
const app = require("../app");

test("creating a new todo", async () => {
  const agent = request.agent(server);
  await login(agent, "userA@gmail.com", "1234");
  const res = await agent.get("/todos");
  const csrfToken = extractCsrfToken(res);
  const response = await agent.post("/todos").send({
    title: "Buy milk",
    dueDate: new Date().toISOString(),
    completed: false,
    _csrf: csrfToken,
  });

  expect(response.statusCode).toBe(302);
});
```

### 2. Integration Tests

Testing the interactions between several levels or components of your application, including as frontend UI elements, backend APIs, and external services, is called integration testing with Cypress. Cypress offers a robust and user-friendly interface for creating integration tests for online applications. I'll go over how to use Cypress to construct integration tests below:

### 1. Installation

Install Cypress using npm or yarn:

```bash
npm install --save-dev cypress
```

### 2. Setting Up Cypress

Once Cypress is installed, you can initialize it in your project:

```bash
npx cypress open
```

This command will create a `cypress` directory in your project and open the Cypress Test Runner.

### 3. Writing Integration Tests

Inside the `cypress/integration` directory, we can create test files for your integration tests. Here's an example of an integration test for a web application:

```js
// cypress/integration/todo_app.spec.js

describe("Todo Application", () => {
  beforeEach(() => {
    // Visit the URL of your web application before each test
    cy.visit("http://localhost:3000"); // Adjust URL accordingly
  });

  it("Should display a list of todos", () => {
    // Assuming there is a list of todos on the page
    cy.get(".todo-list").should("have.length.greaterThan", 0);
  });

  it("Should add a new todo", () => {
    // Click on the input field and type a new todo
    cy.get(".new-todo").type("Buy groceries{enter}");

    // Verify that the new todo is added to the list
    cy.get(".todo-list").should("contain", "Buy groceries");
  });

  it("Should mark a todo as completed", () => {
    // Assuming there is a checkbox for each todo item
    cy.get(".todo-item").first().find(".toggle").click();

    // Verify that the todo item is marked as completed
    cy.get(".todo-item").first().should("have.class", "completed");
  });
});
```

### 4. Running Integration Tests

You can run our Cypress integration tests either via the Cypress Test Runner or using the command line.

#### Test Runner:

```bash
npx cypress open
```

This command will open the Cypress Test Runner, allowing you to select and run your integration tests interactively.

#### Command Line:

```bash
npx cypress run
```

This command will run our integration tests headlessly in the terminal.

# Automatic Test Suite Execution on GitHub

When a test suite is pushed to GitHub, it launches automatically. Cypress and Jest are used to configure the test suite. GitHub Actions is utilized for running the test suite.

## Github Actions

Using a tool like GitHub Actions, we can set up continuous integration (CI) processes to automate the execution of test suites on GitHub.

**Unit Test using Jest**

![Jest Unit Test](<Screenshot 2024-02-14 224600.png>)

# GitHub Actions Walkthrough

### Create GitHub Actions Workflow

1.  Navigate to GitHub repository.
2.  Go to the "Actions" tab.
3.  Click on "Set up a workflow yourself" or choose a template based on your project needs.
4.  Create a YAML file (e.g., `.github/workflows/main.yml`) to define a workflow.

```yml
name: Auto test for todos
on: push
env:
  PG_DATABASE: wd-todo-test
  PG_USER: postgres
  PG_PASSWORD: shyam@1234
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
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: shyam@1234
          POSTGRES_DB: wd-todo-test
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
      # For more information, see https://docs.npmjs.com/cli/ci.html
      - name: Install dependencies
        run: cd todo-app && npm ci

      - name: Run unit tests
        run: cd todo-app && npm test
      - name: Run the app
        id: run-app
        run: |
          cd todo-app
          npm install
          npx sequelize-cli db:drop
          npx sequelize-cli db:create
          npx sequelize-cli db:migrate
          PORT=3000 npm start &
          sleep 5
```

### Explaination

- **name**: Indicates the workflow's name. When seeing the workflow runs in the GitHub Actions UI, this label for the process helps clarify its purpose.

- **on**: Specifies the occasions that start the workflow.

  - push: Indicates that when changes are pushed to the repository, the workflow should start.

- **jobs**: Consists of one or more jobs specifying the actions to be carried out.
  - **test**: Describes the work called test.
- **steps**: Contains a list of actions that must be taken to complete the task.
  - **uses**: Indicates a course of action or series of steps to take.

Github Actions unit test:
![Github1](<Screenshot 2024-02-14 212349-1.png>)

All Jobs completed successfully:
![Github2](<Screenshot 2024-02-14 212412-1.png>)
