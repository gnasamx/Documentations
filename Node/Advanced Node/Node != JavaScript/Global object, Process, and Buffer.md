# Global object, Process, and Buffer

## Global object

The one and only global object in Node is called `global` . When we declare top level variable in Node like the `answer` variable in _util.js_ file, this `answer` variable is local to this _util.js_ file. We can't access it from other files, event after requiring _util.js_ file.

util.js

``` js
var answer = 45;
```

index.js

``` js
require('./util.js');
console.log(answer);
```

output

``` bash
Ganeshs-MacBook-Air:Desktop ganesh$ node index.js
/Users/ganesh/Desktop/index.js:3
console.log(answer);
            ^

ReferenceError: answer is not defined
    at Object.<anonymous> (/Users/ganesh/Desktop/index.js:3:13)
    at Module._compile (internal/modules/cjs/loader.js:774:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:785:10)
    at Module.load (internal/modules/cjs/loader.js:641:32)
    at Function.Module._load (internal/modules/cjs/loader.js:556:12)
    at Function.Module.runMain (internal/modules/cjs/loader.js:837:10)
    at internal/main/run_main_module.js:17:11
```

However when we define `answer` on global object, the `index.js` file which requires `util.js` module, can now access that globally declared variable.

util.js

``` js
global.answer = 45;
```

index.js

``` js
require('./util.js');
console.log(answer);
```

output

``` bash
Ganeshs-MacBook-Air:Desktop ganesh$ node index.js 
45
```

> **Note**: We should avoid defining things on the global object at any cost.

## Properties of global object

Two of the important things that are available on global object in a normal script are the `process` object and `buffer` object.

## Process

The Node's `process` object provides bridge between the Node application and it's running environment.

Some examples of `process` properties.

1. `process.versions` : We can use this commands to read versions of current Node and and it's dependencies.

``` bash

> process.versions

{
  node: '12.4.0',
  v8: '7.4.288.27-node.18',
  uv: '1.29.1',
  zlib: '1.2.11',
  brotli: '1.0.7',
  ares: '1.15.0',
  modules: '72',
  nghttp2: '1.38.0',
  napi: '4',
  llhttp: '1.1.3',
  http_parser: '2.8.0',
  openssl: '1.1.1b',
  cldr: '35.1',
  icu: '64.2',
  tz: '2019a',
  unicode: '12.1'
}
```

 We can use this version numbers to determine is we should run custom code

``` js
if (process.versions.v8 < '4') {
    // do something for older version of v8s
}
```

2. `process.env` : The `process.env` property returns an object containing the user environment

``` bash

> process.env

{
  TERM_PROGRAM: 'Apple_Terminal',
  SHELL: '/bin/bash',
  TERM: 'xterm-256color',
  TMPDIR: '/var/folders/5t/h_ffbhxj7gq0lwypfy1gz8_w0000gn/T/',
  Apple_PubSub_Socket_Render: '/private/tmp/com.apple.launchd.gBn7HUbrYx/Render',
  TERM_PROGRAM_VERSION: '421.2',
  OLDPWD: '/Users/ganesh',
  TERM_SESSION_ID: '5381EB3B-0D38-4A28-B0BE-F7257DB7FCA5',
  USER: 'ganesh',
  SSH_AUTH_SOCK: '/private/tmp/com.apple.launchd.TYcsxylaX8/Listeners',
  PATH: '/usr/local/sbin:/usr/local/sbin:/Users/ganesh/.composer/vendor/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin',
  PWD: '/Users/ganesh/Desktop',
  XPC_FLAGS: '0x0',
  XPC_SERVICE_NAME: '0',
  SHLVL: '1',
  HOME: '/Users/ganesh',
  LOGNAME: 'ganesh',
  LC_CTYPE: 'UTF-8',
  _: '/usr/local/bin/node',
  __CF_USER_TEXT_ENCODING: '0x1F5:0:2'
}
```

You should not read from `process.env` directly. We usually used configuration variables like passwords or API access keys from the environment. Also which PORT we listen to, which database uris to connect to. You should put all of these behind the configuration or setting module and always read from that module, not `process.env` directly. This way if decide to change the way you read these configurations, you modify only one module.

util.js

``` js
export const config = {
    port: process.env.PORT || 8000
};
```

index.js

``` js
const {
    config
} = require('./util.js');

const port = config.port;
```

## Process is an event emitter

The process object is an instance of `EventEmitter` . This means we can emit events from process and we can listen to certain events on the process.

On exit is emitted when Node's event loop has nothing else to do, or when the manual call to `process.exit` has been executed.

``` js
process.on('exit', (code) => {
    // do one final synchronous operation
    // before the node process terminates
})
```

 We can do only synchronous operations inside this event handler, we can't use event loop here.

 One other usually misused event on the process is `uncaughtException` event. which is emitted whenever a JavaScript exception is not handles and it bubbles all the way to the event loop. In that case and by default Node will print the stack the trace and exit. 
 
 

``` js
process.on('uncaughtException', (err) => {
    // something went undefined
    // do any cleanup and exit anyway
})
```

 Unlike the exit event though, if we register the handler on the `uncaughtException` , Node will not exit, and this is bad and unpredictable in some cases, so you should avoid doing this interrupt and let the process exit.

 Here is an example to show difference between `exit` and `uncaughtException` 

