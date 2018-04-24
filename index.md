# e2e Testing with Angular and Protractor

## What is it?

Protractor is a test framework and wrapper around the Selenium-webdriver, which automates browsers. (learn more about <a href='http://www.seleniumhq.org/'>Selenium)</a>  

Please visit the official protractor <a href='http://www.protractortest.org/#/infrastructure'>page</a> for further understanding how protractor works.  

This documnetation contains only information about the most important parts of the protractor API. (See below)  
You can find a basics video here: <a href='https://www.youtube.com/watch?v=LCSwmJwRU1U'>youtube.com</a>  
Installation instructions: <a href='http://www.protractortest.org/#/tutorial'>protractor/tutorial</a>  
Setting up Selenium server: <a href='https://github.com/angular/protractor/blob/master/docs/server-setup.md'> github/server-setup</a>  


## Configuration 

### File
A default Angular app created with the ng cli contains a protractor.conf.js file.  
A basic config for e2e testing could look like this:  
```javascript
const { SpecReporter } = require('jasmine-spec-reporter');

exports.config = {
  allScriptsTimeout: 11000,
  specs: [
    './e2e/**/*.e2e-spec.ts'
  ],
  capabilities: {
    'browserName': 'chrome'
  },
  directConnect: true,
  getPageTimeout: 30000,
  baseUrl: 'http://localhost:4200/',
  framework: 'jasmine',
  jasmineNodeOpts: {
    showColors: true,
    defaultTimeoutInterval: 30000,
    print: function() {}
  },
  beforeLaunch: function() {
    require('ts-node').register({
      project: 'e2e/tsconfig.e2e.json'
    });
  },
  onPrepare() {
    jasmine.getEnv().addReporter(new SpecReporter({ spec: { displayStacktrace: true } }));
  }
};
```
  
Little explanations for this config:  
(all options can be found in the <a href='https://github.com/angular/protractor/blob/master/lib/config.ts'>protractor config file</a>.)
#### capabilities
to set up browsers, see <a href='https://github.com/SeleniumHQ/selenium/wiki/DesiredCapabilities'>selenium/DesiredCapabilities</a>  
Protractor can launch tests on one or more browsers. To configure multiple browsers, use 'multiCapabilities'. Example: <a href='https://github.com/angular/protractor/blob/master/docs/browser-setup.md#testing-against-multiple-browsers'>multiple browsers</a>

#### Browser window size and mobile device emulation 
The browser's size can be set in the capabilities option:  
```javascript 
 capabilities: {
        'browserName': 'chrome',
        'chromeOptions': {
            args: ['--window-size=800,600']
        }
    }
```  
To emulate the tests in mobile device sizes, just add these lines:  
```javascript
 capabilities: {
        'browserName': 'chrome',
        'chromeOptions': {
            'mobileEmulation': {
                'deviceName': 'iPhone 6 Plus' // must be exact string from chrome mobile device list
            }           
        }
    }
```  
If both configs are given, the window gets resized to the defined size and emulates the selected device.  

#### specs
relative to the config file path; array of strings containing spec files to execute. Search for 'specs' in the <a href='https://github.com/angular/protractor/blob/master/lib/config.ts'>protractor config file</a>.
  
#### beforeLaunch
A callback function called before any environment setup and before onPrepare.  
Here, it is called to load the typescript configs. 
  
#### onPrepare
This is called after beforeLaunch and before running the specs. Here, it is used to add a jasmine reporter. 
  
#### directConnect
Protractor can launch tests directly through the browser's webdriver. Only Firefox and Chrome are supported.  
Use 'directConnect=true'  
**Note: there is an open <a href='https://github.com/angular/protractor/issues/4253'>issue</a> about Firefox 52+ and directConnect.**  
To use Firefox, please remove the directConnect and configure the seleniumAddress instead. Start your selenium server with  
```
>> webdriver-manager update 
>> webdriver-manager start
```
  
  
#### Debugging with Protractor  
There are 2 possibilities to debug tests in protractor.  
Please have a look at this well explained, short post: https://www.linkedin.com/pulse/debugging-protractor-sachini-chathurika  
## Selenium Server  
The selenium server provides the connection between the tests and the drivers for the browser. 
If you have installed the Java Development Kit JDK and followed the setup on the official installation guide, you can configure your selenium address:  
```
seleniumAddress: 'http://localhost:4444/wd/hub' // default  
```
  
