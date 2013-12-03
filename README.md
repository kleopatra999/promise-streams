# through-promise

through-promise is like through2, but with promises

# example

```js
var pthrough = require('../');
var B = require('bluebird');
var fs = require('fs');
var split = require('split');
var path = require('path');
var through = require('through2');

fs.createReadStream(path.join(__dirname, 'test.txt'), 'utf8')
    .pipe(split())
    .pipe(pthrough({limit: 3}, function(line) {
        var delayed = B.delay(100).then(function() {
            return line ? parseFloat(line) : null;
        });
        return this.push(delayed);
    }))
    .map(function(el) { return el * 2; })
    .reduce(function(acc, el) { return acc + el; })
    .done(function(sum) {
        console.log("Result", sum);
    }, function(e) {
        console.error(e.stack);
    });
```

# api

#### pthrough

`([opts:Options,] fn:(data[, enc]) => Promise)) => PromiseStream`

Create a through-promise stream. Pass it a function that takes data and 
encoding and uses `this.push` to push values or promises. This function should 
return a promise that indicates when the object/chunk are fully processed.

Returns a PromiseStream. 

#### PromiseStream.push

Like `this.push` in [through2](//github.com/rvagg/through2), but takes promise
arguments. It returns a promise that resolves when the pushed promise resolves,
to make it possible to use `return this.push(data)`

#### PromiseStream.map

`([opts:Options,] fn: (data[, enc]) => Promise) => PromiseStream`

Create a new PromiseStream and pipes this promise stream to it. The function 
should return a promise for the next object that will be pushed to the stream.

#### PromiseStream.reduce

`([opts:Options,] fn: (acc, data[, enc]) => Promise) => Promise`

Reduces the objects in this promise stream. The function takes the resolved 
current accumulator and data object and should return the next accumulator
or a promise for the next accumulator.

