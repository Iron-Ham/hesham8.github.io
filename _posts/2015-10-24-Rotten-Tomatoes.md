---
layout: post
title: "Let's Build: A Rotten Tomatoes iOS App"
excerpt: "Exploring the Rotten Tomatoes API"
tags: [swift, ios]
comments: true
published: true
---
<section id="table-of-contents" class="toc">
  <header>
    <h3>Overview</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

> Part 1

If you're following along, you should start by obtaining your own [Rotten Tomatoes API key](http://developer.rottentomatoes.com). Read the developer docs on the RT developer page, create your account, and apply for an API key. 

If you're not used to using APIs and their structure, read the [Rotten Tomatoes developer documentation](http://developer.rottentomatoes.com/docs/read/Home).

In this example, we're going to present create a few custom kinds of UITableViewCell and present them in separate sections in a UITableView. Let's start with the Box Office Movies.

The template for this link is:

http://api.rottentomatoes.com/api/public/v1.0/lists/movies/box_office.json?apikey=[your_api_key]

Let's begin with our RestApiManager class.

## The Rest API Manger
