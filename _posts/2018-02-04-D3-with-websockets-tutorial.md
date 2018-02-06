---
published: false
---
## Real time visualizations with D3

A while back, I did [some visualizations](http://allisonking.github.io/fanfic-analysis) on Harry Potter fan fiction, scraped from [FanFiction.net](https://www.fanfiction.net). To do this, I built a web scraper in Python, saved off the data as one huge CSV file, and had my visualizations pull from bits and pieces of this data file. 

While this worked well for the data that I had, I found myself wishing that the data could be updated and that I could see in real time when fan fictions were being published. Even though it has been years since a Harry Potter book came out, in my scraping adventures, I found that people were still publishing their writing every hour or so! With the method I used, all of my data stopped on the day I happened to scrape it. How would I go about being able to continuously show updated data?

## Websockets
It turns out a good way to show updated data is through websockets. Before websockets, the way to continuously gather information would be to do long polling of HTTP requests. So I would set my code to scrape FanFiction.net every minute or so. However, with websockets, I only need to ask for a connection to FanFiction.net once, and that connection sticks around, so any time there is an update from FanFiction.net, I get it immediately, as opposed to after the next time I happen to send out an HTTP request. Neat!

So how do you make visualizations in D3 that utilize websockets in order to get real time updates? I put together this tutorial to show an example of how it can be done. Onwards!

