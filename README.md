# io.js v2.0 will be released soon!!!!!!!

I would like to show io.js new features.

# URL performance is blazing fast.

## Before

```
$ git checkout v1.x && make -j4 && bleed benchmark/misc/url.js
misc/url.js parse(): 16510
misc/url.js format(): 18309
misc/url.js resolve("../foo/bar?baz=boom"): 8929.9
misc/url.js resolve("foo/bar"): 9919.5
misc/url.js resolve("http://nodejs.org"): 8177.6
misc/url.js resolve("./foo/bar?baz"): 8984.4

$ bleed benchmark/common.js url
url/url-parse.js
url/url-parse.js type=one n=250000: 82666.55318
url/url-parse.js type=two n=250000: 84669.54973
url/url-parse.js type=three n=250000: 80308.64406
url/url-parse.js type=four n=250000: 262017.65403
url/url-parse.js type=five n=250000: 203834.95043
url/url-parse.js type=six n=250000: 68911.18834

url/url-resolve.js
url/url-resolve.js type=one n=100000: 42331.42854
```

## After

```
$ bleed benchmark/misc/url.js
misc/url.js parse(): 1.6529e+5
misc/url.js format(): 1.9114e+5
misc/url.js resolve("../foo/bar?baz=boom"): 24302
misc/url.js resolve("foo/bar"): 35696
misc/url.js resolve("http://nodejs.org"): 68003
misc/url.js resolve("./foo/bar?baz"): 29983

$ bleed benchmark/common.js url
url/url-parse.js
url/url-parse.js type=one n=250000: 1244025.74054
url/url-parse.js type=two n=250000: 2071492.21773
url/url-parse.js type=three n=250000: 1113916.06067
url/url-parse.js type=four n=250000: 3416688.31936
url/url-parse.js type=five n=250000: 1620766.16897
url/url-parse.js type=six n=250000: 1659564.59318

url/url-resolve.js
url/url-resolve.js type=one n=100000: 184353.89591
```

That's 10-24x faster depending on url.

Details is here. 
https://github.com/iojs/io.js/pull/933

## Note: Url parse does not return JSON object

url module breaks backward compatible. you should call `toJSON` method to get the object.

### Before

```javascript
var url = require('url');
var parsedUrl = url.parse('http://user:pass@host.com:8080/p/a/t/h?query=string#hash');
console.log(parsedUrl); // you can get parsed object.
```

### After

```javascript
var url = require('url');
var parsedUrl = url.parse('http://user:pass@host.com:8080/p/a/t/h?query=string#hash');
console.log(parsedUrl.toJSON()); // you should call toJSON method to get parsed object.
```

# ECMAScript 2015 - feature

## class

class is available. 

no need to specify any `harmony-` arguments.

note: we should put 'use strict' on top.

```javascript
// strict mode needed
'use strict';

class Animal {
  constructor(name) {
    this.name = name
  }
  
  say() {
    // unimplemented
  }
}

class Cat extends Animal {
  say() {
    console.log(`${this.name} < meow`);
  }
}

var cat = new Cat('Mike');
cat.say(); // Mike < meow
```


## enhanced object literals

Now, we can use new `object literals`. 
no need to specify any `harmony-` arguments.

```javascript
'use strict';
// class
class Person {
  constructor(name, age) {
    this.name = name
    this.age = age
  }
 getInfo() {
    let name = this.name;
    let age = this.age;
    let nextAge = this.age + 1;
    // enhanced object literal
    return {
      name,
      age,
      nextAge
    };
  }
}

var bob = new Person('bob', 15);

console.log(bob.getInfo()); //{ name: 'bob', age: 15, nextAge: 16 }
```

## Rest parameters

Rest parameters (`function(...args) {}`) are implemented in staging behind the `--harmony-rest-parameters` flag.

```javascript
// Rest parameters
function max(...args) {
  // rest parameter is not Array-like object, that is just array.
  console.log(Array.isArray(args))  // true
  console.log(args.length)          // 6

  var max = args.reduce(function(max, n) { 
    return n > max ? n : max;
  });
  return max;
}

var maxNum = max(5, 15, 10, 1, 4, 5);
console.log(maxNum); // 15
```

```
$ iojs --harmony-rest-parameters es6/rest_params/rest.js
```

## Computed property names

Computed property names (`{['foo'+'bar']:'bam'}`) are implemented in staging behind the `--harmony-computed-property-names` flag

```javascript
var i = 0;
var a = {
  ["foo" + ++i]: i,
  ["foo" + ++i]: i,
  ["foo" + ++i]: i
};

console.log(a.foo1); // 1
console.log(a.foo2); // 2
console.log(a.foo3); // 3

var param = 'size';
var config = {
  [param]: 12,
  ["mobile" + param.charAt(0).toUpperCase() + param.slice(1)]: 4
};

console.log(config);
```

```
$ iojs --harmony-computed-property-names es6/computed_property/computedProps.js
```

