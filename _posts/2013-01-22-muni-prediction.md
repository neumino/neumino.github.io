---
layout: post
category : Geek
tags : [rethinkdb, mapreduce, muni, bus]
title: Case study with RethinkDB Muni's predictions
---
{% include JB/setup %}

_Note_: RethinkDB has changed the API for the JavaScript driver and these queries are outdated.

<a href="http://www.sfmta.com" title="Muni">Muni</a> stands for San Francisco Municipal Transportation Agency and is the agency running all buses in San Francisco. I'm taking the bus every weekday to go to/come back from the Caltrain station.

Muni provides <a href="http://www.sfmta.com/cms/asite/nextmunidata.htm" title="Muni's nextbus api">an api</a> to get at what time the next bus will arrive.
I have noticed that the predictions are not always accurate and sometimes completely off. So I thought it would be nice to quantify these inaccuracies and play with ReQL, <a href="http://www.rethinkdb.com">RethinkDB</a>'s query language.

<em>Note: I am working at RethinkDB but this article is just the result of me playing around during a week end.</em>

First let's retrieve some data and store it in the database. I have written a script that pulls data from Muni's API every 20 seconds and store it in the database.
It is written in CoffeeScript and use node.js with the http and xml2js libraries.
You can read the <a href="https://raw.github.com/neumino/muni/master/pull.coffee">raw file</a> or clone the <a href="https://github.com/neumino/muni">github repository</a>

Pulling data for all the stops of the line 48 during one day (on a Thursday) filled my table with 500.000 entries.

Now come the interesting part. I could retrieve all the data in the table and make some operations on top of it using coffee-script/python/ruby, but that would be cheating. RethinkDB aims to be able to run long analytics queries, so I gave it a try.

The documents I have stored in my database have the following format.

```
{
    "stop_tag": 3512,        // stop's id
    "next_bus_sec": 1148,    // The next bus is in 1148 seconds
    "next_bus_min": 19,      // The next bus is in 19 minutes
    "vehicle": 8430,         // The id of the next vehicle coming in 19 minutes
    "time": 1356935060062,   // The time when the prediction was made
    "id": "0003ac31-e903-4ec1-9184-3dfee272fd48" // id of the document
}
```

If I order my rows by "time" and map the value of next_bus_min. I would get something like that

```
[7, 6, 5, 4, 3, 2, 1, 0, 10, 9, 8, 8, 7, 6, 5, 5, 4, 3, 3, 2, 1, 0, ...]
 |              |
 |          The next bus is in 2 minutes
 The next bus is in 7 minutes
```

And here is what I want to retrieve is for each prediction: the prediction minus how long the user did wait.

```
How long the user waited
<-------------------->
[7, 6, 5, 4, 3, 2, 1, 0, 10, 9, 8, 8, 7, 6, 5, 5, 4, 3, 3, 2, 1, 0, ...]
 |                    |
 |             Next bus in 0 minute = The bus is here
 |                    
 Next bus will be in 7 minutes
```

To build the query, I used the data explorer because it provides a nice interface to test my query.
I don't have an extra table with a list of stops available, so I first retrieved all the stops using pluck() and distinct()

```js
r.db('muni').table('predictions').pluck('stop_tag').distinct().run()
```

Now for each stops, I will retrieve the list of predictions we have. 
This can be done with an inner join query and a groupBy. I just went for a simple map.
One think about ReQL is that implicit variable (r.row) cannot be used in case of nested queries, so I have to use anonymous functions (also known as lambda functions in Python) for the next steps. 


```js
r.db('muni').table('predictions').pluck('stop_tag').distinct()
    .map( function(stop) { return {
        stop_tag: stop('stop_tag'),
        predictions: r.db('muni').table('predictions').filter( function(prediction) {
                return prediction('stop_tag').eq(stop('stop_tag')) 
            }).orderBy('time').streamToArray()
    }})
    .run()
```

Now that I have for each stop all the predictions, I would like to know how accurate the prediction is. 
I will consider that the time when the bus arrive is the next prediction that show the next bus in 0 minute. So I have to join the inner query with itself. To avoid computing this stream 1+n time (which can be expensive because I use a sort every time), I'm going to use r.let() and store it in memory.

The syntax for r.let() now is

```js
r.let( { "key": <object Obj we want to store> }, <query that uses Obj>).run() 
```
<em>Note: The syntax for r.let() will change on the next release (1.4).</em>

The previous query with r.let() looks like that:

```js
r.db('muni').table('predictions').pluck('stop_tag').distinct()
    .map( function(stop) {
        return {
            stop_tag: stop('stop_tag'),
            prediction_error: 
            r.let(
                {
                    predictions: r.db('muni').table('predictions')
                        .filter( function(prediction) {
                            return prediction('stop_tag').eq(stop('stop_tag'))
                        })
                        .orderBy('time')
                        .streamToArray()
                }, 
                r.letVar('predictions')
            )
        }
    })
    .run()
```


Now let's cross the stream with itself and get for each prediction all the times when a bus arrived.


```js
r.db('muni').table('predictions').pluck('stop_tag').distinct()
    .map( function(stop) {
    return {
    stop_tag: stop('stop_tag'),
    prediction_error: 
    r.let(
        {
            predictions: r.db('muni').table('predictions')
            .filter( function(prediction) {
                return prediction('stop_tag').eq(stop('stop_tag'))
            })
            .orderBy('time')
            .streamToArray()
        }, 
        r.letVar('predictions').map( function(prediction) {
            return r.letVar('predictions').filter(function(next_prediction) {
                return next_prediction('next_bus_min').eq(0)
                    .and(prediction('time').le(next_prediction('time')))
                })
        })
    )
    }})
.run()
```