make sure you started your localhost by running 'ng serve'. 

## Testing in Microsoft Edge
Microsoft Edge supports automated testing. There are some requirements: 
- directConnect must be set to false
- installed WebDriver from https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/
- set selenium address , see <a href="#selenium-server">selenium server</a>  

Then, start your localhost by running 'ng serve' 
Open an other command line tool and type 'webdriver-manager start --edge "path\to\webdriver.exe" '  
You can run your tests now. 

## Executing tests
You have different options to run your tests. First of all, you should have a clean file structure, for example:  
|-- project-folder  
+  |-- app  
+   ...  
+    index.html  
+  |-- test  
    +    |-- unit  
    +    |-- e2e  
        +  |-- page-objects  
            +  home-page.po.ts  
            +  profile-page.po.ts  
            +  contacts-page.po.ts  
        + home.e2e-spec.ts  
        + profile.e2e-spec.ts  
        + contacts.e2e-spec.ts
  
If you open your protractor.conf.js file, you will see, that all files ending with .e2e-spec.ts get executed by default. This is defined in the specs array.  
You can select specific spec.ts files by overgiving them as option in the protractor cli:  
```
>> protractor --specs e2e/home.e2e-spec.ts,e2e/contacts.e2e-spec.ts
```

### Test suites
You can organise your tests even with test suites. In the protracor.conf.js file, write:  
```
suites: {
  homepage: 'tests/e2e/homepage/**/*Spec.js',
  search: [
    'tests/e2e/contact_search/**/*Spec.js',
    'tests/e2e/venue_search/**/*Spec.js'
  ],
  smoke: 'tests/e2e/smoke/*.js',
  performance: 'tests/e2e/performance/*.js
```
command to execute the suites:  
```
>protractor --suite=pages
```  
Unite these tests by a common logic or property.  
<small>Example from <a href='https://stackoverflow.com/questions/30331018/suites-vs-specs-protractor'>stackoverflow</a></small>

## Promises

WebDriverJS is completely **asynchronous**.  
You will notice that all functions return values are promises.  
You don't have to take care of the .then() chaining.  
Protractor's browser has an inbuilt promise scheduler from the WebDriverJS API called ControlFlow.  
<a href='https://github.com/angular/protractor/blob/master/docs/control-flow.md'>The WebDriver Control Flow on Github</a>  
<a href='https://spin.atomicobject.com/2014/12/17/asynchronous-testing-protractor-angular/'>Asynchronous Testing with Protractor’s ControlFlow</a>  

You only should write the .then() by yourself if you explicitly need the resolved value.  
Consider this example:

```javascript
1    it('should click signup link and navigate to /signup', () => {
2       loginPage.visit();
3        loginPage.clickSignupLink();
4        return loginPage.getCurrentUrl().then(url => {
5            expect(url).toMatch('/signup');
6       });
7    });
```
The .then() is used on line 4 because we want the returned url to match '/signup'.  
Ok, this code works. But a definitely cleaner way is to take advantage of the existing chaining by WebDriverJS.  
Because we know that the promise is completed after the call of getCurrentUrl(), we can easily combine it with Jasmine's expect() and toMatch().  

```javascript
1   expect(loginPage.getCurrentUrl()).toMatch('/signup');
```
No need of catching the url's value manually.


## Protractor API

The most important and useful functions are described here.  

### browser
Instance to control the browser  

#### get()
Accessing an address  

```
browser.get('yoururl')
```

#### navigate()
navigate to previous page  
```     
browser.navigate().back();
```

navigate to next page  

```
browser.navigate().forward();
```
    
#### getCurrentUrl()
Current page's URL

