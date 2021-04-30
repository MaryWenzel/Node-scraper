# Overview

This is a web scraper made with Node.js and Puppeteer following this tutorial by 'Gbadebo Bello' :

https://www.digitalocean.com/community/tutorials/how-to-scrape-a-website-using-node-js-and-puppeteer

### What the scraper will do:

- Open Chromium and load a web scraping sandbox: books.toscrape.com
- Scrape all the books on a single page
- Scrape all the books across multiple pages
- Filter our scraped books by category
- Save to JSON file

### Step 1: Setting up the Web Scraper

##### Installing Node.js

- Have Node.js installed on your system
- If you do not already have Node.js you can download here:
- https://nodejs.org/en/

##### Initializing the Project

- Create a folder for the project
- run 'npm init' inside the project folder
- Press ENTER through all the prompts. You can fill the fields out if you would like but leave `entry point:` and `test command` on their default values
- Npm will save the output to a package.json file

##### Install Puppeteer

- Use npm to install Puppeteer: `npm install --save puppeteer`
- Linux machines may need additional dependencies, you can use the following command to find missing dependencies: 'ldd chrome | grep not'

##### package.json

- Open the `package.json` file in preferred text editor
- Find `scripts:` and add :`"start:" "node index.js"`underneath the test script

### Step 2: Setting up the Browser Instance

##### Create four JavaScript files

- `browser.js`
- `index.js`
- `pageController.js`
- `pageScraper.js`

##### Open `browser.js`

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

##### Open `index.js`

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

##### Open `pageController.js`

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

- This exports the browserInstance function and passes it to the `scrapeAll()` function
- `scrapeAll()` passes the instance to `pageScraper.scraper()` as an argument which is what it uses to scrape the pages
- Save and close `pageController.js`

##### Open `pageScraper.js`

- Create an object literal with a `url` property and a `scraper()` method
- The `url` is the URL of whatever webpage you want to scrape
- The `scraper()` method performs the actual scraping
- Add the following code:

```js
const scraperObject = {
  url: "http://books.toscrape.com",
  async scraper(browser) {
    let page = await browser.newPage();
    console.log(`Navigating to ${this.url}...`);
    await page.goto(this.url);
  },
};

module.exports = scraperObject;
```

- Puppeteer has a newPage() method that creates a new page instance in the browser
- Save and close `pageScraper.js`

##### Project File-Structure should look like this:

```md
|--- browser.js
|--- index.js
|--- node_modules
|--- package-lock.json
|--- package.json
|--- pageController.js
|--- pageScraper.js
```

- Run the command `npm run start` to run the scraper application
- This will open a Chromium browser instance, open a new page in the browser, and navigated to the URL. In this case: http://books.toscrape.com/

### Step 3: Scraping Data from a Single Page

##### Navigate the homepage

- Open your preferred web browser and navigate to the [books to scrape homepage](http://books.toscrape.com/)

- Browse the site to get a sense of how the data is structured
  `-- Category is displayed on the left`
  `-- Books are displayed on the right`

![alt text](https://i.imgur.com/NwU9UGb.png)

- When you click on a book the browser navigates to a different page with info on that book.
  `-- In this step, you will replicate this behavior, but with code`

##### Using DevTools

- Inspect the homepage to access the source code by using Dev Tools inside your browser.
- You will notice that the page list each books data under a `section` tag.
- Inside the `section` tag ever book is listed with a `li` tag. Here is where you can find the link to that book’s dedicated page, the price, and the in-stock availability.

![alt text](https://i.imgur.com/5ywvAGM.png)

- You will be:
  -- Scraping these books for URLs
  -- Navigating to each individual book page
  -- Scraping the books data
