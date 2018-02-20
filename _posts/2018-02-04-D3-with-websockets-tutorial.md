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
In this section, we will simply go over the lines of code, what they do, and I'll say a bit how we will change them, though we will save the action of actually changing the lines for the next section. We'll leave the CSS and HTML as is, and focus only on the JavaScript portions, the portions in between the `<script> </script>` tags. 

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

```javascript
g.append("g")
    .attr("clip-path", "url(#clip)")
  .append("path")
    .datum(data)
    .attr("class", "line")
  .transition()
    .duration(500)
    .ease(d3.easeLinear)
    .on("start", tick);
```
These few lines of code are doing a good amount of work. The first `append` appends the clip path, which we defined earlier (the one we have an id of 'clip' to). The next `append` adds a path which uses that line function we defined earlier to calculate the values for the path. And finally, we have a transition which calls the `tick` function. This is responsible for the animation that we see. So what's the `tick` function?

```javascript
function tick() {

  // Push a new data point onto the back.
  data.push(random());

  // Redraw the line.
  d3.select(this)
      .attr("d", line)
      .attr("transform", null);

  // Slide it to the left.
  d3.active(this)
      .attr("transform", "translate(" + x(0) + ",0)")
    .transition()
      .on("start", tick);

  // Pop the old data point off the front.
  data.shift();

}
```

This bit of code is well commented already. Basically for each tick, which you can think of like a time step, we will:
* Add a new value to the data array
* Redraw the line so the new value is included
* Slide the line to the left to make the animated look
* Remove the oldest data point

And that's a quick explanation of what we start with! If you run your http-server at this time, you should see something like [this](https://bl.ocks.org/mbostock/raw/1642989/). Time to start adding our own code and get to the exciting parts!

## Adding in websockets
The first thing we need to do is to make a connection to a websocket. A lot of this websocket code is taken from the Echo test linked above. We'll also need to refer to the GDAX documentation for how to connect to their websocket. According to the documentation, we just need to make the connection, then send a subscribe message for exactly what we want to receive messages about. Let's add some JavaScript, right after the script tags start.
```javascript
<script>
// the URL to gdax's websocket feed
var wsUri = "wss://ws-feed.gdax.com";

// the subscription message to tell the websocket what kind of data we are interested in
var subscribe = '{"type": "subscribe","product_ids": ["ETH-USD"],"channels": [{"name": "ticker","product_ids": ["ETH-USD"]}]}';
```
Cool! We now have two variables, one for the connection URI, and one for our subscription message, a JSON string which in this case says I want the ticker channel for Ethereum. Now how do we actually connect? This is where we will borrow some code from the echo test.

Let's make a function called `initializeWebSocket()` and call it.
```javascript
initializeWebSocket();

/**
* Function that creates a WebSocket object and assigns handlers.
* Adapted from https://websocket.org/echo.html
*/
function initializeWebSocket() {
  websocket = new WebSocket(wsUri);
  websocket.onopen = function(evt) { onOpen(evt) };
  websocket.onclose = function(evt) { onClose(evt) };
  websocket.onmessage = function(evt) { onMessage(evt) };
  websocket.onerror = function(evt) { onError(evt) };
}
```

The first line in our `initializeWebSocket()` function creates a websocket object and passes in the URI we just pointed to GDAX. The rest assign functions to each of the major websocket functionalities-- opening, closing, getting a message, and getting an error. We are assigning those functionalities to our own functions, so now we have to go and define those.

Right under our `initializeWebSocket()` function, add:
```javascript
function onOpen(evt) {
  console.log("connected to websocket!");
  // send the subscription message
  websocket.send(subscribe);
}

function onMessage(evt) {
  // print out the message data
  console.log(evt.data); 
}

function onClose(evt) {
  console.log('disconnected');
}

function onError(evt) {
  console.log('error: ', evt.data);
}
```
And there are our four functions! We will add more to both the `onOpen()` and the `onMessage()` handlers, but for now, we'll just suffice with some log statements so that we know things are working.

At this point, you can refresh your page if you left your server running, and open up the Developer Tools on your browser to see the console. You should see the message "connected to websocket!" as the first output, then you will start seeing more console outputs as messages are received. Excellent-- we've connected to the websocket! To disconnect from it, in the console, type `websocket.close()`. You should get a console output back that says 'disconnected'. It's good practice to close the websocket-- eventually we will make the code do this for us. Otherwise, your server might be stuck running even when you think you've shut it down!

