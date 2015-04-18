---
layout: post
title: "Javascript Callbacks vs Promises"
date: 2015-04-18 11:35:08 +0200
comments: true
categories: javascript, promises
---

Promises are used to deal with asynchronous and concurrent programming. If you are familiar with callbacks, it's worthy to mention that
promises and callbacks are similiarily used to specify what should happen when a result from an asynchronous and non blocking
task that your program may be waiting for becomes available. The most noticeable difference for me is on the way you should
structure your code to handle the returned values from such async calls. To the examples!


###Reading files with callbacks

Most of the built in libraries in Node.js are based on callbacks. If you want to open and read a file, you should supply
a callback that is going to be executed when the data from the file (or an exception) is returned.

``` javascript
var fs = require('fs');

var printContent = function(err, data) {
  if (err) {
    console.log("Something went wrong!");
    return;
  }

  console.log(data);
};

fs.readFile("input.txt", "utf-8", printContent);
```


###Reading files with promises

Imagine the case where, instead of requiring a callback, a function that reads a file returns a promise to you. A promise is
a self contained object where you can plug a function to be executed when the data from the file is returned. You could also
plug another function to handle an exception.

```
var printContent = function(data) {
  console.log(data);
}

var promise = promiseFromFile("input.txt");

promise.then(printContent);
```

You could also inline it and plug an exception handler function.

```
promiseFromFile("input.txt")
  .then(printContent)
  .catch(logError);
```

Notice that `printContent` is going to be executed later on, by the time the content of the file is returned.
Eventually, if a problem occur when opening or reading the file, the `printContent` would be skipped and the
exception would be handled by `logError`. It still reminds callbacks, but the advantages with this code structure
becomes more evident when we have more tasks that should be executed one after another. In the world of callbacks, 
each callback function is nested inside the previous one.

When using promises you can call `then()` with a handler function as parameter, which executes when the promise
realise its value. The handler function might also returns a promise, and in this case you could chain another `then()`,
and so on. This chainning approach makes the code more readable than nesting callbacks.

Usually you are going to be using a module or an API that return promises to you. But you also may be interested in make
you own asynch and non blocking functions return promises, instead of expecting callbacks as parameters. As an example,
let's wrap the callback-based `readFile` function to make it return promises instead.


###Returning promises instead of requiring callbacks

Based on the first code snippet, I'm going to wrap the original `readFile` into another function that returns a promise.
I'll create a deferred object using `defer()`, and then return the `promisse` out of it.

```
var filePromise = function(filename) {

  // create an empty deferred object
  var deferred = q.defer();

  fs.readFile(filename, "utf-8", function(err, data) {
    if (err) {
      console.log("Something went wrong!");
      return;
    }

    console.log(data);
  });

  // the actual promisse
  return deferred.promise;
}
```

It doesn't work yet. At this point, I need to fulfill the value of the promise with the 
returned content of the file, when it is returned. In other words, this is when the promise should be resolved. I also
need to reject the promise with an exception in case of an error is being returned.


```
var filePromise = function(filename) {

  var deferred = q.defer();

  fs.readFile(filename, "utf-8", function(err, data) {
    if (err) {
      // reject the promise when an error is returned
      return deferred.reject(new Error(err));
    }

    // resolving the promisse with the file content
    return deferred.resolve(data);
  });

  return deferred.promise;
}
```

Now I'm ready to try to read a file. I just need to get the promise and plug my handlers on it.

```
filePromise("input.txt")
  .then(function(data) {
    console.log("file content: " + data);
  })
  .catch(function(error) {
    console.log("Something went wrong!");
  });
```

### Callbacks vs Promise

Things would get more interesting, and more useful!, if you need to do other asynchronous tasks by the time you get your
first result back from the promisse. Using callbacks, you should have to nest the functions.

```
// function that reads a file
var readFile(filename, callback) { ... };

// function to post data to a service
var post = function(url, data, callback) { ... };

// function to save data to the database
var save = function(conn, data, callback) { ... };

readFile("input.txt", function(err, data) {
  if (err) {
    return console.log("Something went wrong!");
  }

  post("http://server.com", data, function(err, res) {
    if (err) {
      return console.log("Something went wrong");
    }

    save(conn, data, function(err) {
      if (err) {
        return console.log("Something went wrong");
      }
    });
  });
});
```

With promises, would be possible to chain handlers when they also return promises.

```
// reads a file, and returns a promise
var readFile(filename) { ... return deferred.promise; };

// post data, and returns a promise
var post(url, data) { ... return deferred.promise; };

// save in the db, and returns a promise
var save(conn, data) { ... return deferred.promise; };

readFile("input.txt")
  .then(function(data) {
    return post("http://server.com", data)
  })
  .then(function(data) {
    return save("database:mydb", data);
  })
  .catch(logError);
```

I hope it helps as a starting point to understand how promises work.







