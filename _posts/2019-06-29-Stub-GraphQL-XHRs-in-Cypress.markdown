---
layout: post
title: Stub GraphQL XHRs in Cypress
date: 2019-06-28 10:44:07 +0800
categories: ['HTML', 'Testing']
---

Cypress is an amazing UI testing tool.

One problem though, is that it does not support the stubbing of GraphQL API calls.

For a majority of GraphQL users who use GraphQL clients such as Apollo, a potential workaround is to stub the JavaScript Fetch object in the browser window. This works because Apollo, by default, uses the Fetch API to execute requests. If that is your issue, then there are well documented workarounds on this [Github page](https://github.com/cypress-io/cypress-documentation/issues/122). If you read through the thread, you will even find that one of the Cypress engineers created [a plugin specially for this to run a mock GraphQL server](https://github.com/tgriesser/cypress-graphql-mock).

### We need to stub GraphQL XHRs

However, it didn't meet our use case. Our team chose to use the AWS Amplify library to connect to the AWS AppSync GraphQL service. Out of the box, it provides the [API.graphql](https://aws-amplify.github.io/docs/js/api) method to make queries. For React users (that's us!), there's even a pre-written Connect React component that abstracts this call for you. Sweet! Except that API.graphql just makes an Axios call, and Axios of course uses XHR.

So we needed to stub XHR requests. We have a problem thouh: Cypress cannot stubbing a HTTP endpoint/ URL to return multiple responses. See [the issues](https://github.com/cypress-io/cypress/issues/521).

We typically made multiple GraphQL requests on a single page load - some for data, some of user info, etc. How would we be able then to return different responses using the same endpoint? Cypress.route wasn't working because all the same route responses would only return 1 response, at any given point in time.

We couldn't find a native method in Cypress APIs that would allow us to inspect the XHR's request body somewhere during its lifecycle before it went "inflight" and got intercepted by Cypress' interceptors. And this is critical as we distinguish GraphQL requests by their body.

### Solution: Custom code to intercept the XHR lifecyle

Enter [xhook](https://github.com/jpillora/xhook), a wonderful library swaps out the global XMLHttpRequest on the browser window and lets you place listeners before the XHR request goes "inflight", i.e. before [XMLHttpRequest.send()](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/send) is called.

So we ended with up with this code:

```typescript
let xHookPackage;
let graphqlEndpoint='XXXXXXX';
let mockGraphQLResponsesMap = {
    queryA: {} // some mock response
    queryB: {} // another mock response
}

before(() => {
	const xHookUrl = 'https://unpkg.com/xhook@latest/dist/xhook.min.js';
	cy.request(xHookUrl)
		.then(response => {
			xHookPackage = response.body;
		});
});

Cypress.on('window:before:load', win => {
	// load the library in the cypress window, creates a 'xhook' object on the Window
	win.eval(xHookPackage);
	// tap into the .before() method
	win.xhook.before(req => {

		if (req.method === 'POST' && req.url === graphqlEndpoint) {
			const graphqlQuery = JSON.parse(req.body).query;
			// example:  "query GetAuthUser($cognito_id: String!) { getAuthUser(....", we want the first part
			const graphqlQueryString = graphqlQuery.split('(')[0];
			const graphQLOperationType = graphqlQueryString.split(' ')[0];
			const graphQLOperationName = graphqlQueryString.split(' ')[1];
			console.warn(
				`Stubbing graphQLOperationType ${graphQLOperationType} and graphQLOperationName ${graphQLOperationName}`
			);

			const mockResponseObject = mockGraphQLResponsesMap[graphQLOperationName]; // no need to stringify
			return {
				status: 200,
				text: mockResponseObject,
			};
		}
	});
});
```

For an in-depth explanation of how this works, stay tuned!
