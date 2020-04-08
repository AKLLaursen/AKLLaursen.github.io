---
layout: post
author: andreas
---
Greetings.

Welcome to what is to be my online home. Having finished [university](http://www.econ.au.dk) and started my first (post academia) full-time [job](https://www.nationalbanken.dk/en/about_danmarks_nationalbank/organisation/Pages/Statistics.aspx), I decided to make a place to document the projects on which I will spend some of my newly found spare time.

This will not be a blog in the traditional sense, but simply a place for me to log what I create while working on my hobbies. These hobbies being statistical programming and financial/economic data analysis. Hence the (current) name of the website, Econ and Data. What I will be doing is logging my work on the web, i.e. writing a weblog. I love learning new things and the site is also meant to inspire myself to learn more about the things that interest me. Through writing on this site, I also hope to improve my writing skills. Though I'm fluent in English, my writing has so far been very academic, and thus I have little experience writing less formal texts.

Having dappled with web application development in [R](https://www.r-project.org/) through the [Shiny](http://shiny.rstudio.com/) web application framework, I also wanted to use the creation of the site as a learning opportunity. That is to increase my knowledge about HTML, CSS, JavaScript and preferably a more general web framework.

I started by listing how I wanted to built the site, and the features I wanted to implement. This is a summary of what I came up with:

* I wanted to base the HTML and CSS on the [Bootstrap](http://getbootstrap.com/) framework. [Shiny](http://shiny.rstudio.com/) ui design is very much based on Bootstrap, so I was already pretty familiar with it.
* The site should support the [Markdown](https://daringfireball.net/projects/markdown/) mark-up language. I already use Markdown daily at work and when taking [notes](http://1writerapp.com/) on my mobile devices.
* The site should also support coding blocks with syntax highlighting of the languages I use the most.
* I didn't want to use a standard content management system like [WordPress](https://wordpress.org/), as
  1. It would be boring.
  2. I wouldn't learn anything new.
  3. I wanted to do my own customisations and certainly didn't want to learn PHP, if I could avoid it.
* In designing my own CMS, I wanted to use a language and a framework that would be both fun to learn and useful to know professional perspective. I had already used [Python](https://www.python.org/) a bit, and wanted to learn more as the syntax and the general aesthetics of the language pleased me quite a lot. As Python is used more and more for statistics and data analysis, It also made sense to add it to the list of languages I already knew. Finally, I knew that a very interesting web framework called [Django](https://www.djangoproject.com/) was written in Python.
* I wanted to do things *the right way*, and use best practises of web development as much as possible. I knew that Django has excellent support for testing, so I also wanted to try my hands at test driven development.
* I wanted to self-host the site in order to learn a bit about server management and web-hosting in general. I also wanted to be able to integrate small separate web applications into the site.

I started by catching up on the basics of the Python language. All my knowledge of programming is basically self-taught. Having studied economics, I was never able to take any formal computer science courses at university. Thus I used the fact that I was going to learn Python as an opportunity to also read up on basic computer science concepts. I read John Zelle's introductory [book](http://mcsp.wartburg.edu/zelle/python/) on computer science and while I didn't really learn anything new, it was nice to have familiar concepts presented in a cohesive way for the first time.

Next, I went through a few Django tutorials and read through the [documentation](https://docs.djangoproject.com/en/1.9/) in order to get a general overview of the web framework. I soon discovered that with the experience I had, it would be quite a daunting task to design a web application for, what would basically be a blogging engine, from the bottom. All the web applications I had designed so far were data visualisation tools and were never as complex as a content management system. Thus I started browsing the web for tutorials specifically aimed at creating a blogging engine in Django.

I soon discovered that a guy called Matthew Daly had written not only an excellent [Django blog tutorial](http://matthewdaly.co.uk/blog/2013/12/28/django-blog-tutorial-the-next-generation-part-1/), but a tutorial designing a blog engine that supported all the features I had listed. In addition, the turioal also brought a bunch of useful tools to my attention, my favourites being:

* Package management for web development using [Bower](http://bower.io/)
* Continuos integration using [Travis CI](https://travis-ci.org/).
* Tracking of code coverage using [Coveralls](https://coveralls.io/).
* Automatic deployment trough Travis CI.
* Caching using [Memcached](http://memcached.org/)
* Optimising and minimizing static files using [Grunt](http://gruntjs.com/)

So in the end, I based the site on the code from this tutorial, though I made a few changes while going along:

* I wrote the site in Python 3 instead of Python 2.
* I used Django 1.9 and thus didn't use South for migration.
* Though my site is hosted on [Heroku](https://www.heroku.com/) at this point, I will be hosting it on a visual Linux server in the cloud through [DigitalOcean](https://www.digitalocean.com/). Ubuntu will be the platform as Ubuntu has been the OS of my private workstation for the last couple of years.
* I did not implement a comment system, as I dislike this system of communication on other sites.
* I added some archive views, to get the weblog posts shown in a full archive and a monthly archive.

The source code of the site is available at [Github](https://github.com/AKLLaursen/gamgee) released under the GPL v2.0 licence. I named the project *gamgee*, because when building it, I felt like a certain naive [character](https://en.wikipedia.org/wiki/Samwise_Gamgee) from one of my favourite novels going on an adventure. The site is still under development, and can at this point be considered an early alpha build. Hopefully, I will expand upon it in the months to come - that is unless something more interesting catches my attention.

Being close to finishing the site, my head is already brimming over with ideas of what I want to next, and I cannot wait to get started. It is time to step into the road, let go of my feet and see where I'm swept off to.