## Adapting the D3 code
Alright, time to start making the D3 code work for our data. Let's change this line of code:
```javascript
var n = 40,
    random = d3.randomNormal(0, .2),
    data = d3.range(n).map(random);
```
To:
```javascript
var n = 40,
    data = [];
```
We'll keep `n` at 40 and get rid of any initial data by making it an empty array. This means, unlike the Spline Transition example, the graph will start off empty, and gradually fill up before starting to transition.

After this, the next bit of code we are going to change is this one, where we set the y domain:
```javascript
var y = d3.scaleLinear()
    .domain([-1, 1])
    .range([height, 0]);
```
Many wallets would cry if the value of Ethereum were between -1 and 1, so we are going to have to change that to something more accurate for Ethereum. We will make this dynamic later, but for current testing, let's change it to the current price of Ethereum, plus or minus 5. For example, when first writing this code, this domain was enough for me to see the graph show up:

In order to figure out what that domain should be, take a look at the console output for when you first start getting transaction listings. Look for the 'price' attribute, then add and subtract 5 to get your range. For example, at the time of this writing, my console outputs:
```javascript
{"type":"ticker","sequence":2616782160,"product_id":"ETH-USD","price":"943.69000000","open_24h":"963.01000000","volume_24h":"100157.36096518","low_24h":"943.69000000","high_24h":"982.99000000","volume_30d":"6546933.68083798","best_bid":"943.41","best_ask":"943.69","side":"buy","time":"2018-02-18T20:05:42.919000Z","trade_id":29401592,"last_size":"1.79250000"}
```
Since the price here is listed as 943.69, my code would be:
```javascript
var y = d3.scaleLinear()
    .domain([938, 948])
    .range([height, 0]);
```

This may not catch all of the points you see, but it should be good enough to see a line getting drawn soon.

Finally, let's change this line that sets where the x axis is:
```javascript
g.append("g")
    .attr("class", "axis axis--x")
    .attr("transform", "translate(0," + y(0) + ")")
    .call(d3.axisBottom(x));
```
To:
```javascript
g.append("g")
    .attr("class", "axis axis--x")
    .attr("transform", "translate(0," + height + ")")
    .call(d3.axisBottom(x));
```
Rather can set the x axis for wherever the value 0 would be, we will just set it equal to the height of the SVG so that the x axis is always at the bottom. At this point, if we were to look at our web page, we would see a blank chart with only a x and y axis. The y axis should be hard coded to be near Ethereum's current prices, while the x axis is the same as in our original code, but moved to the bottom. If we open the console, then we can see messages coming in. Let's get those messages to show up on the graph now!

