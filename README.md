# Mixinable

[![travis](https://img.shields.io/travis/dmbch/mixinable.svg)](https://travis-ci.org/dmbch/mixinable)&nbsp;[![npm](https://img.shields.io/npm/v/mixinable.svg)](https://www.npmjs.com/package/mixinable)
<br/>

`mixinable` is a small functional utility library allowing you to use [mixins](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#mixinpatternjavascript) in your code. More specifically, it allows you to create mixin containers that apply mixin method application strategies to mixin method implementations.

Mixins are plain Objects (or hashes) that can easily be shared, modified and extended using standard language features such as [spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator#Spread_in_object_literals) or [`Object.assign()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign).

`mixinable` allows you to provide custom [`constructor`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/constructor) functions and supports asynchronous methods returning [`Promises`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promises). It is built in a functional way, allowing you to, for example, apply [`fn.bind()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind).


### Installation

Using [NPM](https://www.npmjs.com/get-npm):

```bash
npm install -S mixinable
```

Using [Yarn](https://yarnpkg.com/en/):

```bash
yarn add mixinable
```

To be able to use `mixinable`, you will have to make sure your environment supports [`Promise`](https://kangax.github.io/compat-table/es6/#test-Promise) and [`Object.assign()`/`Object.keys()`](https://kangax.github.io/compat-table/es6/#test-Object_static_methods): if you need to support IE11, you will have to [polyfill](https://polyfill.io/v2/docs/) those features.


### API

#### ```define(definition)```

The main export of `mixinable` is a `define()` function accepting a mixin container definition. This definition hash is made up of `strategy` functions prescribing how to handle different mixin methods you provide.

It returns a variadic `mixin()` function that accepts the mixin definitions you want to apply and returns a `create()` function. Mixin definitions are hashes containing mixin method implementations that are being applied using the aforementioned strategies.

And that `create()` function accepts whatever arguments the `constructor()`s of your mixin container and your mixins accept. 

##### Example

```javascript
import define from 'mixinable';

const mixin = define({
  // mixin strategy function
  bar (functions, arg) {
    return functions.pop()(arg);
  }
});

const create = mixin({
  // mixin implementation
  bar (arg) {
    console.log(arg);
  }
});

const foo = create();

foo.bar('yeah!');
// yeah!
```

Speaking of which: `mixinable` creates a separate hidden mixin instance for every mixin you provide. When used inside a mixin method, `this` refers to this hidden instance.

Mixin methods not included in your definition are only accessible from within the mixin instance itself, but not from the outside.

##### Example

```javascript
import define from 'mixinable';

const mixin = define({
  bar (functions, arg) {
    return functions.pop()(arg);
  }
});

const create = mixin({
  // mixin implementation
  bar (arg) {
    this.qux(arg);
  },
  // private mixin method
  qux (arg) {
    console.log(arg);
  }
});

const foo = create();

foo.bar('yeah!');
// yeah!

console.log(typeof foo.bar, typeof foo.qux);
// function undefined
```

Both mixin container and mixin definitions can contain custom constructors. These functions are being passed the `create()` function's arguments upon creation.

```javascript
import define from 'mixinable';

const mixin = define({
  // mixin container contructor
  constructor: function (arg) {
    console.log(arg);
  }
});

const create = mixin({
  // mixin contructor
  constructor: function (arg) {
    console.log(arg);
  }
});

create('yee-hah!');
// yee-hah!
// yee-hah!
```


#### ```define.parallel```

`parallel` is a helper implementating a mixin strategy that executes all defined implementations in parallel. This is probably most useful if asynchronous implementations are involved.

##### Example

```javascript
import define, { parallel } from 'mixinable';

const mixin = define({
  // mixin strategy function
  bar: parallel
});

const create = mixin(
  {
    bar (val, inc) {
      return Promise.resolve(val + inc);
    }
  },
  {
    bar (val, inc) {
      return val + inc;
    }
  }
);

const foo = create();

foo.bar(0, 1).then(res => console.log(res));
// [1, 1]
```


#### ```define.pipe```

`pipe` is a helper implementating a strategy that passes each implementation's output to the next, using the first argument as the initial value. All other arguments are being passed to all implementations as-is.

##### Example

```javascript
import define, { pipe } from 'mixinable';

const mixin = define({
  // mixin strategy function
  bar: pipe
});

const create = mixin(
  {
    bar (val, inc) {
      return Promise.resolve(val + inc);
    }
  },
  {
    bar (val, inc) {
      return (val + inc);
    }
  }
);

const foo = create();

foo.bar(0, 1).then(res => console.log(res));
// 2
```


### Contributing

If you want to contribute to this project, create a fork of its repository using the GitHub UI. Check out your new fork to your computer:

```bash
mkdir mixinable && cd $_
git clone git@github.com:user/mixinable.git
```

Afterwards, you can `yarn install` the required dev dependencies and start hacking away. When you are finished, please do go ahead and create a pull request.

`mixinable` itself is almost entirely written in ECMAScript 5 and adheres to semistandard code style. Please make sure your contribution does, too.
