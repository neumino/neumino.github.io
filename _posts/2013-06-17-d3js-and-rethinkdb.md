---
layout: post
category : Geek 
tags : [d3js, rethinkdb, graph, visualization, rottentomatoes]
title: Playing with D3js and RethinkDB
---
{% include JB/setup %}

Here is my last week end project.

- [Demo](http://justonepixel.com/projects/tomatoesfridge/index.html)
- [Source code](https://github.com/neumino/tomatoesfridge)

This project has three purposes/reasons:

- Kill time
- Play with [RethinkDB](http://www.rethinkdb.com)'s python driver
- Draw cool stuff with [D3.js](http://www.d3js.org)

The project is about drawing a graph of similar movies. The users can expand the graph by clicking on a node. Because I am limited to 10 requests per second, I though it wouldn't be stupid to cache things. Instead of caching things in memory with a home-made memcached, I though it would funnier to use RethinkDB. 

So here is what happens every time a user ask for some data

- We check if we have the data in the database
- If we do, we just return the results
- If we don't, we fetch some data from rottentomatoes, dump it in RethinkDB and send it back to the user.


The main components of the used stack are <a href="flask.pocoo.org">Flask</a>, <a href="http://www.rethinkdb.com">RethinkDB</a> and <a href="http://www.d3js.org">D3.js</a>. There are some secondaries components too like <a href="www.cherrypy.org">Cherrypy</a>, <a href="twitter.github.com/bootstrap/">Bootstrap</a> and <a href="http://jquery.com/">Jquery</a>.

I would note two things from this project

- While the force layout always converge to a "good" representation, it is quite unstable during the first ticks. I tried to tweak a little the algorithm to make it less oscillating at the begining, but could not do it to a certain point.
- It's pretty cool to write queries like

```py
movie = r.table("movie").get(id_movie).do( lambda movie:
    r.branch(
        # If we didn't find the movie or didn't find the similar movies
        (movie == None) | (~movie.has_fields("similar_movies_id")), 
        # We just return the movie/None
        movie,
        # Else we add a field similar_movies with the relevant data
        movie.merge({
            "similar_movies": movie["similar_movies_id"].map(lambda similar_movie_id:
                r.table("movie").get(similar_movie_id)
            )
        })
    )).run( g.rdb_conn )
```

I have started to implement the search feature, but I'll leave it as it is I think. I have other cool idea in my mind :-)
