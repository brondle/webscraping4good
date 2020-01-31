# Web Scraping For Better Datasets

To get the most out of this workshop, you’ll need to have [python](https://www.python.org/downloads/) and [scrapy](https://scrapy.org/) installed. If you haven’t used python before, we’re going to start with the simplest possible applications of it, so you should be able to follow along. If this ends up sparking your fancy, there are tons of resources available online, some of my favorites of which are linked on the workshop’s [are.na](https://www.are.na/brent-bailey/web-scraping-4-social-good).

Since there are likely a variety of skill levels in this group, fair warning: if you’re not familiar with the command line, some of the stuff in the practical portion of this workshop may be harder to follow, but you should be able to follow along by copying and pasting, and I’ll explain what’s happening at every step.

## What’s web scraping?

Simply put, it’s gathering and extracting data from a website or websites. If you’ve ever saved a meme from a website or copied and pasted text from an article, congrats! You’re a web scraper.

Generally, however, what people mean when they talk about web scraping is the automation of this process. Rather than manually searching flickr for pictures of trees and downloading them one by one, automated web scraping allows us to download every picture of a tree on flickr in a short amount of time by executing simple scripts or using software.

## What do people use it for?

## What tools can we use to scrape data?

### Basic

[wget] :

### High-Level

There are a whole lot of them, and you can also write your own! We’re going to focus primarily on scraping with Python, so here are a few of the most popular scraping tools.

[Scrapy](https://scrapy.org/): Python. High level, easy to use. Sometimes needs to be used with Selenium or Beautiful Soup for websites with a lot of dynamic content.
Good for crawling websites - visiting multiple pages in a single session. Extremely fast and extensible.

[Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/): Python. The OG scraper, old, good for parsing HTML, lower level.

[Selenium](https://github.com/SeleniumHQ/selenium): APIs for a number of languages. Often used for web automation or testing. Works well with JavaScript and dynamic content - better at mimicking the way we actually interact with websites.

## Let’s write a scraper!

If you haven’t already, let’s install Scrapy. You can do this easily in the terminal with ```pip install scrapy```.

Once scrapy is installed, we can create an empty project folder for our scraper:
``` scrapy startproject babys_first_scraper```

This will create a folder called “babys_first_scraper”. if you enter the folder, it should contain a structure like this:
![the standard scrapy directory structure](img/file_structure.png)

Now let’s start actually writing our scraper!

Go to the `spiders` directory within the `babys_first_scraper` folder:

```cd babys_first_scraper/spiders``` and create a spider file: ```touch text_spider.py```.

### Wait, what’s a spider?

A spider, also often called a web crawler, is a bot that automatically browses websites. This is how Google gets its search results, and also the primary method for most web scrapers of accessing the internet: rather than creating a spider for the entire world wide web, like google, we automate the process of visiting a single website or a few websites, then within the spider we can specify the data we want and download it.

### Back to making the spider

In your text editor of choice, open `text_spider.py`. We’ll start by telling it what website to visit: in this case, we’ll be using [books.toscrape.com](books.toscrape.com), a prebuilt website intended for people to test their scrapers on.

First, in the `text_spider.py` file, enter:

```
import scrapy

class TextSpider(scrapy.Spider):
  name=”text”
```

This tells the script to use Scrapy’s prebuilt Spider class, so we don’t have to deal with writing our own basic spider code. We then name the spider “text” since that’s what we’re focused on getting right now.

Now, below `name=”text”`, enter:

```
  def start_requests(self):

    #tell it which website(s) to visit
    urls=[
      ‘http://books.toscrape.com’]
    #visit each url
    for url in urls:
      #at each url, run the ‘parse’ function
      yield scrapy.Request(url=url, callback=self.parse)

    def parse(self, response):
      #set a filename to write the response to
      filename = ‘books.html’
      with open(filename, ‘wb’) as f:
        f.write(response.body)
      self.log(‘Saved file %s’ % filename)
```

We now have the simplest possible spider, so let’s run it!

Go to the root folder in the terminal and run

`scrapy crawl text`

If everything in your script is correct, you should see something like this:

![A bunch of Scrapy terminal printouts from the run process](img/successful_scrape.png)

And when your script is finished running, there should now be a `books.html` file in your root folder.

If you open this in the browser, you’ll see a bunch of unstyled HTML. Not super useful - we could just have used  `wget` for that.

So let’s start taking advantage of the power of Scrapy in earnest!

### Parsing HTML

Where Scrapy really starts to become useful is when we start to parse the raw HTML of the pages Scrapy visits. On books.toscrape.com, for example, each book item we see:

![Three books with images, titles, ratings, and price displayed on a mock bookshop website](img/books.png)

is represented in the page’s HTML by an `<article>` object like this:

![An HTML <article> object containing a number of HTML objects](img/article_html.png)

#### A quick aside: #### If you want to be an *elite hacker*, Scrapy has a command line tool for exploring html, [Scrapy shell](https://docs.scrapy.org/en/latest/topics/shell.html#topics-shell). We don’t have time to cover it here, but it’s a useful tool if you’re more comfortable in the terminal.

In this example, if we dig into the HTML further, we’ll find that the book title is contained inside the `<h3>` object:

![An <h3> object containing a link titled “Tipping the Velvet”](img/book_title.png)

So, if we want to get the title of every book on the page, we need to tell Scrapy to go get it from inside that `<h3>`.

Let’s go back to `text_scraper.py`

For now, we can remove the old code under our `parse` function, and instead tell Scrapy to extract data from the page’s HTML:

```def parse(self, response):
      #as we can see in the page’s HTML, each book is contained in an <article> with the class ‘product_pod’, so we’ll tell Scrapy to start there

      for book in response.css(‘article.product_pod’):
        yield {
          ‘title’: book.css(‘h3.text::text’).get()
        }


      ```

If we run our scraper again, we should see it output some book titles:

![A command line printout of a bunch of book titles](img/scrapy_title_print.png)

Now we’re getting somewhere! But if we want to actually store this data, we’ll need to tell Scrapy to keep it. This is super simple for basic JSON objects:

run `scrapy crawl text -o titles.json`

And it should write a JSON file to your folder containing all the objects. JSON not useful for your project? [Scrapy’s feed exports](https://docs.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-exports) include CSV and XML as well.

### Do It Yourself

Now try getting the price or star rating from the <article> object.

### Downloading images














## More useful stuff

[Using Scrapy with XPath](https://docs.scrapy.org/en/latest/topics/selectors.html#topics-selectors) : XPath is a more powerful way of scraping pages by searching for content that’s not just CSS.