We'll start by modifying the `onMessage()` function to:
```javascript
function onMessage(evt) {
  // print out the message data
  console.log(evt.data); 
  
  // parse JSON response
  var point = JSON.parse(evt.data);
  
  // don't do anything if this isn't a 'ticker' message with transaction data
  if (point['type'] != 'ticker') {
    return; 
  }
  
  // add this point to the data array
  data.push(+point['price']);
  
  // redraw the line when new data gets added
  redrawLine();
  
  // if we have too much data, start the animation to move the line out of view
  if (data.length > n) {
    d3.select('.line')
      .transition()
      .duration(2000)
      .ease(d3.easeLinear)
      .on("start", tick); 
  }
}
```
Now when we receive the message, we:
* Print to the console the message we got. We'll keep this around for debugging.
* Convert the string response from GDAX to a JSON object for easy parsing
* Check to make sure the message we received is transaction data (since we send the subscription message, we want to ignore that and any other messages that might come in that don't have transaction data)
* Add the point to the data array. Part of this is converting the string to a float through the use of `+`
* Redraw the line. We still need to define this function.
* Check to see if our data array is too large (greater than our previously defined `n`). If it is, then tell the line to start its animation, in this case, by calling the function `tick()`. This bit of code is pretty much a copy of the original code that called `tick()`. In the original code, we wanted to start transitioning right away since we started with data. But in this code, since we start empty, we don't want the transition to start until after we are overfilling.

Because we told our line to start moving when we have more than `n` points, we don't need that bit of code that starts it off anymore. In this bit of code:
```javascript
g.append("g")
    .attr("clip-path", "url(#clip)")
  .append("path")
    .datum(data)
    .attr("class", "line")
  .transition()
    .duration(500)
    .ease(d3.easeLinear)
    .on("start", tick);
```
Remove everything from `.transition()` on so that it looks like:
```javascript
g.append("g")
    .attr("clip-path", "url(#clip)")
  .append("path")
    .datum(data)
    .attr("class", "line")
```
Now let's make the function for redrawing the line when data gets updated. Add anywhere in your code:
```javascript
function redrawLine() {
  d3.select('.line')
    .attr('d', line)
    .attr('transform', null);
}
```
This, too, is a copy of the original code inside `tick()`, under the "Redraw the line." comment. We'll factor it out into its own function so that both our `onMessage()` and our `tick()` functions can call it. Let's look at that `tick()` function now, and change it to:
```javascript
function tick() {
  // Redraw the Line.
  redrawLine();

  // Slide it to the left.
  d3.active(this)
      .attr("transform", "translate(" + x(0) + ",0)")
    .transition()
      .on("start", tick);

  // Pop the old data point off the front.
  data.shift();

}
```
We got rid of pushing data into the data array since our `onMessage()` function does that, and we changed the bit of code that redrew the line to be a function. Other than that, the `tick()` function is the same for now. Let's see what our page looks like now!

When you refresh, you should still see a blank chart, with just the axes. Your console should log that the websocket connection was made, and then it's just a matter of waiting for points to come in. Because our x scale was set to start at 1, and it takes two points to make a line, we actually won't see a line until there are at least 3 points inside our `data` array. You can type `data` into the console to see how many values are in there. You may also need to make sure that the values in `data` are still within that range we hard coded in for our y scale earlier. 

You should now see the graph begin to draw itself across the screen in real time! If you keep waiting, once the `data` array gets more than 40 values, the graph will start transitioning, just like in the Spline Transition example, by scrolling off the screen. 

And that's the basic idea for integrating websockets with D3! There are still some things that could be changed, for example:
* If data does not come in quickly enough, the animation will win out and we will be left with a blank graph again
* Could use some text to tell the user what is happening
* It'd be nice to automatically close the websocket after a certain amount of time
* Hard coded y domains. No one likes hard coded values!
  * Related: because domain is hard coded, values can go off the graph

## Enhancements
### Avoiding the blank graph
This problem comes because the graph starts animating moving out of the graph and popping data off as a time step. If this time step happens to be faster than the rate that data is coming in, you can end up with a blank graph. You could get rid of animation all together and simply have the `tick()` function called each time data comes in, but then you lose the sliding animation. I personally like it, so to counter this problem, I add to the beginning of the `tick()` function:
```javascript
function tick() {
  // Don't tick if we are short on data points
  if (data.length < n-2) {
   	return; 
  }
  
  // Redraw the Line
  ...
}
```
This just returns from the `tick()` function if we don't have enough data. You can compare `data.length` to anything less than `n`, depending on how you want your graph to look. Using `n-2` always makes the graph look full since that's our domain, but you could have it at `n/2` for instance, if you want the graph to stop sliding when the are only 20 transactions in the `data` array (this will make the graph fill up, then slide over to halfway along the x axis, then stop sliding until the graph is full again, then start sliding again).

### Adding text for the user
Because it may take a while for there to be enough transactions to start drawing the graph, we want to keep the user up to date on what is happening so they aren't just looking at a blank graph. We'll add a paragraph to our HTML where we can put text for the user to read.

At the beginning of the code, change:
```html
</style>
<svg width="960" height="500"></svg>
<script src="//d3js.org/d3.v4.min.js"></script>
<script>
```
to add the paragraph element:

```html
</style>
<p id="info"> </p>
<svg width="960" height="500"></svg>
<script src="//d3js.org/d3.v4.min.js"></script>
<script>
```
Great, now we have somewhere to write out text to. Let's start by modifying the `onOpen()` function. Instead of using the console log, we will write text to the `p` element we just made:

```javascript
function onOpen(evt) {
  d3.select('#info')
    .text('Connected to the websocket! Waiting for transactions...');
  // send the subscription message
  websocket.send(subscribe);
}
```

Now we want to change the text when we do get a transaction. Once again, as explained above, we won't actually be able to graph anything until we have at least 3 points in our `data` array. So we need to add a `count` variable that counts how many points we have seen so that we can alter our text. 

At the top of the JavaScript, after defining the subscription message, add:

```javascript
var count = 0;
```

Then, at the top of our `onMessage()` function, add:
```javascript
function onMessage(evt) {
  // increment counter
  count++;
  
  // parse JSON response
  ...
  // don't do anything if this isn't a 'ticker' message with transaction data
  ...
  
  // add this point to the data array
  ...
  
  // check for enough transactions
  if (count > 2) {
    d3.select(#info)
      .text("Real time visualization of Ethereum prices");
  }
  
  // redraw the line when new data gets added
  ...
}
```

Now if you refresh your page, you should see the message about waiting for a transaction up until the graph starts being drawn. Then, you'll see the real title of the graph, as well as the beginning of your line graph.

### Automatically close the websocket
Let's say we want the code to close the websocket after monitoring the market for five minutes. I did this in order to host the code on bl.ocks without it constantly being connected to the GDAX servers. After calling the `initializeWebsocket()` function, add to your code:

```javascript
// automatically close the web socket after five minutes
setTimeout(function(){
  websocket.close();
  d3.select('#info')
    .text("Websocket automatically disconnected after five minutes.");
}, 5*60*1000);
```
This will also tell the user that the websocket was automatically disconnected. 

### Automatically adjust y domain
Alright, it's high time we got rid of those hard coded values! First, we'll delete that line. Change:
```javascript
var y = d3.scaleLinear()
    .domain([938, 948])
    .range([height, 0]);
```
to:
```javascript
var y = d3.scaleLinear()
    .range([height, 0]);
```
Now we need a way to set the domain of the y scale. We won't know what our domain should be until our first messages comes in. So let's add another variable to the beginning of our code called `firstMessageReceived`, and set it equal to `false`. To determine our y domain, we will use this first message as our `center` point. Then, we will simply expand the y domain plus or minus `sigma`. Lastly, we will also want a `buffer` variable so that our line doesn't end up right at the edge of our graph.

```javascript
// the URL to gdax's websocket feed
...
// the subscription message to tell the websocket what kind of data we are interested in
...
var firstMessageReceived = false;
// keep track of the 'center' point for the y axis
var center;
// how much data we want to display around the center point
var sigma = 0.5;
// buffer for y axis
var buffer = 0.1;

// initialize web socket
...
```
We are going to make our `onMessage()` function do the hard work.

```javascript
function onMessage(evt) {
  // increment counter
  ...
  // parse JSON response
  ...
  // don't do anything if this isn't a 'ticker' message with transaction data
  ...
  
  // add this point to the data array
  ...
  
  // record when the first message is received so we can draw an initial y axis
  if (!firstMessageReceived) {
    firstMessageReceived = true;
    center = +point['price'];
    drawYAxis(center-sigma, center+sigma);
  }
  
  // check for enough transactions
  ...
  
  // code to expand the y domain if the data goes out of the current bounds
  if (+point['price'] > center+sigma) {
    sigma+= +point['price'] - (center+sigma) + buffer;
    drawYAxis(center-sigma, center+sigma);
  } else if (+point['price'] < center-sigma) {
    sigma+= (center-sigma) - +point['price'] + buffer;
    drawYAxis(center-sigma, center+sigma);
  }
  
  // redraw the line when new data gets added
  ...
}
```
Two chunks of code were added in this section. The first chunk sets the variable `center` to the value of the first transaction. Then it calls a yet to be defined function called `drawYAxis()` which takes a low and a high value, in this case, `center` plus or minus `sigma`.

The next chunk of code is to see if our y domain needs to expand in the case that we get new transactions that are outside our sigma range. So we check to see if the new data point is above or below our range, and if it is, then we expand sigma by the difference plus the buffer. Then call the `drawYAxis()` function on this new sigma.

Time to define this `drawYAxis()` function! Anywhere in your JavaScript, add:

```javascript
/**
* Function to draw the axes
*/
function drawYAxis(low, high) {
 // set the y domain
 y.domain([low, high])

 // set y axis
 d3.select('.axis--y')
   .call(d3.axisLeft(y));
}
```

With this new function, we can get rid of the bit of code that originaly called the axis. Change:
```javascript
g.append("g")
    .attr("class", "axis axis--y")
    .call(d3.axisLeft(y));
```
to just:
```javascript
g.append("g")
 .attr("class", "axis axis--y")
```

And that's it! Refresh the page, and watch your y axis adjust to incoming transactions!

## Conclusion
This is a quick example for how you can use D3 with websockets in order to make real time visualizations. I chose GDAX since they have an easily accessible public websocket API, but you could imagine rolling your own websocket server to serve up data that is more relevant to your project. Have fun!