```
browser.getCurrentUrl()
```
    
####  isElementPresent(className, name, linkText, ...)

```
browser.isElementPresent('btn-primary')
```

#### close()
Close the current window

```
browser.close()
```

#### setLocation()
Use in-page navigation
```
browser.get('http://angular.github.io/protractor/#/tutorial');
browser.setLocation('api');
expect(browser.getCurrentUrl())
    .toBe('http://angular.github.io/protractor/#/api');
```

### actions()
Perform actions like drag and drop  
<a href='https://lambda-it.mocoapp.com/activities'>Code example</a>

### call()
Add another function to the queue, ensure it gets called in correct order  
<a href='http://www.protractortest.org/#/api?view=webdriver.WebDriver.prototype.call'>Code example</a>

### sleep()
Causes the webdriver to sleep for a given amount of time. This is scheduled as a command (sleep command).  
Note that this may slow down or fail your tests if sleep() commands are called in many of your tests and when the system runs slower than normal.  
Please use alternatives to sleep().  
Native sleep usage:
```
browser.sleep(3000)
```
Alternatives:  
Use **browser.wait(condition)**.

### wait()
It waits for a condition or for a promise to resolve while blocking the control flow.  
In contrast to sleep(), wait() works with conditions to become true and does not 'just sleep' for a definite given time. Therefore your test continues to work dynamically.  
Let's take a further look at these conditions:  
For example we have a user who logs in and gets redirected to a new page.  
  
1. Login  
2. click button to login  
3. get redirected to new page  
  

**How can we determine the right moment to call our redirection?**  
* use sleep()  
* use wait() and define conditions  
  
Below are code examples comparing both ways.  
use sleep():  
```javascript
1   loginPage.visit();
2   loginPage.setEmail('user.name@mail.com');
3   loginPage.setPassword('Testuser123');
4   loginPage.clickLoginButton();
5   browser.sleep(1000);
6   expect(loginPage.getCurrentUrl()).toMatch('/fidentity-additional');
```
Again we are facing fragility at line 5. How can we know that Angular finishes the clickLoginButton() at line 4 after exactly 1000ms?  
We just can't know. But we all know that the test fails if we wouldn't give the browser enough milliseconds to sleep.  
So here is the better solution with wait(). We don't have to loose time with estimating milliseconds anymore...  
```javascript
1   loginPage.visit();
2   loginPage.setEmail('user.name@mail.com');
3   loginPage.setPassword('Testuser123');
4   loginPage.clickLoginButton();
5   browser.wait(protractor.ExpectedConditions.invisibilityOf(loginPage.loginButton));
6   expect(loginPage.getCurrentUrl()).toMatch('/fidentity-additional');
```
At line 5 the browser now waits for a certain condition. Our condition will be true as soon as the login button gets invisible. But what is this "ExpectedConditions" object?  
Well protractor comes with a library of expected conditions. (Chapter ExpectedConditions)

### ExpectedConditions 
The expected conditions are functions which return a promise of booleans to indicate the condition to be true or false.  
You can mix them with operators 'and', 'or', 'not'. Furthermore, you can write and use your own conditions.  
These conditions are mostly used with brower.wait().  
Visit <a href='http://www.protractortest.org/#/api?view=ProtractorExpectedConditions'>protractortest/ExpectedConditions</a> to see code examples.  
Let's rewrite our user login test.  Before, we had this example:  
```javascript
1   loginPage.visit();
2   loginPage.setEmail('user.name@mail.com');
3   loginPage.setPassword('Testuser123');
4   loginPage.clickLoginButton();
5   browser.wait(protractor.ExpectedConditions.invisibilityOf(loginPage.loginButton));
6   expect(loginPage.getCurrentUrl()).toMatch('/fidentity-additional');
```
Line 5 makes the browser wait until the login button gets invisible, because we want to pass the test by matching the url to a new string. (Line 6)  
ExpectedConditions offers us a function called 'urlContains()'. It lets us write the test condition in just 1 line:  
```javascript
1   loginPage.visit();
2   loginPage.setEmail('user.name@mail.com');
3   loginPage.setPassword('Testuser123');
4   loginPage.clickLoginButton();
5   expect(browser.wait(protractor.ExpectedConditions.urlContains('fidentity-additional'))).toBe(true);
```
We call it in our expect(...) and expect it to be 'true', because 'urlContains()' returns a boolean. 

