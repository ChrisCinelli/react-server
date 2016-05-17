# What is a page?

A page is a javascript representation of [a hypertext document suitable for the
world wide web and a web browser](https://en.wikipedia.org/wiki/Web_page).
Pages should roughly match one-to-one to urls for a web site.  Pages have
lifecycle methods that are called on them by react-server, which produce the
html, either on the server or in the browser.  By convention, pages are written
as [classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes),
but at their core, pages are Javascript objects with keys named for page
lifecycle events, and which have corresponding functions that return elements,
rendered as React instances.

The simplest of pages only needs a `getElements` method

```js
export default class SimplePage {
	getElements () {
		return <h1>Hello react-server</h1>;
	}
}
```


# Understanding the page lifecycle

Page lifecycle methods are called in groups that can return asynchronously in
any order.  We render the head element first

First, we call the following methods together

- renderDebugComments
- renderTitle
- renderScripts
- renderStylesheets

Once `renderStylesheets` has completed, we call the following methods together

- renderMetaTags
- renderLinkTags
- renderBaseTag

All of which is then sent to the browser.  Next, we call the rest of the page
lifecycle is called in order

1. getBodyClasses
1. getBodyStartContent
1. getElements
1. getAboveTheFoldCount (until we've rendered all the above-the-fold content)

Once we reach the above the fold content, we'll start sending javascript.


# Writing page lifecycle methods

See all page lifecycle methods in [Writing Pages](/docs/writing-pages).

If you don't provide a page lifecycle method, we provide a "best-guess" default
value; for instance, you can omit the head methods altogether and get a
performant and featureful page.


# Examples

### Full page

A `react-server` page that serves a full webpage.

```js
import HttpStatus from 'http-status-codes';
import MobileEnabled from ('./middleware/MobileEnabled');
const ExampleComponent = require("./components/example-component");
const ExampleStore = require("./stores/example-store");
const exampleAction = require("./actions/example-action");

class ExamplePage {
	// See [writing middleware](/docs/writing-middleware) for how to write middleware
	static middleware() { return [MobileEnabled]; }

	handleRoute() {
		var params = this.getRequest().getQuery();
		this._exampleStore = new ExampleStore({
			id: +params.id
		});
	}

	getTitle() {
		return "Example page"
	}

	getHeadStylesheets() {
		return [
			"/styles/example.css",
			"/styles/reset.css"
		]
	}

	getMetaTags() {
		var tags = [
			{ name: "example", content: "Demonstrate a full react-server page" },
		];
		return tags;
	}

	getLinkTags() {
		return [
			// prefetch analytics to improve performance
			{ rel: "prefetch", href: "//www.google-analytics.com" },
		];
	}

	getBodyClasses() {
		return ["responsive-page", "typography"];
	}

	getElements() {
		return [
			<ExampleComponent handleOnClick={exampleAction} {...this._exampleStore} />
		];
	}
}
```

### Json endpoint

```js
// returns a promise for example data
const getExampleData = require("./helpers/get-example-data");

module.exports = class ExampleJsonPage {

	// see the example in [writing middleware](/docs/writing-middleware)
	static middleware() { return [JsonEndpoint] }

	handleRoute() {
		const id = this.getRequest().getRouteParams().id;
		this.data = getExampleData(id);
		return {code:200};
	}

	getResponseData() {
		return this.data;
	}
}
```

### Setting Config values

For instance, to make a page into a fragment by setting the `isFragment` config
value

```js
const exampleComponent = require("./components/example-component");
const exampleStore = require("./stores/example-store");

export default class ExampleFragmentPage {
	setConfigValues() { return { isFragment: true }; }

	handleRoute() {
		this._store = exampleStore(this._getStoreOpts());
		return {code:200};
	}

	getTitle () {
		return "Corvair School Fragment";
	}

	getElements() {
		return [
			RootElements.createRootElementWhenResolved(this._store, <exampleComponent/>),
		];
	}

	_getStoreOpts() {
		var storeOpts = {};
		var routeParams = this.getRequest().getRouteParams();
		if (routeParams) {
			storeOpts.exampleId = routeParams.exampleId;
		}
		return storeOpts;
	}
}

module.exports = ExamplePage;
```


# Find out more

Check the [page api](/docs/page-api) to learn more.  If you'd like to check the
code, the page lifecycle is declared at ll 265 of `renderMiddleware.js`; the
code is fairly well commented.

[//]: # see http://stackoverflow.com/questions/4823468/comments-in-markdown
[//]: # TODO: It would be nice to link to blog posts here, and links to a static
[//]: # [github page](https://gist.github.com/domenic/ec8b0fc8ab45f39403dd) with
[//]: # annotated source code using [docco](http://jashkenas.github.io/docco/)