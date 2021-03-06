# Building a node app with a basic web gui using pug

In this tutorial we're going to make a very lean and basic server-side web application that uses a template engine (pug) to dynamically display information pulled from an API.

## Prerequisites

- Nodejs on your machine

## Setup

Start by creating a folder in a directory of your choice using the terminal.
``` mkdir pugapp ```
``` cd pugapp ```
``` npm init ```

... answer the usual questions (the defaults are fine)

## Install dependencies
``` npm install -s axios dotenv pug ```

### Create the initial application files
Open up the application folder in your code editor.

Create a .env file to store your environment variables. (We'll start with the port we'll run the web server on)

.env
```
PORT=3000
```

Then create index.js to act as our application entrypoint and intialise the web server...

index.js
```js
require('dotenv').config()
const { createServer } = require('http')

const PORT = process.env.PORT || 1234

const server = createServer((request,response) => {
	return response.end('<html><h1>Hello world</h1></html>')
})

server.listen(PORT, () => {
  console.log(`The web server is running on port ${PORT}`)
})
```

Let's test it out.
In the terminal run ```node index.js ```
and in your web browser go to: http://localhost:3000

Well done! You just made a Node web server using the inbuilt http module (ie. without using express!! ) 
> Note: Express uses Node's http module under the hood.

In your terminal press ctrl + c to stop the node app from running.

## Templating

As you can see in the index.js file, the node server is returning raw html in the response. We could use insert variables into boilerplate html strings and join them all together but as the page gets larger, things would get quite messy if we were to continue down this path. Furthermore, dealing with dynamic content (like lists of an unknown length) can be tricky.

### Why use templates
Using templating gives us a cleaner way to write our 'presentation' code and helps separate out the 'view' of our application. Templating lets us define html 'views' using a simple syntax and insert variables on-demand to generate html output. In React you use JSX as the templating engine. For our server-side node application, we're going to use a much simpler one: pug.

For example a pug render file that looks like this:

```
doctype html
html
  head
	title Exploring the Pug template 
  body
	h1#myHeading This is a pug template
	p.firstParagraph I love this template!!!
	ul
	  each val in ['sauce', 'beers', 'chips', 'cheese']
		li= val

```

compiles into...
```html
<!DOCTYPE html>  
<html>  
  <head>    
    <title>  
      Exploring the Pug template  
    </title>  
  </head>  
  <body>  
    <h1 id ="myHeading" >  
      This is a pug template  
    </h1>  
    <p id = "firstParagraph">  
      I love this template!!!  
    </p>  
	<ul>
	  <li>sauce</li>
	  <li>beers</li>
	  <li>chips</li>
	  <li>cheese</li>
  </body>  
</html>
```

## Making our basic web application
### Creating html from a template file

We're going to make a web application that displays job vacancies from the API of a careers site. The structure will be straightforward and consist of 2 pages:
1 - An index page that shows a list of available jobs
2 - A details page for a given job.

First let's make the html views for each usng some dummy text and pug templates.

Create a folder to store our template views:

Inside that folder, create the template for the index page: index.pug

```
doctype html
html
  head
    title Not Seek
  body
    h1 Latest vacancies
    ul
      each job in ['job1', 'job2', 'job3']
        li= job
```

> note: pug is very fussy when it comes to indentation so make sure that you are consistent with your use of spaces or tabs (not both)

To create html from our template, we'll use pug's renderFile function in our index.js.
Let's require the pug library and replace our hand-coded html with the pug renderFile command.

```js
require('dotenv').config()
const { createServer } = require('http')
const pug = require('pug');

const PORT = process.env.PORT || 1234

const server = createServer((request,response) => {
  return response.end(pug.renderFile('views/index.pug'))
})

server.listen(PORT, () => {
  console.log(`The web server is running on port ${PORT}`)
})

```

Start the node app ```node index.js ``` and browse to http://localhost:3000 to view your rendered html.

