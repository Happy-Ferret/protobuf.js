<h1><p align="center"><img alt="protobuf.js" src="https://github.com/dcodeIO/protobuf.js/raw/master/pbjs.png" width="120" height="104" /></p></h1>
<p align="center"><a href="https://travis-ci.org/dcodeIO/protobuf.js"><img src="https://travis-ci.org/dcodeIO/protobuf.js.svg?branch=master" alt=""></a> <a href="https://coveralls.io/github/dcodeIO/protobuf.js"><img src="https://coveralls.io/repos/github/dcodeIO/protobuf.js/badge.svg?branch=master" alt=""></a> <a href="https://npmjs.org/package/protobufjs"><img src="https://img.shields.io/npm/v/protobufjs.svg" alt=""></a> <a href="https://npmjs.org/package/protobufjs"><img src="https://img.shields.io/npm/dm/protobufjs.svg" alt=""></a> <a href="https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=dcode%40dcode.io&item_name=Open%20Source%20Software%20Donation&item_number=dcodeIO%2Fprotobuf.js"><img alt="donate ❤" src="https://img.shields.io/badge/donate-❤-ff2244.svg"></a></p>

**Protocol Buffers** are a language-neutral, platform-neutral, extensible way of serializing structured data for use in communications protocols, data storage, and more, originally designed at Google ([see](https://developers.google.com/protocol-buffers/)).

**protobuf.js** is a pure JavaScript implementation with TypeScript support for node and the browser. It's super easy to use, blazingly fast and works out of the box on .proto files!

**Recommended read:** [Changes in protobuf.js 6.0](https://github.com/dcodeIO/protobuf.js/wiki/Changes-in-protobuf.js-6.0)

Features
--------
* Optimized [for performance](#performance)
* Excellent [browser support](#compatibility)
* Managed [TypeScript definitions](#usage-with-typescript)
* Elaborate [API documentation](#documentation)
* Convenient [CLI utilities](#command-line)
* Seamless [browserify integration](#browserify-integration)

Contents
--------

* [Usage](#usage)<br />
  How to include protobuf.js in your project.

* [Examples](#examples)<br />
  A few examples to get you started.

* [Documentation](#documentation)<br />
  A list of available documentation resources.

* [Command line](#command-line)<br />
  How to use the command line utility.

* [Performance](#performance)<br />
  A few internals and a benchmark on performance.

* [Compatibility](#compatibility)<br />
  Notes on compatibility regarding browsers and optional libraries.

* [Building](#building)<br />
  How to build the library and its components yourself.

Usage
---------------

### node.js

```
$> npm install protobufjs [--save --save-prefix=~]
```

```js
var protobuf = require("protobufjs");
```

### Browsers

Development:
```
<script src="//cdn.rawgit.com/dcodeIO/protobuf.js/6.X.X/dist/protobuf.js"></script>
```

Production:
```
<script src="//cdn.rawgit.com/dcodeIO/protobuf.js/6.X.X/dist/protobuf.min.js"></script>
```

**NOTE:** Remember to replace the version tag with the exact [release](https://github.com/dcodeIO/protobuf.js/tags) your project depends upon.

Or [download](https://github.com/dcodeIO/protobuf.js/tree/master/dist) the library.

The `protobuf` namespace will always be available globally / also supports AMD loaders.

Additionally, the library is compiled in different versions. Which one to use depends on whether size is a factor and your use case:

| Build   | Downloads                    | How to require                  | Description
|---------|------------------------------|---------------------------------|-------------
| full    | [dist][dist-full]            | `require("protobufjs")`         | All features. Works with everything.
| light   | [dist/light][dist-light]     | `require("protobufjs/light")`   | All features except tokenizer, parser and bundled common types. Works with reflection, JSON definitions and static code.
| minimal | [dist/minimal][dist-minimal] | `require("protobufjs/minimal")` | Just enough to run statically generated code. Works with static code only.

Examples
--------

### Using .proto files

```protobuf
// awesome.proto
package awesomepackage;
syntax = "proto3";

message AwesomeMessage {
    string awesome_field = 1; // becomes awesomeField
}
```

```js
protobuf.load("awesome.proto", function(err, root) {
    if (err) throw err;

    // Obtain a message type
    var AwesomeMessage = root.lookup("awesomepackage.AwesomeMessage");

    // Create a new message
    var message = AwesomeMessage.create({ awesomeField: "AwesomeString" });

    // Encode a message
    var buffer = AwesomeMessage.encode(message).finish();
    // ... do something with buffer

    // Or, encode a plain object
    var buffer = AwesomeMessage.encode({ awesomeField: "AwesomeString" }).finish();
    // ... do something with buffer

    // Decode a buffer
    var message = AwesomeMessage.decode(buffer);
    // ... do something with message

    // If your application uses length-delimited buffers, there is also encodeDelimited and decodeDelimited.
});
```

You can also use promises by omitting the callback:

```js
protobuf.load("awesome.proto")
    .then(function(root) {
       ...
    });
```

### Using reflection only

```js
...
var Root  = protobuf.Root,
    Type  = protobuf.Type,
    Field = protobuf.Field;

var AwesomeMessage = new Type("AwesomeMessage").add(new Field("awesomeField", 1, "string"));

var root = new Root().define("awesomepackage").add(AwesomeMessage);

// Continue at "Create a new message" above
...
```

### Using custom classes

```js
...

function AwesomeMessage(properties) {
    protobuf.Message.call(this, properties);
}
protobuf.Class.create(root.lookup("awesomepackage.AwesomeMessage") /* or use reflection */, AwesomeMessage);

var message = new AwesomeMessage({ awesomeField: "AwesomeString" });

// Continue at "Encode a message" above
```

Custom classes are automatically populated with static `encode`, `encodeDelimited`, `decode`, `decodeDelimited` and `verify` methods and reference their reflected type via the `$type` property. Note that there are no methods (just `$type`) on instances by default as method names might conflict with field names.

### Using the Reader/Writer interface directly

While only useful for the adventurous cherishing an aversion to [generated static code](https://github.com/dcodeIO/protobuf.js#command-line), it's also possible to use the Reader/Writer interface directly depending just on the [minimal library][dist-minimal] ([basic example](https://github.com/dcodeIO/protobuf.js/blob/master/examples/reader-writer.js)).

Easy ways to obtain example code snippets are either setting `protobuf.util.codegen.verbose = true` while watching the magic as it happens, or simply inspecting generated static code.

### Using services

```protobuf
// greeter.proto
syntax = "proto3";

service Greeter {
    rpc SayHello (HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
    string name = 1;
}

message HelloReply {
    string message = 1;
}
```

```js
...
var Greeter = root.lookup("Greeter");
var greeter = Greeter.create(rpcImpl, false, false); // rpcImpl (see below), requestDelimited?, responseDelimited?

greeter.sayHello({ name: 'you' }, function(err, response) {
    console.log('Greeting:', response.message);
});
```

To make this work, all you have to do is provide an `rpcImpl`, which is an asynchronous function that takes the reflected service method, the binary HelloRequest and a node-style callback as its parameters. For example:

```js
function rpcImpl(method, requestData, callback) {
    // perform the request using an HTTP request or a WebSocket for example
    var responseData = ...;
    // and call the callback with the binary response afterwards:
    callback(null, responseData);
}
```

There is also an [example for streaming RPC](https://github.com/dcodeIO/protobuf.js/blob/master/examples/streaming-rpc.js).

### Usage with TypeScript

```ts
import * as protobuf from "protobufjs";
import * as Long from "long"; // optional
...
```

See also: [Generating your own TypeScript definitions](https://github.com/dcodeIO/protobuf.js#generating-typescript-definitions-from-static-modules)

Additional configuration might be necessary when not utilizing node, i.e. reference [protobuf.js.d.ts](https://github.com/dcodeIO/protobuf.js/blob/master/index.d.ts) and [long.js.d.ts](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/types-2.0/long/index.d.ts).

Documentation
-------------

* [Google's Developer Guide](https://developers.google.com/protocol-buffers/docs/overview)
* [protobuf.js API Documentation](http://dcode.io/protobuf.js/) and [CHANGELOG](https://github.com/dcodeIO/protobuf.js/blob/master/CHANGELOG.md)
* [Frequently asked questions](https://github.com/dcodeIO/protobuf.js/wiki) on our wiki
* [More questions and answers](http://stackoverflow.com/questions/tagged/protobuf.js) on StackOverflow

Command line
------------

The `pbjs` command line utility can be used to bundle and translate between .proto and .json files.

```
Consolidates imports and converts between file formats.

  -t, --target    Specifies the target format. Also accepts a path to require a custom target.

                  json          JSON representation
                  json-module   JSON representation as a module
                  proto2        Protocol Buffers, Version 2
                  proto3        Protocol Buffers, Version 3
                  static        Static code without reflection
                  static-module Static code without reflection as a module

  -p, --path      Adds a directory to the include path.

  -o, --out       Saves to a file instead of writing to stdout.

  Module targets only:

  -w, --wrap      Specifies the wrapper to use. Also accepts a path to require a custom wrapper.

                  default   Default wrapper supporting both CommonJS and AMD
                  commonjs  CommonJS wrapper
                  amd       AMD wrapper
                  es6       ES6 wrapper

  -r, --root      Specifies an alternative protobuf.roots name.

  -l, --lint      Linter configuration. Defaults to protobuf.js-compatible rules:

                  eslint-disable block-scoped-var, no-redeclare, no-control-regex

  Proto sources only:

  --keep-case     Keeps field casing instead of converting to camel case.

  Static targets only:

  --no-create     Does not generate create functions used for reflection compatibility.
  --no-encode     Does not generate encode functions.
  --no-decode     Does not generate decode functions.
  --no-verify     Does not generate verify functions.
  --no-convert    Does not generate convert functions like from/toObject
  --no-delimited  Does not generate delimited encode/decode functions.
  --no-beautify   Does not beautify generated code.
  --no-comments   Does not output any JSDoc comments.

usage: pbjs [options] file1.proto file2.json ...  (or)  other | pbjs [options] -
```

For production environments it is recommended to bundle all your .proto files to a single .json file, which reduces the number of network requests and parser invocations required:

```
$> pbjs -t json file1.proto file2.proto > bundle.json
```

Now, either include this file in your final bundle:

```js
var root = protobuf.Root.fromJSON(require("./bundle.json"));
```

or load it the usual way:

```js
protobuf.load("bundle.json", function(err, root) {
    ...
});
```

As you might have noticed, `pbjs` is also capable of generating static code. For example

```
$> pbjs -t static-module -w commonjs -o compiled.js file1.proto file2.proto
```

will generate static code for definitions within `file1.proto` and `file2.proto` to a CommonJS module `compiled.js`, which then can be used with just the [minimal library][dist-minimal].

**ProTip!** Documenting your .proto files with `/** ... */`-blocks or (trailing) `/// ...` lines translates to generated static code.

### Generating TypeScript definitions from static modules

Likewise, the `pbts` command line utility can be used to generate TypeScript definitions from `pbjs`-generated static modules.

```
Generates TypeScript definitions from annotated JavaScript files.

  -n, --name      Wraps everything in a module of the specified name.

  -o, --out       Saves to a file instead of writing to stdout.

  -m, --main      Whether building the main library without any imports.

  -g, --global    Name of the global object in browser environments, if any.

  --no-comments   Does not output any JSDoc comments.

usage: pbts [options] file1.js file2.js ...  (or)  other | pbts [options] -
```

Picking up on the example above, the following not just generates static code to a CommonJS module `compiled.js` but also its respective TypeScript definitions to `compiled.d.ts`:

```
$> pbjs -t static-module -w commonjs -o compiled.js file1.proto file2.proto
$> pbts -o compiled.d.ts compiled.js
```

Additionally, TypeScript definitions of static modules are compatible with reflection, as long as the following conditions are met:

1. Instead of using `new SomeMessage(...)`, always use `SomeMessage.create(...)` because reflection objects do not provide a constructor.
2. Types, services and enums must start with an uppercase letter to become available as properties of the reflected types as well (i.e. to be able to use `MyMessage.MyEnum` instead of `root.lookup("MyMessage.MyEnum")`).

For example, the following generates a `bundle.json` and a `bundle.d.ts`, but no static code:

```
$> pbjs -t json-module -w commonjs -o bundle.json file1.proto file2.proto
$> pbjs -t static-module file1.proto file2.proto | pbts -o bundle.d.ts -
```

### On reflection vs. static code

While using .proto files requires the [full library][dist-full] (about 18.5kb gzipped) or JSON at least the [light library][dist-light] (about 15.5kb gzipped), pretty much all code but the relatively short descriptors is shared and all features including reflection and the parser are available.

Static code, on the other hand, requires just the [minimal library][dist-minimal] (about 5.5kb gzipped), but generates additional, albeit editable, source code without any reflection features.

There is no significant difference performance-wise as the code generated statically is pretty much the same as generated at runtime and both are largely interchangeable as seen in the previous section.

### Using pbjs and pbts programmatically

Both utilities can be used programmatically by providing command line arguments and a callback to their respective `main` functions:

```js
var pbjs = require("protobufjs/cli/pbjs");

pbjs.main([ "--target", "json-module", "path/to/myproto.proto" ], function(err, output) {
    if (err)
        throw err;
    // do something with output
});
```

Performance
-----------
The package includes a benchmark that tries to compare performance to native JSON as far as this is possible. On an i7-2600K running node 6.9.1 it yields:

```
benchmarking encoding performance ...

Type.encode to buffer x 547,361 ops/sec ±0.27% (94 runs sampled)
JSON.stringify to string x 310,848 ops/sec ±0.73% (92 runs sampled)
JSON.stringify to buffer x 173,608 ops/sec ±1.51% (86 runs sampled)

      Type.encode to buffer was fastest
   JSON.stringify to string was 43.5% slower
   JSON.stringify to buffer was 68.7% slower

benchmarking decoding performance ...

Type.decode from buffer x 1,294,378 ops/sec ±0.86% (90 runs sampled)
JSON.parse from string x 291,944 ops/sec ±0.72% (92 runs sampled)
JSON.parse from buffer x 256,325 ops/sec ±1.50% (90 runs sampled)

    Type.decode from buffer was fastest
     JSON.parse from string was 77.4% slower
     JSON.parse from buffer was 80.3% slower

benchmarking combined performance ...

Type to/from buffer x 254,126 ops/sec ±1.13% (91 runs sampled)
JSON to/from string x 122,896 ops/sec ±1.29% (90 runs sampled)
JSON to/from buffer x 88,005 ops/sec ±0.87% (89 runs sampled)

        Type to/from buffer was fastest
        JSON to/from string was 51.7% slower
        JSON to/from buffer was 65.3% slower

benchmarking verifying performance ...

Type.verify x 6,246,765 ops/sec ±2.00% (87 runs sampled)

benchmarking message from object performance ...

Type.fromObject x 2,892,973 ops/sec ±0.70% (92 runs sampled)

benchmarking message to object performance ...

Type.toObject x 3,601,738 ops/sec ±0.72% (93 runs sampled)
```

Note that JSON is a native binding nowadays and as such is about as fast as it possibly can get. So, how can protobuf.js be faster?

* The benchmark is [somewhat flawed](https://github.com/dcodeIO/protobuf.js/blob/master/bench/index.js).
* Reader and writer interfaces configure themselves according to the environment to eliminate redundant conditionals.
* Node-specific reader and writer subclasses benefit from node's buffer binding.
* Reflection has built-in code generation that builds type-specific encoders, decoders and verifiers at runtime.
* Encoders and decoders do not implicitly call `verify` on messages to avoid unnecessary overhead where messages are already known to be valid. It's up to the user to call `verify` where necessary.
* Quite a bit of V8-specific profiling is accountable for everything else.

You can also run [the benchmark](https://github.com/dcodeIO/protobuf.js/blob/master/bench/index.js) ...

```
$> npm run bench
```

and [the profiler](https://github.com/dcodeIO/protobuf.js/blob/master/bench/prof.js) yourself (the latter requires a recent version of node):

```
$> npm run prof <encode|decode|encode-browser|decode-browser> [iterations=10000000]
```

Note that as of this writing, the benchmark suite performs significantly slower on node 7.2.0 compared to 6.9.1 because moths.

Compatibility
-------------

[![Sauce Test Status](https://saucelabs.com/browser-matrix/protobuf.svg)](https://saucelabs.com/u/protobuf)

* Because the internals of this package do not rely on `google/protobuf/descriptor.proto`, options are parsed and presented literally.
* If typed arrays are not supported by the environment, plain arrays will be used instead.
* Support for pre-ES5 environments (except IE8) can be achieved by [using a polyfill](https://github.com/dcodeIO/protobuf.js/blob/master/scripts/polyfill.js).
* Support for [Content Security Policy](https://w3c.github.io/webappsec-csp/)-restricted environments (like Chrome extensions without [unsafe-eval](https://developer.chrome.com/extensions/contentSecurityPolicy#relaxing-eval)) can be achieved by generating and using static code instead.
* If you need a proper way to work with 64 bit values (uint64, int64 etc.), you can install [long.js](https://github.com/dcodeIO/long.js) alongside this library. All 64 bit numbers will then be returned as a `Long` instance instead of a possibly unsafe JavaScript number ([see](https://github.com/dcodeIO/long.js)).

Building
--------

To build the library or its components yourself, clone it from GitHub and install the development dependencies:

```
$> git clone https://github.com/dcodeIO/protobuf.js.git
$> cd protobuf.js
$> npm install
```

Building the respective development and production versions with their respective source maps to `dist/`:

```
$> npm run build
```

Building the documentation to `docs/`:

```
$> npm run docs
```

Building the TypeScript definition to `index.d.ts`:

```
$> npm run types
```

### Browserify integration

By default, protobuf.js integrates into your browserify build-process without requiring any optional modules. Hence:

* If you need int64 support, explicitly require the `long` module somewhere in your project. It will be excluded otherwise.
* If you have any special requirements, there is [the bundler](https://github.com/dcodeIO/protobuf.js/blob/master/scripts/bundle.js) as a reference.

**License:** [BSD 3-Clause License](https://opensource.org/licenses/BSD-3-Clause)

[dist-full]: https://github.com/dcodeIO/protobuf.js/tree/master/dist
[dist-light]: https://github.com/dcodeIO/protobuf.js/tree/master/dist/light
[dist-minimal]: https://github.com/dcodeIO/protobuf.js/tree/master/dist/minimal
