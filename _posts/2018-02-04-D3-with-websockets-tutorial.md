---
published: false
---
## Real time visualizations with D3

A while back, I did [some visualizations](http://allisonking.github.io/fanfic-analysis) on Harry Potter fan fiction, scraped from [FanFiction.net](https://www.fanfiction.net). To do this, I built a web scraper in Python, saved off the data as one huge CSV file, and had my visualizations pull from bits and pieces of this data file. 

While this worked well for the data that I had, I found myself wishing that the data could be updated and that I could see in real time when fan fictions were being published. Even though it has been years since a Harry Potter book came out, in my scraping adventures, I found that people were still publishing their writing every hour or so! With the method I used, all of my data stopped on the day I happened to scrape it. How would I go about being able to continuously show updated data?

## Websockets
It turns out a good way to show updated data is through websockets. Before websockets, the way to continuously gather information would be to do long polling of HTTP requests. So I would set my code to scrape FanFiction.net every minute or so. However, with websockets, I only need to ask for a connection to FanFiction.net once, and that connection sticks around, so any time there is an update from FanFiction.net, I get it immediately, as opposed to after the next time I happen to send out an HTTP request. Neat!

So how do you make visualizations in D3 that utilize websockets in order to get real time updates? I put together this tutorial to show an example of how it can be done. Onwards!

# The Tutorial
Time to have some fun! We are going to use [GDAX's websocket feed](https://docs.gdax.com/#websocket-feed) as our source data, both because it was one of the few public websocket APIs I could find, and because cryptocurrency seems to be all the rage these days. GDAX is a a website that allows trading of cryptocurrency. Now before you put your trust in me that we'll be having fun and making a cool real time visualization, you probably want to see an end result. [Check it out!](https://bl.ocks.org/allisonking/83e870c24f5031e9a6abe3719bf66032)

What's going on here?
* Live data from the GDAX website of Ethereum transactions 
* Visualized with D3.js, featuring:
  * Scrolling to pop off older data as we get newer data
  * Automatic axes adjustment as data comes in
  
## Prerequisites
* A web browser capable of supporting web sockets. Check out [this site](https://www.websocket.org/echo.html) which will tell you whether or not your browser supports websockets!
* A local web server-- I use `http-server`. See [the D3 wiki](https://github.com/d3/d3/wiki#local-development) for how to use/install.
* A text editor

## Getting Started
Alright, we are ready to go! There will be two examples we will rely heavily on to get started:
* Mike Bostock's [Spline Transition](https://bl.ocks.org/mbostock/1642989) block
* Websocket.org's [Echo example](https://www.websocket.org/echo.html)

Spline Transition is part of a series of blocks on real time visualization that can be found [here](https://bost.ocks.org/mike/path/). The example uses randomly generated numbers, constrained to a certain range (a normal curve centered on 0 with standard deviation of 0.2) that get added every 'tick'. Because the randomly generated numbers are constrained in this example, there's no need to update axes. But we'll be using pretty unpredictable data (who knows what's going on in that crazy cryptocurrency market?) so we'll need an adjusting axis, and since we aren't just randomly generating data and are pulling it from a real source, we'll use websockets to get the data in real time. 

The echo example is a good reference for how to connect to a websocket, send and receive messages, and close a websocket. We'll use this as a reference.

Alright! Go ahead and download Spline Transition's `index.html`. The easiest way to download this is probably to either copy and paste from the block, or go to [the gist](https://gist.github.com/mbostock/1642989) and download the raw `index.html`.

## Understanding the code
We'll leave the CSS and HTML as is, and focus only on the JavaScript portions, the portions in between the `<script> </script>` tags. In this section, we will simply go over the lines of code, what they do, and I'll say a bit how we will change them, though we will save the action of actually changing the lines for the next section.

Let's start with the first three lines of JavaScript.
```javascript
var n = 40,
    random = d3.randomNormal(0, .2),
    data = d3.range(n).map(random);
```

`n` is the number of data points that will be shown on the x axis-- with new data coming in, we need to pop off the older values whenever we exceed `n`. We'll keep `n` at 40 in this tutorial, but feel free to adjust as you want. We won't be using `random`, but this is a function that generates a random number that would fit a normal distribution of (median, standard deviation). Finally, we have our ever important data! This line says essentially says 'give me `n` random values', where `n` is 40 in our case, and `random` is the function we just defined in the line above. We won't start with any data and will instead just start with an empty array since all of our data will be from websockets.

```javascript
var svg = d3.select("svg"),
    margin = {top: 20, right: 20, bottom: 20, left: 40},
    width = +svg.attr("width") - margin.left - margin.right,
    height = +svg.attr("height") - margin.top - margin.bottom,
    g = svg.append("g").attr("transform", "translate(" + margin.left + "," + margin.top + ")");
```
This is standard SVG and D3 code to get the SVG object from the HTML, to get the dimensions, and to make a group that we can append all of our SVG elements to.

```javascript
var x = d3.scaleLinear()
    .domain([1, n - 2])
    .range([0, width]);

var y = d3.scaleLinear()
    .domain([-1, 1])
    .range([height, 0]);
```
Here, the range and domain of our x and y scales are set. The x scale is straightforward and we will not be changing it-- we want the range to be the width of our SVG, and we want the domain to be the number of data points that we will have. Minus two. I believe that's to keep the data looking like it is moving while it comes in. 

The range for our y scale will be the same (the height of our SVG), but our domain will not be -1 to 1, because if it were, I think a lot of people would be very worried about their cryptocurrency portfolios! We won't know what this domain will until we start getting data in.

```javascript
var line = d3.line()
    .curve(d3.curveBasis)
    .x(function(d, i) { return x(i); })
    .y(function(d, i) { return y(d); });
```

This is another function, called `line` that will draw a D3 line based on x and y values. We won't be changing this.

```javascript
g.append("defs").append("clipPath")
    .attr("id", "clip")
  .append("rect")
    .attr("width", width)
    .attr("height", height);
```

Ah, a clip path! This says that anything outside of this rectangle will be clipped off and won't be shown. Why do we need this? Because we have a line which will be moving to x values that aren't in the chart, and we want everything to look like it's staying within the axes. Clip paths can be used for implementing zooming as well, where if you don't have them, you end up with data points outside your x and y axes!

```javascript
g.append("g")
    .attr("class", "axis axis--x")
    .attr("transform", "translate(0," + y(0) + ")")
    .call(d3.axisBottom(x));
g.append("g")
    .attr("class", "axis axis--y")
    .call(d3.axisLeft(y));
```

Here, we make the actual axis SVG element for both the x and y axes. D3 has some magic for axes. In this case, all we need to do is pass in the scales we defined earlier. `d3.axisBottom()` puts the axis at the bottom of the scale, and `d3.axisLeft()` puts it at-- you guessed it!-- the leftmost position of the scale. This example transforms the x axis up to the middle so that we get more of a T shaped graph. I like the x axis at the very bottom for the graph that we will be making, but it's up to you how to show off your data.