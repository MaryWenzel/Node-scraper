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

##### Reopen `pageScraper.js`

- Add the following content under line 6 - (`await page.goto(this.url);`)

```js
// Wait for the required DOM to be rendered
await page.waitForSelector(".page_inner");
// Get the link to all the required books
let urls = await page.$$eval("section ol > li", (links) => {
  // Make sure the book to be scraped is in stock
  links = links.filter(
    (link) =>
      link.querySelector(".instock.availability > i").textContent !== "In stock"
  );
  // Extract the links from the data
  links = links.map((el) => el.querySelector("h3 > a").href);
  return links;
});
console.log(urls);
```

- In this code block we call the `page.waitForSelector` method. This method waited for the `div` that contains all the book related information that will be rendered in the DOM, and
- Then the `page.$$eval()` method is called. This gets the URL element with the slector `selection ol li`

  -- Be sure you always return a string or a number from the `page.$eval()` and `page.$$eval()` methods

- Save and close the file.

##### Re-run the application

- Run command `npm run start`
- The browser will open, navigate to the web page, and then close once the task completes.
- Check your console. It will now contain all the scraped URLs.

```js
Output

Opening the browser......
Navigating to http://books.toscrape.com...
[
  'http://books.toscrape.com/catalogue/a-light-in-the-attic_1000/index.html',
  'http://books.toscrape.com/catalogue/tipping-the-velvet_999/index.html',
  'http://books.toscrape.com/catalogue/soumission_998/index.html',
  'http://books.toscrape.com/catalogue/sharp-objects_997/index.html',
  'http://books.toscrape.com/catalogue/sapiens-a-brief-history-of-humankind_996/index.html',
  'http://books.toscrape.com/catalogue/the-requiem-red_995/index.html',
  'http://books.toscrape.com/catalogue/the-dirty-little-secrets-of-getting-your-dream-job_994/index.html',
  'http://books.toscrape.com/catalogue/the-coming-woman-a-novel-based-on-the-life-of-the-infamous-feminist-victoria-woodhull_993/index.html',
  'http://books.toscrape.com/catalogue/the-boys-in-the-boat-nine-americans-and-their-epic-quest-for-gold-at-the-1936-berlin-olympics_992/index.html',
  'http://books.toscrape.com/catalogue/the-black-maria_991/index.html',
  'http://books.toscrape.com/catalogue/starving-hearts-triangular-trade-trilogy-1_990/index.html',
  'http://books.toscrape.com/catalogue/shakespeares-sonnets_989/index.html',
  'http://books.toscrape.com/catalogue/set-me-free_988/index.html',
  'http://books.toscrape.com/catalogue/scott-pilgrims-precious-little-life-scott-pilgrim-1_987/index.html',
  'http://books.toscrape.com/catalogue/rip-it-up-and-start-again_986/index.html',
  'http://books.toscrape.com/catalogue/our-band-could-be-your-life-scenes-from-the-american-indie-underground-1981-1991_985/index.html',
  'http://books.toscrape.com/catalogue/olio_984/index.html',
  'http://books.toscrape.com/catalogue/mesaerion-the-best-science-fiction-stories-1800-1849_983/index.html',
  'http://books.toscrape.com/catalogue/libertarianism-for-beginners_982/index.html',
  'http://books.toscrape.com/catalogue/its-only-the-himalayas_981/index.html'
]
```

-- This is a great start, but we want to scrape all the relevant data not just the URL

##### Scraping the Relevant Data

- Reopen `pageScraper.js` again
- Add the following after line 20, which will loop through each link, open a new instance and retrieve relevant data

```js
// Loop through each of those links, open a new page instance and get the relevant data from them
let pagePromise = (link) =>
  new Promise(async (resolve, reject) => {
    let dataObj = {};
    let newPage = await browser.newPage();
    await newPage.goto(link);
    dataObj["bookTitle"] = await newPage.$eval(
      ".product_main > h1",
      (text) => text.textContent
    );
    dataObj["bookPrice"] = await newPage.$eval(
      ".price_color",
      (text) => text.textContent
    );
    dataObj["noAvailable"] = await newPage.$eval(
      ".instock.availability",
      (text) => {
        // Strip new line and tab spaces
        text = text.textContent.replace(/(\r\n\t|\n|\r|\t)/gm, "");
        // Get the number of stock available
        let regexp = /^.*\((.*)\).*$/i;
        let stockAvailable = regexp.exec(text)[1].split(" ")[0];
        return stockAvailable;
      }
    );
    dataObj["imageUrl"] = await newPage.$eval(
      "#product_gallery img",
      (img) => img.src
    );
    dataObj["bookDescription"] = await newPage.$eval(
      "#product_description",
      (div) => div.nextSibling.nextSibling.textContent
    );
    dataObj["upc"] = await newPage.$eval(
      ".table.table-striped > tbody > tr > td",
      (table) => table.textContent
    );
    resolve(dataObj);
    await newPage.close();
  });

for (link in urls) {
  let currentPageData = await pagePromise(urls[link]);
  // scrapedData.push(currentPageData);
  console.log(currentPageData);
}
```

~~Working on it...~~
