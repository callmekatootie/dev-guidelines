# Nodejs Best Practices Guide
This guide is aimed at the Topcoder community. In the interest of using the same yardstick when reviewing contest submissions, this guide has been created to capture some best practices that improve the quality of submissions and make it easier to review, integrate and deploy the submissions. The expectation is that reviewer and submitter alike follow these guidelines.

This guide does not claim to be comprehensive or aim to be perfect. With the ever changing technology, especially in the JavaScript world, this document will have to be periodically updated to reflect the current trends and adapt to the latest conventions. It is expected that the Topcoder community actively contributes in improving this documentation and helps the platform maintain a high standard of code.

## Table of Contents
1. [Nodejs Version](#nodejs-version)
2. [Use of npm scripts](#use-of-npm-scripts)
3. [Dependencies vs DevDependencies](#dependencies-vs-devdependencies)
4. [ES6](#es6) (ES2015)
5. [Linter](#linter)
6. [App Configuration](#app-configuration)
7. [HTTP Status Codes](#http-status-codes)
8. [401 vs 403](#401-vs-403)
9. [Avoid callback hell](#avoid-callback-hell)
10. [Logging](#logging)
11. [Error handling](#error-handling)
12. Data validation
13. Folder structure
14. Build tools

## Nodejs Version

 - Unless otherwise specified in the contest forum, the nodejs version to be used will be version 6. The latest minor version of this release will be used.
 - If you are working on an existing app, you can change the version of nodejs used in the app only if the contest requirements ask that of you or if the co-pilot / PM confirms that the nodejs version can be changed.
 - You are required to specify both the nodejs and npm versions in the `package.json` file as follows:
    ```javascript
    // Sample package.json
    {
        "engines": {
            "node": "6.6.0",
            "npm": "3.10.3"
        }
    }
    ```

    This lets the end user know which node version is needed for the app to run successfully. Additionally, this is necessary when deploying the app on heroku, which is usually the preferred hosting platform when evaluating submissions or demonstrating submissions to clients

[[Go Top](#nodejs-best-practices-guide)]

## Use of npm scripts

 - Starting the app? Populating the database with test data? Running tests? Lint checks? Building the app? Use npm scripts to do all of these. You need to write scripts using nodejs itself and then use npm scripts to execute them.
 - Check out the [documentation](https://docs.npmjs.com/misc/scripts) to know more.
 - Instead of requiring the user to run `npm install` followed by `bower install`, use the `postinstall` script to run `bower install` automatically once the `npm install` command completes execution
 - Reduce the number of commands that the user has to execute in order to carry out an action through npm scripts.
 - For example, if there are separate scripts to run tests on the front end and back end, you could provide three commands as follows:
```javascript
// Sample package.json
{
    "scripts": {
        "testFrontEnd": "node tests/frontEnd.js",
        "testBackEnd": "node tests/backEnd.js",
        "test": "npm run testFrontEnd && npm run testBackEnd"
    }
}
```
Thus, with this approach, you offer the user the flexibility of running tests just for the front end or back end or running tests for the overall app.

[[Go Top](#nodejs-best-practices-guide)]

## Dependencies vs DevDependencies

 - The decision on whether to include a package in `dependencies` or `devDependencies` used to be quite clear a couple of years back when `npm` was used only for managing a nodejs based app's dependencies.
 - However, multiple libraries (particularly the front end ones) started using both `bower` and `npm` to allow the users to install these libraries. This resulted in certain libraries which normally would only be needed during development phases (such as build tools) to actually be defined in `dependencies` instead of `devDependencies`.
 - The easiest way to determine if a module / library needs to be defined in `dependency` vs `devDependency` is to deploy the app to a cloud based provider (such as [heroku](https://www.heroku.com/)) and use it as a metric. Having all your modules in `dependencies` is bad if that module is not needed to deploy the app.

[[Go Top](#nodejs-best-practices-guide)]

## ES6

 - There are plenty of articles on the web that go into details of ES6 and thus this guideline will not bring those up again.
 - You are expected to use the ES6 features that nodejs v6 offers.
 - Unless otherwise mentioned, do not use Babel or any other transpiler with nodejs. You are encouraged to use the features that nodejs v6 already offers w.r.t ES6.

[[Go Top](#nodejs-best-practices-guide)]

## Linter

 - Unless otherwise mentioned, use the official Topcoder linter for nodejs [TODO]

[[Go Top](#nodejs-best-practices-guide)]

## App Configuration

 - Do not declare app constants as app configuration. The two are not the same and have a fine difference between them.
 - Values that will not change even with multiple deployments of the app are constants.
 - Values that the user would like to change when deploying the app are configuration.
 - The first place for the app to look for a configuration value should be the `process.env` variable. Provide default values for configuration where it makes sense but always read the `process.env` variable first.
 - Use capital letters for the environment variables as well as the configuration values. Group configuration values logically.
```javascript
// Avoid
{
    "API_KEY": 123,
    "API_URL": "http://foobar.com/",
    "API_PORT": 8000
}

// Use (default values listed here might go into a separate file
// if you end up using a module such as node-config
{
    "API": {
        "KEY": process.env.API_KEY || 123,
        "URL": process.env.API_URL || "http://foobar.com",
        "PORT": process.env.API_PORT || 8000
    }
}
```

 - Use a module such as [node-config](https://github.com/lorenwest/node-config) or [nconf](https://github.com/indexzero/nconf) to create a configuration file. Using these modules, you can avoid `require()`-ing the configuration file from deeply nested modules
```javascript
// Avoid
var config = require('../../../config');
...
fetch(config.API_URL)
...

// Use (using module node-config here)
var config = require('config');
...
fetch(config.get('API.URL'));
...
```

 - Avoid sharing confidential configuration in your submission. Even for default values, if you are unsure or the contest specification does not make it clear, you can use an empty string too.

[[Go Top](#nodejs-best-practices-guide)]

## HTTP Status Codes

 - This is relevant when creating API servers but it is also important when dealing with anything HTTP.
 - In some cases, the framework that you use takes care of the http status code to send with your responses. However, in certain cases you do need to override the default http status code to correctly describe the state of the request.
 - Most of the times, you would need to use the 4xx and 5xx series of http status codes and override the default 2xx status code.
 - There are many articles on the web that describe the 4xx and 5xx status codes in detail so it will not be described again here. The takeaway from this is that the http status code should depend on the type of response returned.

[[Go Top](#nodejs-best-practices-guide)]

### 401 vs 403

 - Quite commonly and incorrectly, each of these two http status codes are used in place of the other.
 - Use 401 when the user is not logged in, but should be. It's meant for authentication, not authorization.
 - Use 403 when the user is already logged in, but the user does not have permission to access the requested resource. This status code is for authorization.
 - User has entered wrong password - use 401
 - User has entered correct password but is trying to access a page only visible for admin users (which the user is not) - use 403.

[[Go Top](#nodejs-best-practices-guide)]

## Avoid Callback Hell

 - For small nodejs scripts or apps, it is alright to use callbacks to work with asynchronous code.
 - For not so small apps, it is best that you use a library such as [q](https://github.com/kriskowal/q), [async](https://github.com/caolan/async) or [bluebird](https://github.com/petkaantonov/bluebird/) instead of using callbacks.
 - Even for small apps, if you find yourself creating more than two stacks of callbacks, avoid it and use the above libraries instead.
 - With es6, you can now also make use of [Promise](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise) if you are only interested in some basic features.

[[Go Top](#nodejs-best-practices-guide)]

## Logging

 - For really small apps, it is alright to use `console.log()` to log information on to the console.
 - For medium and large apps, it is best that you use a logging library. Some popular ones are [winston](https://github.com/winstonjs/winston) and [node-bunyan](https://github.com/trentm/node-bunyan).
 - The default log level for production should be error and for development should be info.
 - When logging to a file, you need to provide a script to maintain the logs as well - delete old logs or move logs to another folder to archive. Use [npm scripts](#use-of-npm-scripts) to carry this out.

[[Go Top](#nodejs-best-practices-guide)]

## Error Handling
- All errors should be handled. Do not ignore any error.
- There is an excellent article about errors in Nodejs. [Check it out](https://www.joyent.com/node-js/production/design/errors). We will not repeat the same again here. [TODO]