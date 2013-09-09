# Promise-Returning Tests for Jasmine

Extension of Jasmine Spec::runs() to support Promises and automagically wait for the promise to resolve or reject.

Until now you've been making do with manual solutions that explicity use `runs()` and `waitsFor()` to force the test runner to pause
the tests in order to wait for the async response. like this:

```js
it( "should be fulfilled with 5", function () {
	var ready  = false,
		result;
		
		// Run the async yourAsyncCall() method...					
		runs( function() {
			yourAsyncCall().then(
				function onResponse( data )
				{
					result = data;
					ready  = true;		// continue test runner
				},
				function onError( fault )
				{
					ready  = true;		// continue test runner
				}
			);
		});

		// Pause test runner until timeout or yourAsyncCall() responds		
		waitsFor( function() {
			return result;
		});
		
		
		// Run the code that checks the expectations…
		
		runs( function() {
			expect( result ).toBeEqual( 5 );
		});	    
});
```

## Better Solution

So you really like [Jasmine](). But you also really like [promises](). And you'd like to see
support in [Jasmine]() for the promise-returning test style found in [Mocha as Promised]() and others.

Consider the simplicity of code achieved now when your unit tests return Promises:

```js
it( "should be fulfilled with 5", function () 
{
	// NOTE: yourAsyncCall() returns a Promise

	runs( function() {    	
		return yourAsyncCall()
			      .then( function (result) 
			      {
				      expect( result ).toBeEqual( 5 );
			      });
	});
});
```

You could even separate your `expect()` calls if wanted. Instead of nesting your expectations inside
the promise handler, consider another supported approach:

```js
it( "should be fulfilled with 5", function () 
{
	// NOTE: yourAsyncCall() returns a Promise

	runs( 
		function() 
		{    	
			return yourAsyncCall();
		},
		function checkExpectations( result ) 
		{
			expect( result ).toBeEqual( 5 );
		}
	);
});
```

Now, with Jasmine as Promised, you have a much nicer option available! No more need to worry about waitsFor(), etc.


## How to Use

Once you install and set up Jasmine as Promised, you now have a second way of creating asynchronous tests, besides Jasmine's
usual `runs(); waitsFor(); runs();` style. Just return a promise: if it is fulfilled, the test expectations are checked, and if it is rejected, the test
fails, with the rejection reason as the error. Nice, huh?

Jasmine as Promised works with all Jasmine interfaces: BDD, TDD, QUnit, whatever. It hooks in at such a low level, the
interfaces don't even get involved.

## Installation and Usage

### Node

Do an `npm install jasmine-as-promised --save-dev` to get up and running. Then:

```js
require("jasmine-as-promised")();
```

You can of course put this code in a common test fixture file; for an example, see
[the Jasmine as Promised tests themselves][fixturedemo].

### AMD

Jasmine as Promised supports being used as an [AMD][amd] module, registering itself anonymously. So, assuming you have
configured your loader to map the Jasmine and Jasmine as Promised files to the respective module IDs `"jasmine"` and
`"jasmine-as-promised"`, you can use them as follows:

```js
define(function (require, exports, module) {
    var jasmine = require("jasmine");
    var jasmineAsPromised = require("jasmine-as-promised");

    jasmineAsPromised(jasmine);
});
```

### `<script>` tag

If you include Jasmine as Promised directly with a `<script>` tag, after the one for Jasmine itself, then it will
automatically plug in to Jasmine and be ready for use:

```html

<script src="jasmine"></script>
<script src="jasmine-as-promised.js"></script>

```

### Node, the Advanced Version

The `require("jasmine-as-promised")()` above tries to detect which instance of Jasmine is being used automatically. This
way, Jasmine as Promised can plug into either the local Jasmine instance installed into your project, or into the global
Jasmine instance if you're running your tests using the globally-installed command-line runner.

In some cases, if you're doing something weird, this can fall down. In these cases, you can pass the Jasmine instance into
the Jasmine as Promised function. For example, if you somehow had your Jasmine module as a property of the `foo` module,
instead of it being found in the usual npm directory structures, you would do

```js
require("jasmine-as-promised")(require("foo").MyJasmine);
```

## How Does This Work!?

This is a reasonable hack.

Note that Jasmine as Promised  just overrides the `Jasmine.Spec.prototype.runs` method.
