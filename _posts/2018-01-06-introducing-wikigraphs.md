---
published: true
layout: post
title: Introducing Wikigraphs
---
**Update (1/24/2018):** This is now deployed on [Heroku!](https://wikigraphs.herokuapp.com/)

------------

For the past few weeks, I have been working on a project that I am currently calling Wikigraphs. What in the world is that, you ask? Well, thanks for asking! If I had to summarize it, I'd say it is a framework that allows for community driven data as input into data driven graphs, thereby allowing for transparent data sources.

Let's break that down. 

## The Data Driven Graph
There are a lot of great data driven graphs out there. D3js, the popular JavaScript library, stands for Data Driven Documents, after all, and powers many of those awesome *New York Times* visualizations. These polished graphs are easily interpreted as objective and authoritative-- after all, data does not lie, right? But data can omit and misrepresent, and when that is put into a nice graph it's easy to forget that everything might not be there. 

[![NYT visualization example](http://iibawards-prod.s3.amazonaws.com/app/public/ckeditor_assets/pictures/218/content_screen_shot_2015-12-10_at_01_06_30.png)](https://www.nytimes.com/interactive/2015/03/19/upshot/3d-yield-curve-economic-growth.html)

I mean, look at that graph! The first thing that I'm thinking is definitely not "Hm, I wonder if the data behind this graph is representative of everything." Much more likely, the first thing I'm thinking is, "Yes, do take me on this visual tour of economic growth!" There is an implicit trust in the graph maker, who spent so long making such a beautiful graph, that their data is accurate as well. 

This becomes dangerous knowing that data, the fundamental unit on which these graphs rely on, can be skewed. Take, for example, the data on sexual assault on college campuses. If this data were graphed, it would appear that certain schools have a very high rate of sexual assault while others have none. Looking at a graph like this, it's easy to conclude that sexual assault only happens at these crazy schools, but not at the schools that have zero reported incidents. In reality, the reporting rate corresponds more to the school environment and whether or not it is a space [where people can feel comfortable reporting](https://www.theatlantic.com/education/archive/2016/01/why-the-prevalence-of-campus-sexual-assault-is-so-hard-to-quantify/427002/). So how do we work towards making sure our data does not exclude voices?

## Community Driven Data
The best example of community driven data is probably Wikipedia. Consider this excerpt:

> "Comparing the shifting of authority from the *Encyclopedia Britannica* to Wikipedia-- an authoritative collection of experts vs a self-organizing community of bookworms for the common good-- is a great indicator of this phase change. In 2005, *Nature* published a study that revealed that the two were comparable in quality. Since then, we have witnessed the steady ascension of Wikipedia. Capable not only of instantly responding to new information (a celebrity's death, the onset of hostilities between two rival factions) but fostering dissent, deliberation, and ultimately consensus on how that information should be presented." -- *Whiplash: How to Survive Our Future Faster*, by Joi Ito and Jeff Howe

So even though anybody can edit Wikipedia, eventually, and sometimes after lengthy edit wars, the information that ends up on the site is often as accurate as an encyclopedia entry written by experts. And since *anybody* can edit Wikipedia, this seems like a start for how to capture data that might not otherwise be represented. 

# Back to Wikigraphs
As the name suggests, the idea of wikigraphs is a combination of Wikipedia's data collection model with data driven graphs.

## An Example
Recently, I decided that I wanted to do some sort of data visualization on Asian American authors, and maybe something about the coverage they get from the media. The easiest data source to start from seemed to be the [New York Times Books API](https://developer.nytimes.com/books_api.json), which lets you get a list of book reviews they have done for books by a given author. Great! The next step, then, was to get a list of Asian American authors.

What does it mean to be an Asian American author? That's not an easy question to answer, and any answer that I gave alone would not cover everybody's definition. And of course, I, as one person, simply don't know all of the Asian American authors out there! I came up with what I saw as an initial list with the intent that the list could be modified in the future, possibly by other people. 

### First, a graph
I decided to make a graph that showed the number of reviews a given author has gotten from the NYT, as well as the ability to see a timeline of when these reviews were published. 
![graph example](https://raw.githubusercontent.com/allisonking/wiki-graphs/master/readme-imgs/shift_data.gif)

There's some additional functionality as well-- you can hover on points to see the book that was reviewed, who reviewed it, and clicking on the circle redirects you to the actual review. This data was gathered by first compiling a list of names, then querying the NYT API for the data, which was stored as a JSON and read in by D3. It's easy to stop here, with a reasonable looking graph with a good amount of interactivity, but the data is incomplete. 

### Next, user input
The pipeline so far looks like this:
![first pipeline](https://raw.githubusercontent.com/allisonking/wiki-graphs/master/readme-imgs/pipeline_2.png)

What we want though, is something more like this:
![second pipelines](https://raw.githubusercontent.com/allisonking/wiki-graphs/master/readme-imgs/pipeline_2.png)

So what if we could let people reading the graph alter the underlying data if they see something missing or incorrect? To trust unknown people to gather data and do so correctly may seem risky, but we have seen that it can succeed in Wikipedia. 

In order to accommodate data changes, I added a database and server side code (in the form of MongoDB and Flask since they are both minimal and quick to implement, with MongoDB not needing a database schema and Flask being a microframework). To the front end, I added the ability to query the NYT API dynamically given a user input, as well as the ability to remove an author.
![second pipelines](https://raw.githubusercontent.com/allisonking/wiki-graphs/master/readme-imgs/add_delete.gif)

With the database in place, all user modifications are now saved and logged.

## Concluding Thoughts
Wikigraphs is a proof of concept of how user discussion/discourse may lead to more representative graphs. There are still questions outstanding as well as improvements that could be made:
* Displaying user activity, i.e. when authors are added or removed, to the side, or on a separate page, may be useful, if only to remind readers that the data should not be trusted without second thought
* Adding a database and server side code is a non-trivial task. Could a tool be developed to make this all easier? Hosting the data in a cloud environment meant for this kind of data manipulation, for example?
* The example model trusts the NYT API as a source to reliably return the reviews for a given author. If there were a bug in the API, or if the database somehow missed out on some sources, this would not be reliable, and perhaps this part, too, could also be editable by users
* What about just using Wikipedia for data? Certainly easier than setting up another framework, but I see a difference between a link that says 'data from ___ Wikipedia page', where a user might not think to go edit that page, and actually being prompted by the graph itself to edit the underlying data

Finally, an acknowledgment-- a lot of this line of thinking was inspired by the article [*What Would Feminist Data Visualization Look Like?*](https://civic.mit.edu/feminist-data-visualization) by Catherine D'Ignazio, so super thanks to her work. 

Thanks for reading! I still need to actually go out and make this project hosted somewhere, so stay tuned! In the meantime, [here's the repository](https://github.com/allisonking/wiki-graphs/tree/readme).