# Strong Mode

v8 has new features for strengthening JavaScript. if you put 'use strong' on top of your code and put `--strong_mode` argument, then you can use strong mode. 

if you would like to know more, please check this article.
https://developers.google.com/v8/experiments

note: this is an experimental feature. this feature is not stable.

## var => let, const

```javascript
'use strong';

var a = 'hoge';
```

```
$ iojs --strong_mode strong_mode/vars.js

/Users/yosuke/iojs_v2_features/strong_mode/vars.js:3
var a = 'hoge';
^^^
SyntaxError: Please don't use 'var' in strong mode, use 'let' or 'const' instead
    at exports.runInThisContext (vm.js:53:16)
    at Module._compile (module.js:411:25)
    at Object.Module._extensions..js (module.js:446:10)
    at Module.load (module.js:353:32)
    at Function.Module._load (module.js:308:12)
    at Function.Module.runMain (module.js:469:10)
    at startup (node.js:124:18)
    at node.js:882:3
```

## arguments => ...args 

```javascript
'use strong';

function some() {
  let args = Array.prototype.slice.call(arguments);
}

some();
```

```
$ iojs --strong_mode strong_mode/arguments.js
/Users/yosuke/iojs_v2_features/strong_mode/arguments.js:4
  let args = Array.prototype.slice.call(arguments);
                                        ^^^^^^^^^
SyntaxError: Please don't use 'arguments' in strong mode, use '...args' instead
    at exports.runInThisContext (vm.js:53:16)
    at Module._compile (module.js:411:25)
    at Object.Module._extensions..js (module.js:446:10)
    at Module.load (module.js:353:32)
    at Function.Module._load (module.js:308:12)
    at Function.Module.runMain (module.js:469:10)
    at startup (node.js:124:18)
    at node.js:882:3
```

## eqeq => eqeqeq

```javascript
'use strong';

if ('a' == 'a') {
}
```

```
$ iojs --strong_mode strong_mode/eqeq.js

/Users/yosuke/iojs_v2_features/strong_mode/eqeq.js:3
if ('a' == 'a') {
        ^^
SyntaxError: Please don't use '==' or '!=' in strong mode, use '===' or '!==' instead
    at exports.runInThisContext (vm.js:53:16)
    at Module._compile (module.js:411:25)
    at Object.Module._extensions..js (module.js:446:10)
    at Module.load (module.js:353:32)
    at Function.Module._load (module.js:308:12)
    at Function.Module.runMain (module.js:469:10)
    at startup (node.js:124:18)
    at node.js:882:3
```

## don't use delete, use map or set instead.

```javascript
'use strong';
let obj = { key: 'value'};
delete obj.key;
console.log(obj);
```


```
$ iojs --strong_mode strong_mode/delete.js
/Users/yosuke/iojs_v2_features/strong_mode/delete.js:5
delete obj.key;
           ^^^
SyntaxError: Please don't use 'delete' in strong mode, use maps or sets instead
    at exports.runInThisContext (vm.js:53:16)
    at Module._compile (module.js:411:25)
    at Object.Module._extensions..js (module.js:446:10)
    at Module.load (module.js:353:32)
    at Function.Module._load (module.js:308:12)
    at Function.Module.runMain (module.js:469:10)
    at startup (node.js:124:18)
    at node.js:882:3
```


## for-in => for-of

```javascript
'use strong';

for (let k in [1, 2, 3]) {
  console.log(k);
}

```

```
$ iojs --strong_mode strong_mode/for.js
/Users/yosuke/iojs_v2_features/strong_mode/for.js:3
for (let k in [1, 2, 3]) {
           ^^
SyntaxError: Please don't use 'for'-'in' loops in strong mode, use 'for'-'of' instead
    at exports.runInThisContext (vm.js:53:16)
    at Module._compile (module.js:411:25)
    at Object.Module._extensions..js (module.js:446:10)
    at Module.load (module.js:353:32)
    at Function.Module._load (module.js:308:12)
    at Function.Module.runMain (module.js:469:10)
    at startup (node.js:124:18)
    at node.js:882:3
```

## empty 

```javascript
'use strong';

if (true);
console.log('hello');
```


```
$ iojs --strong_mode strong_mode/empty.js
/Users/yosuke/iojs_v2_features/strong_mode/empty.js:3
if (true);
         ^
SyntaxError: Please don't use empty sub-statements in strong mode, make them explicit with '{}' instead
    at exports.runInThisContext (vm.js:53:16)
    at Module._compile (module.js:411:25)
    at Object.Module._extensions..js (module.js:446:10)
    at Module.load (module.js:353:32)
    at Function.Module._load (module.js:308:12)
    at Function.Module.runMain (module.js:469:10)
    at startup (node.js:124:18)
    at node.js:882:3
```

# Want to know more??

please check the following issue.

https://github.com/iojs/io.js/pull/1532

https://github.com/iojs/io.js/wiki/Breaking-Changes
