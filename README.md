This is a web scraper made with Node.js and Puppeteer following this tutorial by 'Gbadebo Bello' :

https://www.digitalocean.com/community/tutorials/how-to-scrape-a-website-using-node-js-and-puppeteer

What the scraper will do:

- Open Chromium and load a web scraping sandbox: books.toscrape.com
- Scrape all the books on a single page
- Scrape all the books across multiple pages
- Filter our scraped books by category
- Save to JSON file

Step 1: Setting up the Web Scraper

Installing Node.js

- Have Node.js installed on your system
- If you do not already have Node.js you can download here:
- https://nodejs.org/en/

Initializing the Project

- Create a folder for the project
- run 'npm init' inside the project folder
- Press ENTER through all the prompts. You can fill the fields out if you would like but leave `entry point:` and `test command` on their default values
- Npm will save the output to a package.json file

Install Puppeteer

- Use npm to install Puppeteer: `npm install --save puppeteer`
- Linux machines may need additional dependencies, you can use the following command to find missing dependencies: 'ldd chrome | grep not'

package.json

- Open the `package.json` file in preferred text editor
- Find `scripts:` and add :`"start:" "node index.js"`underneath the test script

Step 2: Setting up the Browser Instance

Create four JavaScript files

- `browser.js`
- `index.js`
- `pageController.js`
- `pageScraper.js`

Open `browser.js`

- `require` Puppeteer at the top of the file
- Create an `async` function called `startBrowser`
- This function will start the browser and return the instance
- Paste the following into `browser.js`

```js
const puppeteer = require("puppeteer");

async function startBrowser() {
  let browser;
  try {
    console.log("Opening the browser......");
    browser = await puppeteer.launch({
      headless: false,
      args: ["--disable-setuid-sandbox"],
      ignoreHTTPSErrors: true,
    });
  } catch (err) {
    console.log("Could not create a browser instance => : ", err);
  }
  return browser;
}

module.exports = {
  startBrowser,
};
```

- Puppeteer has a `.launch()` method that launches an instance of a browser. This method returns a Promise, so you have to make sure the Promise resolves by using a `.then` or `await` block.
- Save and close `browser.js`

Open `index.js`

- `require` `browser.js` and `pageController` at the top of the file
- Call the `startBrowser()` function and pass the created browser instance to the page controller
- Add the following code to `index.js`

```js
const browserObject = require("./browser");
const scraperController = require("./pageController");

//Start the browser and create a browser instance
let browserInstance = browserObject.startBrowser();

// Pass the browser instance to the scraper controller
scraperController(browserInstance);
```

- Save and close `index.js`

Open `pageController.js`

- Add the following to `pageController.js`

```js
const pageScraper = require("./pageScraper");
async function scrapeAll(browserInstance) {
  let browser;
  try {
    browser = await browserInstance;
    await pageScraper.scraper(browser);
  } catch (err) {
    console.log("Could not resolve the browser instance => ", err);
  }
}

module.exports = (browserInstance) => scrapeAll(browserInstance);
```
