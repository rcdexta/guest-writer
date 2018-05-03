---
layout: post
title: "Using Cypress to write effective integration tests for your React web applications"
description: ""
date: "2018-05-03 08:30"
author:
  name: "RC"
  url: "rcdexta"
  mail: "rc.chandru@gmail.com"
  avatar: "https://twitter.com/rcdexta/profile_image?size=original"

---

**TL;DR:** Briefly describe what this article is about and what the reader will achieve/learn after reading it. Please,
also provide the link to a GitHub repository that contains code related to this article.



## Writing your first Cypress test

We have created a sample todo app using React to better illustrate using Cypress to write automation tests. Screenshot of the application that we are going to test against can be seen below:

![todo-app](https://photos-1.dropbox.com/t/2/AAAEaHPEJWymttorIMhwF39F15D-PAGkwtcIm4XvMzTHJQ/12/5183783/png/32x32/3/1525341600/0/2/todoapp.png/ENfW8AMY8KIBIAcoBw/2-4F6CbaOo2BQuB174_dOtWx3I0YnaBTwVXEsUL-lzc?dl=0&preserve_transparency=1&size=2048x1536&size_mode=3)



It has a list of todos that are open or completed. The todo can be completed or deleted using the action icons present against each todo item. The text field below and the button can be used to add a new todo item.

Let's get started with installing Cypress and automating adding a todo item to the list. To get started, the code for the app is available to download and run:

```bash
$ git clone git@github.com:rcdexta/cypress-todo-example.git
$ cd cypress-todo-example
$ yarn install
```

When you are done installing all the dependencies, run `yarn start` to run the application on the browser to make sure it looks good. The same application is also available in [codesandbox](https://codesandbox.io/s/4j5m27o8ox) if you have trouble running it on your machine.

Now, let's bootstrap cypress and write our first test. With the application running, open another terminal session and run the following commands:

```bash
$ yarn add --dev cypress
$ yarn cypress open
```

Cypress will sense that you are running it for the first time and create a folder called `cypress` with necessary files to get you going. It would also launch the cypress test runner and think of it as a GUI for running/debugging automation specs.

We will create a file called `todo_spec.js`  in `cypress/integration` folder. 

> Note: You should already see an `example_spec.js `file that contains tests for the sample [Kitchen Sink](https://example.cypress.io/)  application with plenty of documentation for various scenarios. Would recommend reading through the file at a later point in time to for reference

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

The test should be pretty self-explanatory. `cy` is a global cypress object that drives the tests. It has various helper methods to visit web pages, interact with web elements and also to fetch data present in the DOM. 

Cypress has excellent [documentation](https://docs.cypress.io/guides/overview/why-cypress.html) and a [sample application](https://example.cypress.io/) that you would be frequently referring to as you add more test cases to your application. Cypress also bundles [chai](https://docs.cypress.io/api/commands/should.html#Syntax) assertion library and the should matcher you see in the test is chained to the cypress helper methods seamlessly for readability.

## Using the Cypress Test Runner

When you have saved the`todo_spec.js` file, it will start appearing in the Cypress test runner. When you click on todo_spec.js, a new browser instance will open up and run the test visually and you can cypress hop through each step that we  wrote earlier in out add todo item test. Check out a snapshot of the browser as it runs the test:

![cypress-gui-runner](https://photos-1.dropbox.com/t/2/AAAJwG6srLKNZ5lAl2-bo0oRFxfBNebg8IpOEqhXECALDA/12/5183783/png/32x32/3/1525341600/0/2/cypress_gui_runner.png/ENfW8AMY8qIBIAcoBw/a3Z6HXWm6czpCPeUhBGnP7KFJEzeykdwOzJ0Mz-OGuo?dl=0&preserve_transparency=1&size=2048x1536&size_mode=3)

The left pane is called the `Command Log` and lists the name of the test followed by each step in sequence and its outcome. When the test runs, you can see Cypress interacting with the application on the browser webview to the right and we see that the new todo item was added to the list successfully. And the last step on the left is the assertion to verify if our new item is present in the list and voila!

One very helpful feature on the browser view is the DOM selector that is shown highlighted below the URL address bar. It will appear when you click on the `Open Selector Playground` icon to the left of the address bar. You can type in a selector in the textbox like we have entered `#addBtn` or you can select an element from the screen and Cypress will try to find the optimal selector for you to use in your test. 

## Setting up Continuous Integration

The whole point of writing and maintaining browser-based automation tests is to test the application for any flows that break and give early feedback to the team. You can set up the suite of cypress tests to run on your continuous integration machine to run on every check-in or as a nightly build. We will quickly walk through the steps to set up the build and we will keep it agnostic of any particular CI tool.

Most CI build agents are servers and not workstations, so we cannot expect them to have a GUI or X Window system to launch a real browser UI. So, it is a common practice to run web automation tests in [headless mode](https://en.wikipedia.org/wiki/Headless_browser). To keep it simple, we would be using a virtual GUI like [xvfb][http://elementalselenium.com/tips/38-headless] to fire a web browser virtually and run the target application and cypress tests against it.

Assuming, you are setting up a docker machine on the CI server to run the tests, this is the specification for the`Dockerfile`

```dockerfile
FROM cypress/base

// Clone your code base here

RUN npm install
RUN $(npm bin)/cypress run
```

Refer to this [page](https://docs.cypress.io/guides/guides/continuous-integration.html#) for setting up Cypress on your favorite CI server. 

## Time Travel and Debugging 

Another cool feature present in the Cypress Test Runner tool is that as you hover through the command log on the left, for each step in the test, you can check the state of the application on the right. You can also click on the pin icon to freeze the runner for that particular step, open the developer console and get insights into the specific web interactions that happened, the selectors used, parameters passed and results.

![time-travel-gif](https://dl.dropboxusercontent.com/content_link/PfeFUk1RayIMNrG8bzZJaLHhE5FaeHHYfl7RJnt6XTXIPPrNSeO9Le2iGRuSxHTY/file?dl=0&duc_id=oPAPaaIYSZgm7bQk067E2mVpCwhXZKDHSoOtLhloSnODw3xQQpWdwgJyKLV6DPeS&raw=1&size=2048x1536&size_mode=3)

## Postmortem analysis with screenshots and videos

The last thing we want with a browser automation test is false positives. By that, a test case failing because of the way the automation test was written and not because something in the actual application is breaking. When that frequently happens, the test suite loses its credibility and a red build is not worth its attention anymore! 

So, when a test fails on the CI we need enough trails and artifacts to understand what really went wrong. Like most automation frameworks in the market, Cypress generates screenshots and videos of the entire test runs too if configured correctly. All of these settings can be tweaked using the `cypress.json` configuration file generated in the root of your project. By default, screenshots and video recording are turned on and you can see them generated inside the `cypress` folder after running the tests.

Check out the [configuration documentation](https://docs.cypress.io/guides/references/configuration.html#Videos) for additional settings and options.

 