# cypress101-tips-and-tricks
Welcome to a curated list of **101 Cypress tips and tricks** gathered from my own experience in UI and API automation. This guide is perfect for QA Automation engineers building scalable and maintainable Cypress frameworks

---

## ✅ Tips 1 - 7: Core Cypress practices
1. Integrate custom accessibility plugins for your framework, for instance `WICK-A11Y` Cypress open-source Accessibility plugin
2. Use IDE extensions (e.g., `VS Code Cypress Snippets`)
3. Use Cypress Cloud for analytics, reporting, and parallel runs, UI Coverage metrics, AI tool

### UI elements manipulation
4. Assert element state by verifying partial class name if nothing else available: `[class*="disabled-true"]` or `[class*="isError-true"]`
5. Find parent locator of a child element: `cy.get('#element').parent()`
6. Get element if it is out of visible screen zone: `cy.get('#element').scrollIntoView().click()`
7. Use `.should()` for UI web elements assertions: `cy.get('#element').should('have.attr', 'data-state', 'off')`

## ⚙️ Tips 8 - 30: Advanced Cypress practices

### Alert handling

8. Handle `window:alert` using `cy.on('window:alert')` for confirm action in browser alert window
```js
cy.on('window:confirm', (text) => {
  expect(text).to.equal('Are you sure?');
  return true; // Simulates clicking "OK"
});
```

9. Handle `window:alert` using `cy.on('window:alert')` for abort action in browser alert window
```js
cy.on('window:confirm', (text) => {
  expect(text).to.equal('Are you sure?');
  return false; // Simulates clicking "Cancel"
});
```

### Input and assertion tricks

10. Use `.clear({ force: true })` for clear input if it is not active input `cy.get('input[name="email"]').clear({ force: true }).type('test@example.com');`
11. Use RegEx in assertions for reliable text validation, e.g., price ranges - `expect(invokedNormalizedText).to.match(/Min:\s*\d+\sUSD\s*-\s*Max:\s*\d+\sUSD/);`

### Test tags and events

12. Use test tags to group test suites: 
```js
///testSuite.spec.js

describe("Regression Tests", { tags: ["regression"] }, () => {
    it("Should test something", () => {
        //Test code
    });
});
```

13. Use `.trigger()` to simulate browser events. Helps when you need to simulate events like drag-and-drop or custom actions that aren’t directly available through Cypress commands
```js
      cy.get('#paswordInput')
        .clear({ force: true })
        .type(phoneNumber)
        .trigger("input")
        .trigger("change");
```

### Configuration and environment setup

14. Modify `defaultCommandTimeout` in config for better test control: `defaultCommandTimeout: 40000`
15. Modify `requestTimeout` / `responseTimeout` in config for network-heavy web applications: `requestTimeout: 20000` or `responseTimeout: 20000`
16. Create multiple config files for testing different states/environments `cypress.stage.config.js` then run it with `npx cypress run --config-file cypress.stage.config.js --headless`
17. Create separate file with environment specific URLs for testing different environments by using CLI flag `--env url=stage`
18. Use `cy.viewport()` for testing mobile responsiveness - `cy.viewport('iphone-16')`
19. Use `Cypress.env()` to manage environment-specific variables with combination of environment specific URLs

```js
// environmentURLs.js

const URL = {
  stage: {
    homePage: "https://myhomepage.stage.com",
  },
  dev: {
    homePage: "https://myhomepage.dev.com",
  }
}

export default URL;
```

```js
// Get environment variable and default to 'stage' if none is provided
const environment = Cypress.env("url") || "stage";
// Construct environment specific URL object
const envConfig = URL[environment];
// Construct environment specific the API URL based on the environment
const homePageURL = envConfig.homePage;
```

20. Use npm scripts to run Cypress tests efficiently in CLI `npm run cypress:tests:cloud:stage`
```js
// package.json
  "scripts": {
    "cypress:tests:cloud:stage": "npx cypress run --config-file cypress.stage.config.js --headless --record --key {{CYPRESS_CLOUD_PROJECT_KEY}} --env url=stage",
  },
```

## 🔄 Tips 31 - 38: Advanced command chaining & flow control

21. Use `cy.should('be.visible').and('include.text', 'Success')` chaining for multiple assertions
22. Use `cy.clock()` to control time-based logic (timers, intervals, polling logic)
23. Use `cy.tick()` to simulate time passing instantly
```js
cy.clock();
cy.get('#toast-btn').click();
cy.tick(3000); // Simulates the 3 seconds instantly
cy.get('.toast').should('not.exist');
```

