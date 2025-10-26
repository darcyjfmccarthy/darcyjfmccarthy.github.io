---
title: "Discovering VGC Archetypes with Clustering"
date: 2025-10-23T10:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["first"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Using data science to help with regional prep"
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/<path_to_repo>/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

## Motivation

In general, when I have a regional coming up, one of the first questions I have is _"what am I most likely to play against?"_. Getting a good answer to this makes an enormous difference - I think we've all had experiences of bringing the wrong team to a tournament and having a pretty bad weekend because of it. 

Despite this being a problem I assume everyone has though, there's pretty much no way to find good answers. You've got a few options:
- Make some really basic inferences about what was played last weekend (what won? what was common?)
- Play lots of ladder and hope that whatever you notice there translates to your tournament
- Look at some really basic stats online (e.g. usage rates of indiviudal or combinations of Pokemon)

For a while now I've been frustrated that there seems to be nothing better than this. VGC is a growing sport, and filled with really smart people, yet the most detail you can get on what's being played right now is just usage rates of individual pokemon. I'm a strong believer that this is just not helpful info - how much do I care if Incineroar usage has gone up 5%? Am I likely to make any big decisions off that information? Probably not. More than that, knowing what Pokemon are getting used doesn't even particularly tell you what your matchups will be, since knowing the context of the other Pokemon on the team is really important. For example, let's say we find out that not only are Incineroar and Rillaboom the two most used Pokemon, but they're also on the same team 80% of the time. That's still not helpful information - we knew that intuitively to begin with, and the teams that those two Pokemon are on are generally defined by the 4 Pokemon that aren't Incineroar and Rillaboom.

On top of the current state of Pokemon usage stats not being useful, it also just generally conflicts with the way we understand Pokemon teams. Generally in popular discourse we talk about 'cores' or 'archetypes' - things like (speaking from a Regulation H lens here) rain, DUG, etc. And yet, no one seems to have figured out a good way to tell you what archetypes exist, and which ones you can expect to play against on a given weekend. It seems to me that having information on this would be _much_ more valuable than what we have right now. This article is my attempt to create something that fills that gap - a data driven list of team compositions that would adequately prepare you for your next regional.

**(If you don't care about the methodology, click here to see the current archetypes in VGC26 Regulation H)**

## Methodology

### Data
For the raw data, I downloaded the tournament results JSONs from https://pokedata.ovh/standingsVGC/. This data is complete with names, results, matchups, and open teamsheets.

We then need a way to transform this into something a computer can understand. The approach I took was a really simple one-hot encoding, where the rows of the final matrix were a tournament entry (i.e. a primary key of player | tournament), and the columns were binary indicators showing the presence or absence of pokemon X on team Y. It looks something like this (but with >400 columns):

| Player | Tournament | Rillaboom | Incineroar | Spidops |
| ------ | ---------- | --------- | ---------- | ------- |
| Mr. A  | Frankfurt  | 1         | 1          | 0       |
| Ms. B  | Frankfurt  | 0         | 1          | 0       |
| Mr. A  | Lille      | 0         | 0          | 1       |

We now have an N-length vector for each team, where N = the total number of unique Pokemon seen across the dataset, and each row contains six values of 1, with every other value at 0.

There's a couple of issues with this representation:
- A lot of the columns are predictably going to end up not mattering. Intuitively, we know that there isn't going to be a archetype in the metagame defined by (for example) Piplup, even if someone did bring it to a regional. You may or may not be surprised to learn that, particularly at the bottom end of tournaments, there are lots of Pokemon that only appear once. This is going to distract our model - we'll need a way to get rid of these
- This matrix is extremely sparse. Each row contains almost no information - almost every value is necessarily zero! This is bad for the geometry of our clustering and likely leaves a lot of improvement on the table. The results (as we'll see later) are ok, so I leave this to future work.

For the first point, the solution for this is pretty easy - we just filter out Pokemon that appear below a certain percentage. For my testing, I find removing Pokemon that appear less than 3% of the time generally improves the modelling significantly.

### Clustering 

Once the data is transformed and cleaned, we're ok to start clustering. For this article, I've used agglomerative hierarchical clustering, since I think it's a really logical fit for the problem at hand. It works like this:
1. Start by assuming every single team is its own cluster. Obviously this is not true, not in the least because many people bring exactly the same team, so;
2. At each step, we merge together the most similar pairs of teams. At the start of the process this will be grouping together identical teams, but later we'll be merging together teams with some differences.
3. Stop when we've reached an appropriate number of clusters. There's various ways to define 'appropriate', but for this project I picked the number with the highest silhouette score for clusters counts between 2 and 20.

Hopefully it's clear what this technique does and why it was a good fit for this task. If not, here's a good breakdown: https://www.geeksforgeeks.org/machine-learning/hierarchical-clustering/


## Archetype Identification

After clustering, there's still a little bit of work to do before we get our final set of archetypes. To translate our data into something actionable, we need to actually pull out what 'defines' each cluster.

To do this, I've taken any Pokemon that appears in >50% of the teams in its cluster to be the 'archetype' or 'core' we're concerned about. Additionally, if an archetype appears in <1% of teams, I've eliminated that entirely, as it likely isn't a great descriptor of the metagame.

After doing these steps, we get a realistic and clearly interpretable set of team compositions. Time to take a look!

## Results
Before we break down the clusters one by one, here's a graph showing them in 2D space. You can see the names and Pokemon of each cluster on the right.
Each dot here represents a real team that was played in Milwaukee & Belo Horizonte regionals. You can hover over them to see the player name, their full team, and their record. See if you can try find yourself!

_(Note: different to what you might be used to, the x and y axes don't particularly 'mean' anything here, at least not on purpose - they are just the two most informative dimensions onto which we can project these points. Exercise for you as the reader: see if you can try interpret them!)_

<div style="position:relative; width:150%; height:0; padding-bottom:75%;">
  <iframe
    src="/pokemon_archetypes.html"
    style="position:absolute; top:0; left:0; width:100%; height:100%; border:none;"
  ></iframe>
</div>

## What next?