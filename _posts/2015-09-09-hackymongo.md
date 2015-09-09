---
layout: post
title: Probably the Hackiest DB Duplication Handling Ever
date: 2015-09-08
categories: code mongo
---
A project I am currently working on deals with big data. (It's called Gitit, check it out [here](https://github.com/Gitit-TGA/Gitit).) It's not BIG big data, but it's still pretty damn big. This has lead to a lot of database problems that I've never encountered before. I've had to deduplicate data before, but not on the scale of minutes per query. Upsert isn't an options when you have a db with millions of documents.

Here's my hacky solution for making sure I don't duplicate data:

First I query the results collection, which is our source of truth

```
Results.find(function(err, data){});
```

Second, I stringify the results object (I told you this was hacky). Javascript has no limit for string size, and searching a string is faster than a db call.

```
Results.find(function(err, data){
  data = JSON.stringify(data);
});
```

On every item I want to insert, I check if the value is in the results.

```
Results.find(function(err, data){
  data = JSON.stringify(data);
  records.forEach(function(row) {
    var current = {};
    current.repo_name = row.repo_name;
    current.repo_url = row.repo_url;
    var reg = new RegExp(current.repo_name);
    if(data.match(current.repo_name === null)){
    }
  });
});
```


If the value isn't in the string, push it to the db

```
Results.find(function(err, data){
  data = JSON.stringify(data);
  records.forEach(function(row) {
    var current = {};
    current.repo_name = row.repo_name;
    current.repo_url = row.repo_url;
    var reg = new RegExp(current.repo_name);
    if(data.match(current.repo_name === null)){
	    parsed_records.push(current);
    }
  });
  saveUrlsToDB();
});
```

"But Ryan couldn't you just set the value to be unique in Mongo?"

Yes, but that was taking up precious milliseconds.

I've found that when performing hundreds of thousands of queries, every uneccessary db call is extremely noticeable in the runtime. By making one giant call, and then searching with regex, is much faster. By doing this I've shaved hours off of our file parsing process. Is it hacky? Yes of course, but sometimes the best things in life are. It wasn't too long ago that I was using recursion to generate an sql query that was multiple millions of characters long. Compared to that, this is a pretty elegant solution.