24. Chain `.then()` to access values dynamically 
```js
cy.get('#user-email').invoke('text').then((email) => {
  expect(email).to.include('@');
});
```

25. Iterate over DOM with `.each()` 
```js
cy.get('.product-item').each(($el) => {
  cy.wrap($el).should('be.visible');
});
```

26. Wrap external values using `cy.wrap()`
```js
const user = { name: 'John' };
cy.wrap(user).its('name').should('eq', 'John');
```

27. Use `.spread()` for multiple aliases
```js
cy.get('.user-name').as('name');
cy.get('.user-age').as('age');

cy.get('@name').then((nameElement) => {
  cy.get('@age').then((ageElement) => {
    cy.wrap([nameElement.text(), ageElement.text()]).spread((name, age) => {
      expect(name).to.not.be.empty;
      expect(Number(age)).to.be.greaterThan(0);
    });
  });
});
```

28. Invoke jQuery functions with `.invoke()`
```js
cy.get('#element').invoke('val').should('eq', 'expectedValue');
```

29. Use `.its()` to access nested properties
```js
cy.window().its('navigator.language').should('eq', 'en-US');
```

30. Alias values with `.as()` to get them later in your test
```js
cy.get('#user-profile').as('profile');
cy.get('@profile').should('be.visible');
```

31. Use `Cypress._` for a helpful JavaScript library - Lodash that makes it easier to work with arrays, objects, and data.
```js
const items = [1, 2, 3];
const doubled = Cypress._.map(items, (i) => i * 2);
expect(doubled).to.deep.equal([2, 4, 6]);
```

32. Use `Cypress.Promise` if application or test logic requires asynchronous behavior
```js
return new Cypress.Promise((resolve) => {
  setTimeout(() => {
    resolve('done');
  }, 1000);
});
```

33. Use `.retry()` plugin for custom retry logic
```js
cy.get('.status').then(($el) => {
  const checkStatus = () => {
    if ($el.text() !== 'Ready') {
      setTimeout(checkStatus, 500);
    } else {
      expect($el.text()).to.equal('Ready');
    }
  };
  checkStatus();
});
```

34. Manually retry assertions within `.then()`
```js
cy.get('#user').then(($el) => {
  cy.log('User element found:', $el.text());
});
```

35. Use `cy.origin()` for cross-origin testing (Cypress 12+): login on external auth pages, interact with payment gateways, work with widgets hosted on other domains
```js
// Start from your main app
cy.visit('http://localhost:3000');
// Click the login button, which redirects to a different domain
cy.get('#login-button').click();
// Now we enter the new origin block
cy.origin('https://auth.example.com', () => {
  // These commands run inside the login page domain
  cy.get('#username').type('testuser');
  cy.get('#password').type('testpass');
  cy.get('#submit').click();
});
// After login, you're back on your main app domain
cy.url().should('include', 'dashboard');
```

36. Scope actions with `.within()`. Inside `.within()` all `cy.get()` commands will search only inner elements, not the whole page. It is faster.
```js
cy.get('#login-form').within(() => {
  cy.get('input[name="email"]').type('user@example.com');
  cy.get('input[name="password"]').type('123456');
});
```

37. Avoid fixed waits `cy.wait(1000)` with `.wait('@alias')` - network request/response to finish.
```js
cy.intercept('/api/user').as('getUser');
cy.visit('/dashboard');
cy.wait('@getUser'); // ✅ wait only until the request finishes
```

38. Use `cypress-if` plugin for conditional logic
```js
cy.get('button#subscribe')
  .if('exists')                // Check if the button exists
  .click();                    // Click only if it does
```

---

## 🧪 Tips 39 - 56: API testing techniques
39. Intercept API calls and wait for API response to assert UI element without `cy.wait(ms)`, `cy.wait("@aliasName')`
40. Use `expect` for API request/response assertions `expect(response.statusCode).to.eq(200);`
41. Use `cy.task` to operate with Node (e.g., store or fetch values): `cy.task("getItem", "bearerUserToken");`
42. Pass headers in `cy.request()` when needed:

```js
cy.request({
    method: GET,
    url: yourURL,
    headers: {
        Authorization: `Bearer ${bearerUserToken}`,
    }, 
});
```