### element
Compare, manage and get information about the UI's html elements.  
To select elements, you should know about the possible <a href='#locators'>locators</a>  

#### element.isPresent()
Determine whether the element is present in the DOM or not.  
```
const isPresent = element(by.css('h2')).isPresent();
```
#### element.isDisplayed()
Determine by visibility (css style) whether the element is visible or not.  
```
html 
<div id="foo" style="visibility:hidden">

test
const foo = element(by.id('foo'));
expect(foo.isDisplayed()).toBe(false);
```
#### element.isEnabled()
Returns a boolean of the disabled state defined in the style of the element.  
```
html 
<input id="container" disabled=true>

test
const container = element(by.id('container'));
expect(container.isEnabled()).toBe(false);
```
#### element.isSelected()
Returns a boolean of the checked state defined by clicking a checkbox element.  
```
html 
<input id="coke" type="checkbox">

test
var coke = element(by.id('coke'));
expect(coke.isSelected()).toBe(false);
coke.click();
expect(coke.isSelected()).toBe(true);
```
#### element.element() 
Reaching child elements:  
<a href='http://www.protractortest.org/#/api?view=ElementFinder.prototype.element'>protractor/element</a>  
  
Reaching child elements by shortcut $:  
<a href='http://www.protractortest.org/#/api?view=ElementFinder.prototype.$'>protractor/$</a>

#### element.evaluate()
Get an element's value by its scope
```
html 
<span id="name">{{name}}</span>

test
const value = element(by.id('name')).evaluate('name');
```
#### element.submit()
Submit a form. Element must be a form.  
```
html 
<form id="login">
  <input name="user">
</form>

test
const form = element(by.id('login'));
form.submit();
```

#### element.click()
Perform a click.
```
const loginButton = element(by.buttonText('Login'));
loginButton.click();
```

### element.all()
Uses the ElementArrayFinder to return an array of elements. Any action will be called on each element in the array.  
Good examples <a href='http://www.protractortest.org/#/api?view=ElementArrayFinder'>here</a>  


### locators  
Element location always starts with the keyword by().  
For example:  
```
element(by.buttonText('Login'));
```
#### className()  
Find elements by class name
```
element(by.className('btn-primary'));
```

#### css()  
Find elements by css selectors  
For example, common selectors are: h1, h2, h3,..., p, .my-clss, #elem, div...  
More <a href='https://drafts.csswg.org/selectors-3/'>selectors</a>
```
element(by.css('h3'))
```  
Shortcut for element(by.css()) is: $  
```
const elem = $('h3');
```

#### Locators for Angular  
Note: Protractor no longer supports these locators due to the new Angular Version (AngularJS-->Angular)  
Official <a href='https://angular.io/guide/upgrade#e2e-tests'>angular.io</a> suggests using the locator by.css().
**bindings** Deprecated
```
html
<span>{{person.name}}</span>

locator
var name = element(by.binding('person.name'));
```
**ng-repeat** Deprecated  
Find elements by ng-repeat  
Good examples <a href='http://www.protractortest.org/#/api?view=ProtractorBy.prototype.repeater'>by.repeater</a>

**ng-model** Deprecated
```
html
<input type="text" ng-model="person.name">

locator
var input = element(by.model('person.name'));
```

## Sources
<a href='https://amitgharat.wordpress.com/2015/11/01/protractor-the-secret-service/'>Sleep, Wait, ExpectedConditions</a>  
<a href='https://www.blackpepper.co.uk/what-we-think/blog/if-youre-using-browser-sleep-in-your-cucumber-protractor-selenium-tests-youre-doing-it-wrong'>Avoid sleep()</a>  
<a href=''></a>  
<a href=''></a>
