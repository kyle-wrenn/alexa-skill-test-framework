# Alexa Skill Test Framework

This framework makes it easy to create full-coverage black box tests for an Alexa skill using [Mocha](https://mochajs.org/).

## How To
Install the package as a dev dependency with `npm install alexa-skill-test-framework --save-dev`. //TODO: add to npm

Write tests in a Javascript file and run them with Mocha. For example, if your test is at 'test/index.js', run `mocha test/index.js`.

For some simple examples, see the 'examples' directory.

## Test Framework Documentation

### alexaTest.initialize(index, version, appId, userId)
Initializes the test framework. Must be called before generating requests or running any tests.
* `index`: The object containing your skill's 'handler' method. Must define a method called `handler(event, context, callback)`, which runs the skill.
  * The test framework passes 'true' as a fourth parameter to the handler. Obviously this should be used sparingly, if at all.
* `appId`: The Skill's app ID. Looks like "amzn1.ask.skill.00000000-0000-0000-0000-000000000000".
* `userId`: The Amazon User ID to test with. Looks like "amzn1.ask.account.LONG_STRING".

### alexaTest.initializeI18N(resources)
Initializes i18n. You only need this if you use i18n in your skill, and you want to use i18n to fetch result strings to test against. You must have installed the optional dependencies `i18n` and `i18next-sprintf-postprocessor`.
* `resources`: The same resource object you pass to i18n inside your skill.

### alexaTest.setLocale(locale)
Changes the locale used by the test framework and the skill. Default is 'en'.
* `locale`: A string representing the locale to use.

### alexaTest.getLaunchRequest([locale])
Returns a [LaunchRequest][launchrequest docs]. The request can be passed to `test` (see below).
* `locale`: Optionally, an override locale for the request.

### alexaTest.getIntentRequest(intentName, [slots], [locale])
Returns an [IntentRequest][intentrequest docs]. The request can be passed to `test` (see below).
* `intentName`: The name of the intent to invoke.
* `slots`: Optionally, an object containing key-value pairs. The keys are the names of the slots required by the specified intent, and the values are the slot values.
* `locale`: Optionally, an override locale for the request.

### alexaTest.getSessionEndedRequest(reason, [locale])
Returns a [SessionEndedRequest][sessionendedrequest docs]. The request can be passed to `test` (see below).
* `reason`: The reason. See the [SessionEndedRequest][sessionendedrequest docs] documentation.
* `locale`: Optionally, an override locale for the request.

### alexaTest.test(sequence)
Tests the skill with a sequence of requests and expected responses. This method should be called from inside a Mocha `describe` block.
* `sequence`: An array of requests to test. Each element can have these properties:
  * `request`: The request to run. Generate these with one of the above `getFooRequest` methods.
  * `says`: Optional String. Tests that the speech output from the request is the string specified.
  * `saysNothing`: Optional Boolean. If true, tests that the response has no speech output.
  * `shouldEndSession`: Optional Boolean. If true, tests that the response to the request ends or does not end the session.
  * `saysCallback(context, speech)`: Optional Function. Recieves the speech from the response as a parameter. You can throw nice assertions using `context.assert` and get text from i18n with `context.t`.
  * `callback(context, response)`: Optional Function. Recieves the [response][response examples] object from the request as a parameter. You can throw nice assertions using `context.assert` and get text from i18n with `context.t`.

### alexaTest.t(arguments)
Forwards the request to `alexaTest.i18n.t` and returns the result. You must have called `alexaTest.initializeI18N` previously.

## CallbackContext Documentation
Callback context objects are passed to `callback` and `saysCallback` in tests.

### context.assert(data)
Throws an assertion error.
* `data` Object holding data for the error. Can include:
  * `message`: Optionally, a string describing the failure.
  * `expected`: Optionally, the expected value.
  * `actual`: Optionally, the actual value.
  * `operator`: Optionally, the comparison operator that was used.
  * `showDiff`: Optionally, true if Mocha should diff the expected and actual values.

### context.t(arguments)
Forwards the request to `alexaTest.i18n.t` and returns the result. Additionally, ensures the language is the language used in the request. You must have called `alexaTest.initializeI18N` previously.

## Note About DynamoDB
If your skill uses the [Alexa Skills Kit for Node.js][nodekit] and uses its built-in DynamoDB persistence, it may be desireable not to connect to the database during testing, since the ASK Node kit uses it to persist session attributes. For that case, the framework passes a fourth parameter of `true` to the `handler` function, which you can conditionalize on to disable the database connection.

[sessionendedrequest docs]: https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/custom-standard-request-types-reference#sessionendedrequest
[launchrequest docs]: https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/custom-standard-request-types-reference#launchrequest
[intentrequest docs]: https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/custom-standard-request-types-reference#intentrequest
[response examples]: https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/alexa-skills-kit-interface-reference#response-examples
[nodekit]: https://github.com/alexa/alexa-skills-kit-sdk-for-nodejs