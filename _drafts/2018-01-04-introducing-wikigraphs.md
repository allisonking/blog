---
published: false
layout: post
title: Introducing Wikigraphs
---
For the past few weeks, I have been working on a project that I am currently calling Wikigraphs. What in the world is that, you ask? Well, thanks for asking! If I had to summarize it, I'd say it is a framework that allows for community driven data as input into data driven graphs, thereby allowing for transparent data sources.

Let's break that down. 

### The Data Driven Graph
There are a lot of great data driven graphs out there. D3js, the popular JavaScript library, stands for Data Driven Documents, after all, and powers many of those awesome *New York Times* visualizations. These polished graphs are easily interpreted as objective and authoritative-- after all, data does not lie, right? But data can omit and misrepresent, and when that is put into a nice graph it's easy to forget that everything might not be there. 

[![NYT visualization example](http://iibawards-prod.s3.amazonaws.com/app/public/ckeditor_assets/pictures/218/content_screen_shot_2015-12-10_at_01_06_30.png)](https://www.nytimes.com/interactive/2015/03/19/upshot/3d-yield-curve-economic-growth.html)

I mean, look at that graph! The first thing that I'm thinking is definitely not "Hm, I wonder if the data behind this graph is representative of everything." Much more likely, the first thing I'm thinking is, "Yes, do take me on this visual tour of economic growth!" There is an implicit trust in the graph maker, who spent so long making such a beautiful graph, that their data is accurate as well. 

This becomes dangerous knowing that data, the fundamental unit on which these graphs rely on, can be skewed. Take, for example, the data on sexual assault on college campuses. If this data were graphed, it would appear that certain schools have a very high rate of sexual assault while others have none. Looking at a graph like this, it's easy to conclude that sexual assault only happens at these crazy schools, but not at the schools that have zero reported incidents. In reality, the reporting rate corresponds more to the school environment and whether or not it is a space [where people can feel comfortable reporting](https://www.theatlantic.com/education/archive/2016/01/why-the-prevalence-of-campus-sexual-assault-is-so-hard-to-quantify/427002/). So how do we work towards making sure our data does not exclude voices?

### Community Driven Data


> "Comparing the shifting of authority from the *Encyclopedia Britannica* to Wikipedia-- an authoritative collection of experts vs a self-organizing community of bookworms for the common good-- is a great indicator of this phase change. In 2005, *Nature* published a study that revealed that the two were comparable in quality. Since then, we have witnessed the steady ascension of Wikipedia. Capable not only of instantly responding to new information (a celebrity's death, the onset of hostilities between two rival factions) but fostering dissent, deliberation, and ultimately consensus on how that information should be presented." -- *Whiplash: How to Survive Our Future Faster*, by Joi Ito and Jeff Howe