If there are any problems, check your terminal for debugging details.

### Injecting variables into our template

We manually defined a list of jobs in our pug template file but like most templating engines, pug gives us an easy way to render variables in our html. The renderFile command take a key-value list (object) of variable names and values as its second argument.
eg: ``` pug.renderFile('views/index.pug', {name: 'Chris', priority: 1}))```

Define a data object in our Node index.js containing an array of vacant jobs.
```const data = { jobs: ['accountant', 'electrician', 'mechanic', 'teacher'] }```

and pass it as a second argument to the renderFile method.
```pug.renderFile('views/index.pug', data)```

index.js
```js
require('dotenv').config()
const { createServer } = require('http')
const pug = require('pug');

const PORT = process.env.PORT || 1234

const data = { jobs: ['accountant', 'electrician', 'mechanic', 'teacher'] }

const server = createServer((request,response) => {
  return response.end(pug.renderFile('views/index.pug', data))
})

server.listen(PORT, () => {
  console.log(`The web server is running on port ${PORT}`)
})

```

Now replace the list in the pug template file with the variable name we created in the key-value data object 'jobs'.
```
doctype html
html
  head
    title Not Seek
  body
    h1 Latest vacancies
    ul
      each job in jobs
```

Restart the node app, refresh your browser and you should see the list of jobs from index.js now displayed in your html list!

### Getting data from an external API
Let's make this app a bit more useful with some real jobs from an external API.

In index.js, require axios and replace our data variable definition with an axios request to the dataatwork jobs board.

```js
require('dotenv').config()
const axios = require('axios')
const { createServer } = require('http')
const pug = require('pug');

const PORT = process.env.PORT || 1234

const data = {}

axios.get('http://api.dataatwork.org/v1/jobs').then(res => {
  data.jobs = res.data;
});

const server = createServer((request,response) => {
  return response.end(pug.renderFile('views/index.pug', data))
})

server.listen(PORT, () => {
  console.log(`The web server is running on port ${PORT}`)
})

```

Restart the node app and take a look at the output. Because the API call returns an array of 'job' objects we're getting a list of objects rather than just the job descriptions.

Open up the API in the browser to inspect the JSON objects:
http://api.dataatwork.org/v1/jobs

From here we can see the Object's properties. "title" looks like a good one to display.
We could write some Javascript to reduce the API data down to just a simple array but fortunately Pug's variable substitution is clever enough that we can just reference the property name in our list item. Change:
```li job``` to ```li job.title```

```
doctype html
html
  head
    title Not Seek
  body
    h1 Latest vacancies
    ul
      each job in jobs
        li= job.title
```

Retart the server and you should now see a list or real jobs!

Right-click on your browser window to inspect the rendered source code.
You'll see that there is no Javascript. All the JS is executed by the Node server application to dynamically produce the HTML that gets sent to the client/browser.

...

