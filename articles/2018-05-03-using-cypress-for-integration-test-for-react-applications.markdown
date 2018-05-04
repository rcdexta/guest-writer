---
layout: post
title: "Using Cypress to write Integration Tests for your React Application"
description: ""
date: "2018-05-04 08:30"
author:
  name: "RC"
  url: "rcdexta"
  mail: "rc.chandru@gmail.com"
  avatar: "https://twitter.com/rcdexta/profile_image?size=original"

---

This post an introduction to using [Cypress](https://www.cypress.io/) for writing browser-based automation tests for web applications. This guide uses a React application as an example to illustrate the testing framework, but you can apply the learnings to write integration tests for any web application. 

## The Test Pyramid and Importance of E2E tests

A typical test suite for your application can generally consist of three layers (you could have many more depending on the nature of the application):

* Unit Tests
* Integration Tests
* End-to-end Tests

![test-pyramid](https://raw.githubusercontent.com/rcdexta/guest-writer/react-cypress/assets/cohn_test_pyramid.png)

Numerous tools and libraries help you write tests that offer safety net to your application at each of the layers listed above. But, when it comes to level of redundancy wrt to test coverage and which layer should assert what, the following two rules might help:

1. Write tests with different granularity
2. The more high-level you get the fewer tests you should have

Also, the feedback cycle for tests in each of the layers can vary. The unit tests offer the lowest latency on feedback and the latency increases as you go up the pyramid. For this post, we will focus on E2E tests and specifically User Interface tests. 

User Interface or UI tests, are tests that test the system just like a real user would interact with the application on the browser. They mimic the user’s actions in the form of a script, and it basically interacts with the app just like a regular user would.  Cypress is one of the many web automation tools available to help you write effective user interface tests.

## How is Cypress different as a UI Testing Tool?

Most browser-based automation tools are [selenium-based](https://www.seleniumhq.org/), which works by implementing a web driver that remotely executes commands on the browser through the network. Whereas, Cypress runs in the same run-loop as your parent application. Also, Cypress tests are only written in JavaScript. While you can compile down to JavaScript from any other language, ultimately the test code is executed inside the browser itself. These architectural improvements unlock the ability to run tests much faster in browser mode as well as headless mode. You can read more about the internal details [here](https://www.cypress.io/how-it-works/). As you start writing more Cypress tests and build the pipeline with a test suite, you will see that many aspects of writing automation tests have improved with this library. 

## Writing your first Cypress test

We have created a sample todo app using React to better illustrate using Cypress to write automation tests. Screenshot of the application that we are going to test against can be seen below:

![todo-app](https://raw.githubusercontent.com/rcdexta/guest-writer/react-cypress/assets/todoapp.png)



It has a list of todos that are open or completed. The todo can be completed or deleted using the action icons present against each todo item. The text field and the button can be used to add a new todo item.

Let's get started with installing Cypress and automating adding a todo item to the list. To get started, the code for the app is available to download and run:

```bash
$ git clone git@github.com:rcdexta/cypress-todo-example.git
$ cd cypress-todo-example
$ yarn install
```

When you are done installing all the dependencies, run `yarn start` to run the application in the browser to make sure it looks good. The same application is also available in [codesandbox](https://codesandbox.io/s/4j5m27o8ox) if you have trouble running it on your machine.

Now, let's bootstrap cypress and write our first test. With the application running, open another terminal session and run the following commands inside the same folder:

```bash
$ yarn add --dev cypress
$ yarn cypress open
```

Cypress will sense that you are running it for the first time and create a folder called `cypress` with necessary files to get you going. It would also launch the cypress test runner. Think of it as a GUI for running/debugging your automation specs.

We will create a file called `todo_spec.js`  in `cypress/integration` folder. 

> Note: You should already see an `example_spec.js `file that contains tests for the sample [Kitchen Sink](https://example.cypress.io/)  application with plenty of documentation for various scenarios. Would recommend reading through the file at a later point in time for reference

```javascript
//cypress/integration/todo_spec.js

describe('Todo App', function () {

    it('.should() - allow adding a new todo item', function () {
        cy.visit('http://localhost:3000') 
        cy.get('input[data-cy=newItemField]').type('Write Test') 
        cy.get('#addBtn').click() 

        cy.get('tr[data-cy=todoItem]:nth-child(1)').should('contain', 'Write Test') 
    })
})
```

The test is pretty self-explanatory. `cy` is a global cypress object that drives the tests. It has various helper methods to visit web pages, interact with web elements and also to fetch data present in the DOM. 

Cypress has excellent [documentation](https://docs.cypress.io/guides/overview/why-cypress.html) and a [sample application](https://example.cypress.io/) that you would be frequently referring to as you add more test cases to your application. Cypress also bundles [chai](https://docs.cypress.io/api/commands/should.html#Syntax) assertion library and the should matcher you see in the test is chained to the cypress helper methods seamlessly for readability.

## Using the Cypress Test Runner

When you have saved the`todo_spec.js` file, it will start appearing in the Cypress test runner. When you click on todo_spec.js, a new browser instance will open up and run the test visually. You can observe cypress hop through each step that we wrote in the add todo item test. Check out a snapshot of the browser as it runs the test:

![cypress-gui-runner](https://raw.githubusercontent.com/rcdexta/guest-writer/react-cypress/assets/cypress_gui_runner.png)

The left pane is called the `Command Log` and lists the name of the test followed by each step in sequence and its outcome. When the test runs, you can see Cypress interacting with the application on the browser webview to the right and we see that the new todo item was added to the list successfully. And the last step on the left is the assertion to verify if our new item is present in the list and voila!

One very helpful feature on the browser view is the DOM selector that is shown highlighted below the URL address bar. It will appear when you click on the `Open Selector Playground` icon to the left of the address bar. You can type in a selector in the textbox like we have entered `#addBtn` and cypress will highlight the selected item for you. Or you can select an element from the screen and Cypress will try to find the optimal selector for you to use in your test. 

## Setting up Continuous Integration

The whole point of writing and maintaining browser-based automation tests is to test the application for any flows that break and give early feedback to the team. You can set up the suite of cypress tests to run on your continuous integration server to run on every check-in or as a nightly build. We will quickly walk through the steps to set up the build and we will keep it agnostic of any particular CI tool.

Most CI build agents are servers and not workstations, so we cannot expect them to have a GUI or X Window system to launch a real browser UI. So, it is a common practice to run web automation tests in [headless mode](https://en.wikipedia.org/wiki/Headless_browser). To keep it simple, we would be using a virtual GUI buffer like [xvfb](http://elementalselenium.com/tips/38-headless) to fire a web browser virtually, launch the target application and run cypress tests against it.

Assuming, you are setting up a docker agent on the CI server to run the tests, this is the specification for the`Dockerfile`

```dockerfile
FROM cypress/base

// Clone your code base here

RUN npm install
RUN $(npm bin)/cypress run
```

Refer to this [page](https://docs.cypress.io/guides/guides/continuous-integration.html#) for setting up Cypress on your favorite CI server. 

## Time Travel and Debugging 

Another cool feature present in the Cypress Test Runner tool is that as you hover through the command log on the left, for each step in the test, you can check the state of the application on the right. You can also click on the pin icon to freeze the runner for that particular step, open the developer console and get insights into the specific web interactions that happened, the selectors used, parameters passed and results.

![time-travel-gif](https://raw.githubusercontent.com/rcdexta/guest-writer/react-cypress/assets/timetravel.gif)

## Postmortem analysis with screenshots and videos

The last thing we want with a browser automation test is false positives. By that, a test case failing because of the way the automation test was written and not because something in the actual application is broken. When that frequently happens, the test suite loses its credibility and a red build is not worth its attention anymore! 

So, when a test fails on the CI we need enough trails and artifacts to understand what really went wrong. Like most automation frameworks in the market, Cypress generates screenshots and videos of the entire test runs if configured correctly. All of these settings can be tweaked using the `cypress.json` configuration file generated in the root of your project. By default, screenshots and video recording are turned on and you can see them generated inside the `cypress` folder after running the tests.

Check out the [configuration documentation](https://docs.cypress.io/guides/references/configuration.html#Videos) for additional settings and options.

## Source Code

The source code for the react application and the corresponding cypress tests are available in the following this [repository](https://github.com/rcdexta/cypress-todo-example)

The `master` branch should help you get started with this guide. Navigate to `cypress` branch to look at the final setup with all the tests.

## Further Reading

1. [Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
2. [Most common Selenium challenges](https://crossbrowsertesting.com/blog/selenium/problems-selenium-webdriver/)
3. [Cypress architecture](https://www.cypress.io/how-it-works/)