process.js
 ```js
process.on('exit', (code) => {
  console.log( `About to exist with a code: ${code}` ); 
}); 

process.on('uncaughtException', (err) => {
  console.error(err); // don't do just that
}); 

// keep the event loop busy for some time
process.stdin.resume(); 

// trigger a TypeError exception
console.dog(); 
 ``` 
If we only had the process event handler the script will report the exit code and actually exit. But since we have registered a handler for `uncaughtException` and did not manually exit the process, Node will keep running, often in an unpredictable state.

output

``` bash
Ganeshs-MacBook-Air:Desktop ganesh$ node process.js 
TypeError: console.dog is not a function
    at Object.<anonymous> (/Users/ganesh/Desktop/process.js:18:9)
    at Module._compile (internal/modules/cjs/loader.js:774:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:785:10)
    at Module.load (internal/modules/cjs/loader.js:641:32)
    at Function.Module._load (internal/modules/cjs/loader.js:556:12)
    at Function.Module.runMain (internal/modules/cjs/loader.js:837:10)
    at internal/main/run_main_module.js:17:11
```

The safest option here is to let process exit anyway.

process.js
 ```js
process.on('exit', (code) => {
  console.log( `About to exist with a code: ${code}` ); 
}); 

process.on('uncaughtException', (err) => {
  console.error(err); // don't do just that

  // Force exit the process too
  process.exit(1); 
}); 

// keep the event loop busy for some time
process.stdin.resume(); 

// trigger a TypeError exception
console.dog(); 
 ``` 

``` bash
Ganeshs-MacBook-Air:Desktop ganesh$ node process.js 
TypeError: console.dog is not a function
    at Object.<anonymous> (/Users/ganesh/Desktop/process.js:18:9)
    at Module._compile (internal/modules/cjs/loader.js:774:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:785:10)
    at Module.load (internal/modules/cjs/loader.js:641:32)
    at Function.Module._load (internal/modules/cjs/loader.js:556:12)
    at Function.Module.runMain (internal/modules/cjs/loader.js:837:10)
    at internal/main/run_main_module.js:17:11
About to exist with a code: 1
```

So here when process exits, exit event handler will invoked.

## Buffer

The `Buffer` class also available on `global` object, is used heavily in Node to work with binary streams of data. A buffer is essentially a chunk of memory allocated outside of v8 heap, and we can put some data in that memory and that data is interpreted in one of many ways, depending on the length of character, for example. That's why when there is buffer, there is character encoding, because whatever we place in buffer does not have any character encoding, so to read it, we need to specify  an encoding. When we read content from files or socket, if we don't specify encoding we get back a buffer object. So a buffer is a lower-level data structure to represent a sequence of binary data, and unlike arrays, once a buffer is allocated, it can't be resized.

``` js
Ganeshs - MacBook - Air: ~ganesh$ node
Welcome to Node.js v12 .4 .0.
Type ".help"
for more information. >
    buffer {
        Buffer: [Function: Buffer] {
            poolSize: 8192,
            from: [Function: from],
            of: [Function: of ],
            alloc: [Function: alloc],
            allocUnsafe: [Function: allocUnsafe],
            allocUnsafeSlow: [Function: allocUnsafeSlow],
            isBuffer: [Function: isBuffer],
            compare: [Function: compare],
            isEncoding: [Function: isEncoding],
            concat: [Function: concat],
            byteLength: [Function: byteLength],
            [Symbol(kIsEncodingSymbol)]: [Function: isEncoding]
        },
        SlowBuffer: [Function: SlowBuffer],
        transcode: [Function: transcode],
        kMaxLength: 2147483647,
        kStringMaxLength: 1073741799,
        constants: {
            MAX_LENGTH: 2147483647,
            MAX_STRING_LENGTH: 1073741799
        },
        INSPECT_MAX_BYTES: [Getter / Setter]
    }
```

 We can create buffer using one of three major ways: `alloc()` creates a filled buffer of certain size.
 
 

``` js

> Buffer.alloc(8); <

Buffer 00 00 00 00 00 00 00 00 >
```

 While `allocUnsafe()` will not fill the created buffer. So that might contain old or sensitive data, and need to be filled right away. To fill a buffer we can use `fill()` .

``` js

> Buffer.allocUnsafe(8).fill() <

    Buffer 00 00 00 00 00 00 00 00 >
```

We can also create buffer from `from()` method.

buffer.js

``` js
const string = 'NodeJs';
const buffer = Buffer.from('NodeJs');

console.log(string, string.length);
console.log(buffer, buffer.length);
```

output

``` bash
Ganeshs-MacBook-Air:Desktop ganesh$ node buffer.js 
NodeJs 6
<Buffer 4e 6f 64 65 4a 73> 6
```

Buffers are useful when  we need to read the image file from tcp stream or compressed file or any other form of binary data access. Just like arrays and strings, on Buffers when can use operations like, includes, indexOf, and slice, but there are some differences with these methods when we use them on buffers. 
for examples, when we do slice operation on buffer, the sliced buffer shared the same memory with original buffer.

**StringDecoder**

When converting streams of binary data, we should use the `StringDecoder` module, because it handler multi-byte characters much better, especially incomplete multi-byte characters. The `StringDecoder` preserves the incomplete encoded characters internally until it's complete, and then it returns the result. The default toString operation on buffer does not do that.

