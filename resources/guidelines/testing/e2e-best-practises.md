# Best practices on writing end-to-end tests

A typical E2E test can be complex, with many steps that take a lot of time to complete manually. Because of this complexity, E2E tests can be difficult to automate and slow to execute. The following tips can help reduce the cost and pain of E2E testing and still reap the benefits.

Cypress got you covered with their best practices as well: So please also look at their best practices to get to know their patterns:

{% embed url="https://docs.cypress.io/guides/references/best-practices.html" %}

{% hint style="warning" %} We strongly recommend to follow Cypress' own best practices as well. {% endhint %}

## Amount and prioritisation of end-to-end tests

### Video

When it comes to dividing test types, selecting and prioritizing test cases, and thus designing tests, things get a bit more complicated. We have generally aligned our test strategy with the test pyramid, although not 100%. The pyramid states that end-to-end tests should be written in a few, but well-chosen test cases - because end-to-end tests are slow and therefore expensive.

At [Shopware Community Day](https://scd.shopware.com/en/) 2020, we gave a talk on how we approach automated testing in Shopware, how far we have come on this journey, and what we have gained so far:

{% embed url="https://www.youtube.com/watch?v=sxvQoWF4KS0" %}

To sum it up briefly for your End-to-End testing context, End-Tto-End tests are the one tests being slow and thus expensive to maintain. That's why we need a way to prioritize our test cases.

### When should I write an end-to-end test?

{% hint style="danger" %} Cover every possible workflow with E2E tests. {% endhint %}

{% hint style="success" %} Use proper prioritisation to choose test cases covered by E2E tests. {% endhint %}

You see, due to running times it is not advisable to cover every single workflow available. The following criteria may help you with that:
* **Cover the most general, most used workflows of a feature**, e.g. CRUD operations. The term "[happy path](https://en.wikipedia.org/wiki/Happy_path)" describes those workflows quite well.
* **Beware the critical path**: Cover those workflows with E2E tests, which are most vulnerable and would cause most damage if broken.
* **Avoid duplicate coverage**: E2E tests should only cover what they can cover, usually big-picture user stories \(workflows\) that contain many components and views.
  * Sometimes unit tests are suited better: For example, use an E2E test to test your application's reaction on a failed validation, not the validation itself.

## Workflow-based end-to-end tests

{% hint style="danger" %} Write E2E test as you would write unit tests. {% endhint %}

{% hint style="success" %} Write E2E tests in a "workflow-based" manner, that means writing the test describing a reals
user's workflow - just like a real user would use your application.{% endhint %}

A test should be written "workflow-based" -  We like to use this word very much, because it is simply apt for this purpose. You should always keep your persona and goal of an E2E test in mind: The test is then written from the user's point of view, not from the developer's point of view.

## Structure and scope

### Test scope

{% hint style="danger" %} Write long E2E tests, covering lots of workflows and use cases. {% endhint %}

{% hint style="success" %} Keep tests as simple as possible! Only test the workflow you explicitly want to test - ideally use **one test for one workflow**. {% endhint %}

The second most important thing is to just test the workflow you explicitly want to test: Any other steps or workflows to get your test running should be done using API operations in the `beforeEach` hook, as we don't want to test them more than once.
For example: if you want to test the checkout process you shouldn't do all the steps like create the sales channel, products and categories although you need them in order to process the checkout. Use the API to create these things and let the test just do the checkout.

You need to focus on the workflow to be tested to ensure minimum test runtimes and to get a valid result of your test case if it fails. Fot this workflow, you have to think like the end-user would do: Focus on usage of your feature, not technical implementation.

Other examples of steps or workflow to cut off the actual tests are:
* The routines which should only provide the data we need: Just use test fixtures to create this data to have everything available before the test starts.
* Logging in to the Administration: You need it in almost every Administration test, but writing it in all tests is pure redundancy and way more error sensitive.

{% hint style="info" %} This [scope practice](https://docs.cypress.io/guides/references/best-practices.html#Organizing-Tests-Logging-In-Controlling-State) is also mentioned in Cypress' best practices as well.
{% endhint %}

### Focus on stability first!

{% hint style="danger" %} Design your tests dependent on each other, doing lots of write operations without removing corresponding data. {% endhint %}

{% hint style="success" %} Keep tests isolated, enable them to run independently and restore a clean installation between tests {% endhint %}

Don't ever rely on previous tests! You need to test specs in isolation in order to take control of your application’s state. Every test is supposed to be able to run on its own and independent from any other tests. This is crucial to ensure valid test results. You can realize this using test fixtures to create all data you need beforehand and taking care of the cleanup of your application using an appropriate reset method.

## Choosing selectors

{% hint style="danger" %} Choose fuzzy selectors which are prone to change, e.g. xpath. {% endhint %}

{% hint style="success" %} Use selectors which won't change often. {% endhint %}

XPath selectors are quite fuzzy and rely a lot on the texts, which can change quickly. Please avoid using them as much as possible. If you work in Shopware platform and notice that one selector is missing or not unique enough, just add another one in the form of an additional class.

## Waiting in E2E tests

{% hint style="danger" %} Waiting for arbitrary time periods, e.g. using `cy.wait(500)` {% endhint %}

{% hint style="success" %} Use route aliases or assertions to guard Cypress from proceeding until an explicit condition is met. {% endhint %}

Never use fixed waiting times in the form of `.wait(500)` or similar. Using Cypress, you never need to do this. Cypress has a built-in retry-ability in almost every command, so you don't need to wait e.g. if an element already exists. If you need more than that, we got you covered: Wait for changes in the UI instead, notification, API requests, etc. via the appropriate assertions. For example, if you need to wait for an element to be visible:

```javascript
cy.get('.sw-category-tree').should('be.visible');
```

Another useful way for waiting in the Administration is using Cypress' possibility to work with [network requests](https://docs.cypress.io/guides/guides/network-requests.html). Here you can let the test wait for a successful API response:

```javascript
cy.server();

// Route POST requests with matching url and assign an alias to it
cy.route({
    url: '/api/v3/search/category',
    method: 'post'
}).as('getData');

// Later, you can use the alias to wait for the API response
cy.wait('@getData').then((xhr) => {
    expect(xhr).to.have.property('status', 200);
});
```

{% hint style="info" %} This [best practice](https://docs.cypress.io/guides/references/best-practices#Unnecessary-Waiting) is also mentioned in Cypress' best practices as well. Actually, it can be considered as general best practice to avoid flakiness.
{% endhint %}