Now that I have all the data I need, I just have to do some math to get the error.
So among all the prediction that we retrieve in the inner query, we just need the first one (the next time the bus arrive). Using nth(0) could break in case a bus never comes (calling .nth(0) on an empty array). A solution to make sure that nth(0) is not going to break is to use r.branch (which is the syntax for "if"). I was lazy so I just used a trick adding an element that is going to yield a zero error.
Then we can safely compute the error.


```js
r.db('muni').table('predictions').pluck('stop_tag').distinct()
        .map( function(stop) {
        return {
            stop_tag: stop('stop_tag'),
            prediction_error: 
            r.let(
                {
                    predictions: r.db('muni').table('predictions')
                        .filter( function(prediction) {
                            return prediction('stop_tag').eq(stop('stop_tag'))
                        })
                        .orderBy('time')
                        .streamToArray()
                }, 
                r.letVar('predictions').map( function(prediction) {
                    return r.letVar('predictions').filter(function(next_prediction) {
                    return next_prediction('next_bus_min').eq(0)
                        .and(prediction('time').le(next_prediction('time')))
                    })
                    .union([{time: prediction('time').add(prediction('next_bus_min').mul(60*1000))}])
                    .nth(0)('time').sub(prediction('time')).div(60*1000).sub(prediction('next_bus_min'))
                })
            )
        }
    })
    .run()
```


So the error now looks like this:

```js
{
    "stop_tag":6744,
    "prediction_error":[
        -0.001300000000000523,
        -0.3384333333333327,
        -0.665916666666666,
        0.00016666666666687036,
        /* more... */
    ]
}
```

The following query compute the norm 1 error.

```js
r.db('muni').table('predictions').pluck('stop_tag').distinct()
    .map( function(stop) {
        return {
            stop_tag: stop('stop_tag'),
            prediction_error: r.let(
                {
                    predictions: r.db('muni').table('predictions')
                    .filter( function(prediction) {
                        return prediction('stop_tag').eq(stop('stop_tag'))
                    })
                    .orderBy('time')
                    .streamToArray()
                }, 
                r.letVar('predictions').map( function(prediction) {
                    return r.letVar('predictions').filter(function(next_prediction) {
                        return next_prediction('next_bus_min').eq(0)
                            .and(prediction('time').le(next_prediction('time')))
                    })
                    .union([{time: prediction('time').add(prediction('next_bus_min').mul(60*1000))}])
                    .nth(0)('time').sub(prediction('time')).div(60*1000).sub(prediction('next_bus_min'))
                })
                .map(function(error) { return r.branch(error.lt(0), error.mul(-1), error) })
                .reduce(0, function(acc, val) { return acc.add(val)})
                .div(r.letVar('predictions').count())
            )
        }
    })
    .run()
```

Since I am looking at the data in term of user experience, I am also (and mostly) interested in the worst case. So here is how I got the maximum error:

```js
r.db('muni').table('predictions').pluck('stop_tag').distinct()
    .map( function(stop) {
        return {
            stop_tag: stop('stop_tag'),
            prediction_error: r.let(
                {
                    predictions: r.db('muni').table('predictions')
                        .filter( function(prediction) {
                            return prediction('stop_tag').eq(stop('stop_tag'))
                        })
                        .orderBy('time')
                        .streamToArray()
                }, 
                r.letVar('predictions').map( function(prediction) {
                    return r.letVar('predictions').filter(function(next_prediction) {
                        return next_prediction('next_bus_min').eq(0)
                            .and(prediction('time').le(next_prediction('time')))
                    })
                    .union([{time: prediction('time').add(prediction('next_bus_min').mul(60*1000))}])
                    .nth(0)('time').sub(prediction('time')).div(60*1000).sub(prediction('next_bus_min'))
                })
                .reduce(0, function(acc, val) { 
                    return r.branch(val.gt(acc), val, acc)
                })
            )
        }
    })
    .run()
```

The results look like that:

```js
{
    "stop_tag": 3248,
    "prediction_error": 35.01116666666667
},
{
    "stop_tag": 3249,
    "prediction_error": 6.006866666666667
},
{
    "stop_tag": 3250,
    "prediction_error": 35.333483333333334
},
{
    "stop_tag": 3251,
    "prediction_error": 35.33631666666666
},
{
    "stop_tag": 3252,
    "prediction_error": 6.002116666666666
},
{
    "stop_tag": 3304,
    "prediction_error": 5.670216666666667
},
{
    "stop_tag": 3305,
    "prediction_error": 34.669200000000004
},
{
    "stop_tag": 3306,
    "prediction_error": 6.00385
},
{
    "stop_tag": 3411,
    "prediction_error": 87.99973333333334
},
{
    "stop_tag": 3424,
    "prediction_error": 16.666383333333332
},{
    "stop_tag": 3432,
    "prediction_error": 87.0006
}
...
```

So Muni's prediction are quite aweful. I've tried to check for predictions with a bus coming at 1 and 0 minutes, but that didn't really change the results.
I have seen this thing happening during the evening and from my personal experience, it is because the system that makes the predictions does not know when a bus is going to the garage until it does go there.

Back to the technical part, it was pretty cool to use the data explorer to build an analytic query without having to write a script myself.
The javascript driver doesn't always return user-friendly errors which is not really cool, but that should be fix on the next release with the new protobuf specs.
