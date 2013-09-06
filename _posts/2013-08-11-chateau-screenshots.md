---
layout: post
category : Geek
tags : [chateau, rethinkdb, phpmyadmin, dataexplorer, screenshots]
title: Chateau - Few screenshots
---
{% include JB/setup %}

Quick post about Chateau because I have way too many things to finish this week end -- and too many things that I won't have time to do :'(.

You can browser you databases and tables:  
<a href="{{ site.url }}/assets/images/Screenshot-from-2013-08-11-092119.png">See screenshot</a>

Basic operations are available on databases and tables. You can create/delete them. Chateau relies only on the JavaScript driver now, so renaming a table is no more available for the moment.  
<a href="{{ site.url }}/assets/images/Screenshot-from-2013-08-11-092141.png">See screenshot</a>

Documents are displayed in a table similar to the RethinkDB native web interface.  
<a href="{{ site.url }}/assets/images/Screenshot-from-2013-08-11-092126.png">See screenshot</a>

Hovering on a document shows some actions available on the left:  
<a href="{{ site.url }}/assets/images/Screenshot-from-2013-08-11-092156.png">See screenshot</a>

You can delete a document just by clicking on the trash icon and confirming the deletion.  
<a href="{{ site.url }}/assets/images/Screenshot-from-2013-08-11-092200.png">See screenshot</a>

You can also update the document.  
<a href="{{ site.url }}/assets/images/Screenshot-from-2013-08-11-092208.png">See screenshot</a>

Eventually you can add a new document. Because RethinkDB doesn't enforce any schema, Chateau is going to sample your table and create a schema based on the documents it retrieves. So you probably should just have to fill the fields and not create one.  
<a href="{{ site.url }}/assets/images/Screenshot-from-2013-08-11-092219.png">See screenshot</a>

More features are coming.
Feedback and pull requests are are appreciated : )