43. Write custom commands for REST API calls and then use that as a templates for REST API testing
44. Auto-generate GraphQL commands via introspection. Write custom commands for GraphQL queries and (or) mutations and then use that as a templates for GraphQL testing
45. Manage tokens for user sessions (e.g., login via API)
46. Test real-time events using WebSockets (e.g., SignalR)
47. Validate schemas with Cypress plugin `AJV-SCHEMA-VALIDATOR` or `zod`
48. Chain `cy.request()` calls for API workflows
49. Assert cookies, headers, and status codes
50. Prepare data or cleanup using API
51. Chain UI and API tests for full coverage
52. Mock API failures (401, 500) to test error states
53. Use `cy.log()` inside commands to improve readability
54. Extract dynamic values from API responses
```js
cy.request('/api/users').then((response) => {
  const userId = response.body.data[0].id;
  expect(userId).to.exist;
});
```

55. Load mock data using `.fixture()`
```js
cy.fixture('user.json').then((user) => {
  expect(user.name).to.equal('Jane');
});
```

56. Use `Wiremock` for API mocking

---

## 🧠 Tips 57 - 66: Debugging & testing strategy

57. Use `debug()` and `pause()`, `cy.stop()` to investigate issues
58. Use `console.log()` inside `.then()` 
59. Set retries using `Cypress.config('retries', 2)`
60. Follow Arrange → Act → Assert structure
61. Test file upload with `.selectFile()`
62. Use `cy.viewportPreset()` or custom sizes
63. Use page object pattern for test architecture
64. Group tests using `describe()` and `.only`
65. Use `.skip()` or conditional `it()`
66. Use `beforeEach()` to handle shared setup

---

## 📦 Tips 67 - 77: DOM, forms, and user interaction

67. Simulate typing using `.type({ delay: 100 })`
68. Test downloads using `cypress-downloadfile` plugin
69. Use `should('be.visible')` vs `should('exist')`
70. Force click hidden elements: `.click({ force: true })`
71. Use `.scrollTo('top'/'bottom')` to navigate view
72. Use `.check()` and `.uncheck()` for checkboxes
73. Use `.select()` for dropdowns
74. Assert form validations dynamically
75. Use `cy.clock()` and `.type()` for date pickers
76. Iterate and click multiple elements with `.each()`
77. Use `cy.submit()` for forms

---

## 🛡️ Tips 76 – 85: Auth & session management techniques

78. Use `cy.session()` to cache login sessions
79. Login via API and store token in `localStorage`
80. Clear `localStorage` / `sessionStorage` before each test. By default, Cypress built-in mechanism does it
81. Use `cy.setCookie()` / `cy.clearCookies()` for cookie handling
82. Validate cookies with `cy.getCookie()`
83. Test multi-role access by switching tokens
84. Verify unauthenticated redirects
85. Simulate session with `cy.window().then(win => win.sessionStorage.setItem(...))`
86. Store credentials securely with `Cypress.env()`
87. Skip login UI flow entirely using `cy.request()`

---

## 📊 Tips 88 – 96: CI, cloud & reporting

88. Use `mochawesome` for advanced reporting
89. Enable video recording for debugging
90. Use any CI/CD provider to run your tests automatically: GitHub Actions, GitLab CI, Jenkins, Argo Workflows, etc.
91. Filter tests using `cypress-grep` plugind by tag or regex
92. Send test status to Slack via webhook or via Slack integration in the Cypress Cloud
93. Capture custom screenshots on failure only
94. Run tests in parallel with Cypress Cloud
95. Set `baseUrl` in config or CLI for multi-env testing
96. Use `start-server-and-test` for one-liner dev+test runs

---

## 🧩 Tips 97 – 101: Final touches

97. Test localizations with query param or language switch
98. Simulate feature flags and toggle behavior in tests
99. Integrate with Percy or Applitools for visual testing
100. Create Cypress boilerplate repo for team use
101. Keep changelog for Cypress updates & breaking changes. And always read the [Cypress Changelog](https://docs.cypress.io/guides/references/changelog) to stay up to date

---

## 💬 Bonus: Slack notification integration (Cypress Cloud subscription required)

### Report Cypress test results to Slack in 4 simple Steps:

1. **Create a Cypress Cloud Project**
   - Go to [Cypress Dashboard](https://cloud.cypress.io) and create a project.

2. **Update `cypress.config.js` with projectId:**
```js
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  projectId: 'your-project-id',
});
```

3. **Create a Slack channel**
   - Create a dedicated channel (e.g., `#cypress-tests`) in Slack.

4. **Connect Slack via Cypress Cloud Integrations**
   - In Cypress Cloud: Go to your project → `Settings` → `Integrations` → Add Slack → Select your channel.

Done! You’ll now receive test run results right in Slack 🚀

---

Contributions welcome! Feel free to suggest new tips via issues or pull requests 🤝
