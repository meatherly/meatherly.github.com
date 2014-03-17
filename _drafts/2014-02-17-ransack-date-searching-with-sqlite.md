---
layout: post
title: "Ransack date searching with SQLite"
category: ruby
tags: [ransack, sqlite, mysql, dates, rails]
---
{% include JB/setup %}

This past week I was working on a project that was using the [ransack gem](https://github.com/activerecord-hackery/ransack) to search for dates. I was using MySql for my developement db and SQLite for my test db. Everything was working perfectly in development but when I was testing the feature nothing was working right. I would make 3 objects with dates ranging from 2014-01-01 to 2014-01-03. When I filled in my date text boxes and hit search none of them would return. I could not figure out why. I searched for answers and nothing came up. After looking into the database at the dates that was stored I noticed something strange. The date stored in MySql looked like: `2014-01-01` but the date in SQLite looked like `2014-01-01 12:00:00`. This might be problem I thought. This led me to look into how SQLite stores dates. 

Believe it or not SQLite doesn't have a date type. It using some weird hackery to store the date as a string then expects your code base to parse it in to a date when you retrieve it. This is what the [SQLite Docs](http://www.sqlite.org/datatype3.html) say about the date type

>SQLite does not have a storage class set aside for storing dates and/or times. Instead, the built-in Date And Time Functions of SQLite are capable of storing dates and times as TEXT, REAL, or INTEGER values

*Simple fix to solve this problem is to make sure you format the date before you save it in SQLite.* 

For instance if you were to create an object and store the date like 
    Date.today
    2.days.ago
Then it would create a date like this in SQLite ``

----


I'm going to create an event app. This app allows you to create an event with a title and date of the event. Then you can search all your events. I'll be using MySql for the development db and SQLite for the test db.

First we'll create the rails app. Here is the rails wizard link to help speed up the process <http://railswizard.org/879e7d530221adfa0578>