# Pagination
There are 3 components to achieving pagination in our application.
![alt text](https://github.com/chris-ryan/node-ssr-tutorial/blob/main/pagination.png?raw=true)

## 1. The restricted list ##

Much like we did for page 2. We're going to use a url query to change what information gets stored on our index page. In this instance, we're going to use ?page= to pass in a page number that will tell us which section of the array to display on the index page. (10 jobs per page)

eg: ``` http://localhost:3000/?page=2 ``` should display the second page, ie list items 11 - 20.

Currently, when the index page is requested from our server, the logic is handled by the showAll controller function which passes a list of jobs to the pug template:

```js
const showAll = (jobs, url) => {
  return { jobs: jobs }
}
```

Let's start by getting the page number from the url query, add adding it as another property that we pass along to the pug template.

```js
const showAll = (jobs, url) => {
  let page = url.searchParams.get('page')
  return { jobs: jobs, page: page }
}
```

So that we can confirm what page we're on, let's add it in to our pug template.

```
doctype html
html
  head
    title Not Seek
  body
    h1 Latest vacancies
    h2 Page #{page}
    ul
      each job in jobs
        li
          a(href=`/page2.html?id=${job.uuid}`) #{job.title}
```
Restart the server and browse to http://localhost:3000/?page=2
You should now see "Page 2" as a subheading.

But what happens if we leave out the page argument? Eg: http://localhost:3000 

You'll see that the Page subheading number is blank. Let's fix that with an if statement to handle this scenario.

```js
const showAll = (jobs, url) => {
  let page = url.searchParams.get('page')
  if (!page) page = 1
  
  return { jobs: jobs, page: page }
}
```

Back in our index.js, use the page number to slice the array into only the bit we want to display.

``` Array.Slice() ``` takes 2 arguments: The start and the end indexes of the elements we want.
As each page will contain 10 elements, we can use our page number to calculate the start and end indexes to pass into our slice.

```js
const showAll = (jobs, url) => {
  let page = url.searchParams.get('page')
  if (!page) page = 1

  const endIndex = page * 10
  const startIndex = endIndex - 10
  const pagedJobs = jobs.slice(startIndex, endIndex)
  return { jobs: pagedJobs, page: page }
}
```
**Note** that we changed the value of the jobs property that we're returning so that its the newly sliced array *pagedJobs*

Restart the server and try browsing to http://localhost:3000 and http://localhost:3000/?page=2
You should see both the page number heading and the list of jobs change.

## 2. The pagination navigation list ##
To create our list of links with page numbers we can use a simple pug iterator (while loop) but first lets calclulate how many pages we're going to need so that we can tell pug when to stop iterating.

To calculate the number of required pages, we divide our array length by the number of elements per page and round up to the nearest whole number.
(The rounding up is taken care of by Javascript's Math.ceil() function

``` let pageCount = Math.ceil(jobs.length / 10) ```

Add this into the showAll function and pass it to pug as another variable.

```js
const showAll = (jobs, url) => {
  let page = url.searchParams.get('page')
  if (!page) page = 1

  let pageCount = Math.ceil(jobs.length / 10)

  const endIndex = page * 10
  const startIndex = endIndex - 10
  const pagedJobs = jobs.slice(startIndex, endIndex)
  return { jobs: pagedJobs, page: page, pageCount: pageCount }
}
```

Now in our index.pug, we can add it in next to our page number to test that its working.
``` h2 Page #{page} of #{pageCount} ```

Restart and refresh and you should now see Page X of Y on your jobs list page.

Back in index.pug, to create the list of links we create a loop using pug's while syntax.
Start by initialising a local variable at the top of the pug template file:
``` - var n = 1; ```

Then we loop through the numbers until we hit our calculated page count
```
    ul
      while n < pageCount
        li
          a(href=`/?page=${n}`) #{n++}
```

The index pug template should now look like this...

```
- var n = 1;

doctype html
html
  head
    title Not Seek
  body
    h1 Latest vacancies
    h2 Page #{page} of #{pageCount}
    ul
      each job in jobs
        li
          a(href=`/page2.html?id=${job.uuid}`) #{job.title}
    ul
      while n < pageCount
        li
          a(href=`/?page=${n}`) #{n++}
```

> Note: Just like in plain Javascript, while loops require manual progress which is what the n++ does.

Restart and refresh. You should now have a list of links that you can use to cycle between pages!
(There may be a blank final page as the array we're pulling down has an empty element at the end)

Use CSS to make the list horizontal. ``` display: inline-block; ``` should do the trick

## 3. Relative pagination navigation links ##

As we're already passing our current page number into our pug template, creating previous and next links should be fairly straightforward.
Add the following two lines to our index.pug file on either side of our navigation list:
```
a(href=`/?page=${page - 1}`) < Previous
a(href=`/?page=${page + 1}`) Next >
```

Restart, refresh and inspect the resultant page.

There are a few caveats here that we need to address.

First, take a look at the *Next* link. On page 1 its trying to take us to page 11 and on page 2 its linking to page 21.
That's because we're getting the page number as a url query parameter which by default is treated as a string. When we go to apply "+ 1" to it, the server literally appends the Number "1" to our variable.

Back in our index.js, lets put the function that gets the page number into parseInt() so that we can be certain that it will get stored as a Number type.

```js
const showAll = (jobs, url) => {
  let page = parseInt(url.searchParams.get('page'))
```

Another caveat is that our *Previous* and *Next* links can go beyond the scope of the number of pages (eg: page=0 and page=7).

Fortunately pug has a simple conditional syntax (if and else statements) that we can use to only show the links if they are valid options.

In index.pug: Prefix the *Previous* and *Next* links with the required if statements:

```
if page > 1
  a(href=`/?page=${page - 1}`) < Previous
```

```
if page < pageCount
  a(href=`/?page=${page + 1}`) Next >
```

> Note: This will make the links disappear completely. If you want to leave them visible but disabled and possibly greyed with a bit of CSS, you can add in an else statement to replace each link with just the text.

eg:
```
if page > 1
  a(href=`/?page=${page - 1}`) < Previous
else
  span < Previous
```

# Writing tests

Though writing tests are useful when building functionality in our code in terms of making us think about all the required inputs and behaviours of individual modules, their real power is in picking up new unexpected bugs from changes you made to some unsuspecting code in a related module or dependency update.

When writing tests for an application, you typically write them in batches that are module specific. These batches are referred to as suites or specs. Each suite typically contains a batch of tests organized in a way that almost mirrors the application itself. This works well with Node, where you can import/require the module you want to test against and write a collection of tests for each part of the module's functionality.

Since you'll end up with multiple test files, we employ a test runner: a command-line application that will run all our test files and give us the test results. We'll use Mocha as its by far the most popular Node test runner at the time of writing.

By default Mocha looks for a folder called 'test' then scans all the .js files within looking for specificly named function blocks:
- describe(): The parent function that contains a group of tests using Mocha functions.
- beforeEach(): Gets executed before the test to set up any required pre-test work.
- afterEach(): Gets executed after a test to perform any required cleanup before other tests in the group are run.
- it(): These blocks contain the "individual tests". Within each *it()* statement it's generally considered good practice to include a single assertion. (An assertion is a type of boolean expression that throws an error if the expression returns as false). ie. You want to *assert* that something is true and if not, return an error, which in this cae is picked up by Mocha as a failed test.

Here's some sample code for a typical test block:

```js
???describe('The code', function() {
    beforeEach(function(){
        // optional preparation for each test (such as connecting to a database)
    });
    afterEach(function(){
        // optional cleanup after each test (such as disconnecting from a database)
    });

    it('should test something', function(){
        var something = 1;
        // here we "expect" some condition to declare our test
        // in this case, we expect the variable to exist
        expect(something).to.exist;
    });
    it('should test something_else', function(){
        var something_else = false;
        // now we test a different variable against its value
        // and expect that value to equal false
        expect(something_else).to.equal(false);
    });
});???
```

## Defining assertions

There are a number of different syntaxes and libraries for constructing assertions. A very popular one that is often used with Mocha is Chai.js
Chai's syntax gives us a very natural logical and gramatical to write and read test logic.

## Writing a test helper
As you can see there's a consistent bit of overhead that we're requiring in our test files.
```
const expect = require("chai").expect
const nock = require('nock')
const request = require('superagent')
const apiCall = require("../index")

```

Rather than add it to every file, we can pop it into a single file and tell Mocha to automatically require it for every test file that is run.

1. Make a new file in the test folder called testhelper.js
test/testhelper.js

```
const expect = require("chai").expect
const nock = require('nock')
const request = require('superagent')
const apiCall = require("../index")

```

2. Add the file as an argument to our test run command in pakage.json

package.json
```
"scripts": {
    "test": "mocha --reporter spec --require test/testhelper.js"
  },
```