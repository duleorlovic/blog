---
layout: post
---

# Install

```
yarn add jest
```
Add script test command
```
// package.json
{
  "scripts": {
    "test": "jest"
  }
}
```

Create two files
```
// sum.js
function sum(a, b) {
  return a + b
}
module.exports = sum
```

```
// sum.test.js
const sum = require('./sum')

test('adds 1 + 2 returns 3', () => {
  expect(sum(1, 2)).toBe(3)
}
```

and run with yarn
```
yarn test

# or if you install globally: yarn global add jest
jest sum.test.js

# To run only one single test use `test.only('test', () => {`
# or use call with testNamePattern option 
jest sum.test.js -t adds

# run single file
jest --runInBand test/initialise.test.js

# debug in chrome, open chrome://inspect and use debugger in test or code
node --inspect $(yarn bin)/jest --runInBand test/initialise.test.js

# watch for file changes, like guard file listener
node --inspect node_modules/.bin/jest --watch
```

Mocking functions

```
const mockCallback = jest.fn(x => 42 + x)
some_func_to_call(mockCallback)
expect(mockCallback.mock.calls.length).toBe(2) // called twice
expect(mockCallback.mock.calls[0][0]).toBe(0) // first call first argument be
expect(mockCallback.mock.calls[0].value).toBe(0) // first call result

```
Instances
```
const myMock = jest.fn()
const a = new myMock()
const b = {}
const bound = myMock.bind(b)
bound()

console.log(myMock.mock.instances) // <a> <b>
```

To mock stylesheet files https://jestjs.io/docs/en/webpack
```
// package.json
{
  "jest": {
    "moduleNameMapper": {
      "\\.(css|less)$": "<rootDir>/__mocks__/styleMock.js"
    }
  },
}
```

## JSDOC

https://jsdoc.app/index.html#block-tags

* start with two stars `/**`


# Puppeteer

```
yarn add puppeteer
```

Create file
```
// example.js
const puppeteer = require('puppeteer')

(async () => {
  const browser = await puppeteer.launch()
  const page = await browser.newPage()

  await page.goto('https://mail.google.com')
  await page.screenshot({path: 'mail.png'})

  await browser.close()
})()
```

run with node
```
node example.js
# but easier with jest

```

browser

* launch in head mode dsable gpu
  ```
  const browser = await puppeteer.launch({
    headless: false,
    dumpio: true, // <== output chrome logs to STDOUT
    devtools: true,
    args: [
      '--disable-gpu',
      '--window-size=1280,800',
    ],
  })
  ```

page

* `await page.click('a[href=""])`. To click by text
  ```
  const [button] = await page.$x("//button[contains(text(), 'Button text')]");
  await button.click();
  ```
* `await page.type('input[type=email]', process.env.GOOGLE_USER)`
* when navigation occurs you can wait
  ```
  await page.waitForNavigation();
  await page.waitForNavigation({waitUntil: 'load'});
  ```
  this is also used with
  ```
  await page.waitForSelector('#login_field')
  ```
* use debugger in brower
  ```
  # increase jest.setTimeout(100000) and launch with {devtools: true}
  await page.evaluate(() => {debugger;})
  ```
* use debugger in node with [chrome::/inspect/]
  ```
  node --inspect $(yarn bin)/jest my.test.js
  ```
* capture console log from the browser
  ```
  page.on('console', msg => console.log('PAGE LOG:', msg.text()));

  await page.evaluate(() => console.log(`url is ${location.href}`));
  ```
* to `$eval` on specific element you can
  ```
  let className = await page.$eval(
    '#my-input',
    (input) =>  input.className
  )
  ```
* create screenshot `await page.screenshot({path: 'mail.png'})`
* sleep wait some time
  ```
  await page.waitFor(4000)
  ```
  but it is deprecated since you need to use `waitForSelector('button')`,
  `waitForFunction(someFunc)` or `waitForXPath('//div')`.
  but if you really want to sleep
  ```
  function delay(time) {
     return new Promise(function(resolve) { 
         setTimeout(resolve, time)
     });
  }

  // and in code

  await delay(4000);
  ```

Helpers
Click (wait before it shows up)
```
async clickRedirect(element) {
    const page = this.helpers['Puppeteer'].page;
    await page.waitForSelector(element);
    return await page.click(element);
}
```

Find and click a button or link by its text
```
const escapeXpathString = str => {
  const splitedQuotes = str.replace(/'/g, `', "'", '`);
  return `concat('${splitedQuotes}', '')`;
};

async function clickButton(page, text) {
  xpath = `//*[contains(text(), '${text}')]`
  await page.waitForXPath(xpath)
  const [button] = await page.$x(xpath)
  if (button) {
    await button.click()
  } else {
    throw new Error(`Button not found: ${text}`);
  }
}
```

You can rescue from async exceptions with
```
const run = async () => {
  throw new Error(`Some error ${myVar}`)
}
const logErrorAndExit = err => {
  console.log(err);
  process.exit();
};
run().catch(logErrorAndExit);
```
