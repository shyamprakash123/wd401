## Problem Statement:

You've been assigned to a project that involves enhancing a critical feature for a web application. The team places a strong emphasis on the pull-request workflow, with a focus on code reviews, merge conflict resolution, and the recent integration of CI/CD. As you navigate through the development task, you encounter challenges such as feedback during code reviews and discussions on effective merge conflict resolution. The team looks to you to demonstrate your understanding of these challenges and your ability to adapt to the added complexity of CI/CD integration.

## Code Submitted For Review (Author):

```javascript
const createproduct = async () => {
  try {
    let response = await fetch(`${API_ENDPOINT}/createProduct`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        product_name: productName,
        product_description: productDescription,
        product_price: productPrice,
      }),
    });

    if (!response.ok) {
      throw new Error("Product creation failed with status");
    }

    const data = await response.json();
    const isProductAvailable = data.product_quantity > 0 ? true : false;
    if (isProductAvailable) {
      setProduct(product, data);
    } else {
      throw new Error("Product is not available");
    }

    navigate("/home");
  } catch (err) {
    console.error("Something went wrong!..", err);
  }
};
```

## Handling Code Review Feedback:

```javascript
// use camelCase syntax for better readability.
const createProduct = async () => {
  try {
    // use const variable for promise response.
    const response = await fetch(`${API_ENDPOINT}/createProduct`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      // if payload items are more than two then use multiple lines to maintain code quality.
      body: JSON.stringify({
        product_name: productName,
        product_description: productDescription,
        product_price: productPrice,
      }),
    });

    if (!response.ok) {
      // make proper intendation and also mention the response status code.
      throw new Error(`Product creation failed with status ${response.status}`);
    }

    const data = await response.json();
    // No need of ternery operator if the value is boolean.
    const isProductAvailable = data.product_quantity > 0;
    if (isProductAvailable) {
      setProduct(
        // use spread operator.
        ...product,
        data
      );

      // if the product is not available then do not navigate user to home.
      navigate("/home");
    } else {
      // No need to throw an error just alert the user.
      Notify.error("Product is not available");
    }
    // don't use shortcut variables, use error for the standard code.
  } catch (error) {
    console.error("Something went wrong!..", error);
  }
};
```

## Iterative Development Process:

![Flow-Chart](https://github.com/shyamprakash123/wd401/assets/106866225/77f7e23b-9be8-4ec4-be85-f559e541d529)

## Resolving Merge Conflicts:

A merge conflict occurs in version control systems when two branches or versions of a codebase have changes that cannot be automatically merged. This situation typically arises when two developers make conflicting modifications to the same part of a file, and the version control system is unable to determine which changes should take precedence.

### Code in branch ABC:

```javascript
const createProduct = async () => {
  try {
    const response = await fetch(`${API_ENDPOINT}/createProduct`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        product_name: productName,
        product_description: productDescription,
        product_price: productPrice,
      }),
    });

    if (!response.ok) {
      console.error(`Product creation failed with status ${response.status}`);
    }

    const data = await response.json();
    const isProductAvailable = data.product_quantity > 0;
    if (isProductAvailable) {
      setProduct(...product, data);

      navigate("/home");
    } else {
      Notify.error("Product is not available");
    }
  } catch (error) {
    console.error("Something went wrong!..", error);
  }
};
```

### Code in branch XYZ:

```javascript
const createProduct = async () => {
  try {
    const response = await fetch(`${API_ENDPOINT}/createProduct`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        product_name: productName,
        product_description: productDescription,
        product_price: productPrice,
      }),
    });

    if (!response.ok) {
      throw new Error(`Product creation failed with status ${response.status}`);
    }

    const data = await response.json();
    const isProductAvailable = data.product_quantity > 0;
    if (isProductAvailable) {
      setProduct(...product, data);

      navigate("/home");
    } else {
      Notify.error("Product is not available");
    }
  } catch (error) {
    console.error("Something went wrong!..", error);
  }
};
```

### Merge confict occurs in the branch XYZ when branch ABC is merged with the develop branch then:

```javascript
const createProduct = async () => {
    try{
        const response = await fetch(`${API_ENDPOINT}/createProduct`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            product_name: productName,
            product_description:
            productDescription,
            product_price: productPrice
            }),
        });

        if (!response.ok) {
            <<<<<<< HEAD
            throw new Error(`Product creation failed with status ${response.status}`);
            =======
            console.error(`Product creation failed with status ${response.status}`);
            >>>>>>> Branch ABC
        }

        const data = await response.json();
        const isProductAvailable = data.product_quantity > 0
        if (isProductAvailable){
            setProduct(
                ...product,
                data
            );

            navigate("/home");
        }else{
            Notify.error("Product is not available");
        }
    } catch (error) {
        console.error("Something went wrong!..", error);
    }
}
```

> There are two options to resolve the conflicts:
>
> - Accept Incomming changes.
> - Keep current changes.
>   By accepting the incomming changes the conflict of branch XYZ is resolved with the branch ABC.
>   By keeping the current changes the conflict of branch XYZ merging with develop will be resolved with current changes.
>   > The main AIM is to resolve the merge conflicts such a way that the application works as expected without producing new errors.

### Afer resolving the merge conflicts by keeping the changes in XYZ:

```javascript
const createProduct = async () => {
  try {
    const response = await fetch(`${API_ENDPOINT}/createProduct`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        product_name: productName,
        product_description: productDescription,
        product_price: productPrice,
      }),
    });

    if (!response.ok) {
      throw new Error(`Product creation failed with status ${response.status}`);
    }

    const data = await response.json();
    const isProductAvailable = data.product_quantity > 0;
    if (isProductAvailable) {
      setProduct(...product, data);

      navigate("/home");
    } else {
      Notify.error("Product is not available");
    }
  } catch (error) {
    console.error("Something went wrong!..", error);
  }
};
```

## CI/CD Integration:

> Using node packages like prettier, eslint, jest, husky to formate the code and these ensures the code quality standards.
> These tests runs after commiting the changes to the files.

### This is basic CI/CD integration with test suites:

- Here, I have taken creating a new sport and new session from my previous project.

```javascript
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

> If these test suite fails then merging and pushing for the pull-request is denied this ensures the code quality checks and expected working.

## Using Github Actions for Tests and Quality checks and also to deploy:

`.github/workflows/main.yml`

```yml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 14

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test

  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 14

      - name: Deploy to production
        run: npm run deploy
```
