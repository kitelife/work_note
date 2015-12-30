# NodeJS

## 基本知识

Try changing the registry to the http version rather that the default https one using the command：

```shell
npm config set registry http://registry.npmjs.org/
```

------

To get the exact filename that will be loaded when `require()` is called, use the `require.resolve()` function.

Here is the high-level algorithm in pseudocode of what require.resolve does:

```
require(X) from module at path Y
1. If X is a core module,
   a. return the core module
   b. STOP
2. If X begins with './' or '/' or '../'
   a. LOAD_AS_FILE(Y + X)
   b. LOAD_AS_DIRECTORY(Y + X)
3. LOAD_NODE_MODULES(X, dirname(Y))
4. THROW "not found"

LOAD_AS_FILE(X)
1. If X is a file, load X as JavaScript text.  STOP
2. If X.js is a file, load X.js as JavaScript text.  STOP
3. If X.json is a file, parse X.json to a JavaScript Object.  STOP
4. If X.node is a file, load X.node as binary addon.  STOP

LOAD_AS_DIRECTORY(X)
1. If X/package.json is a file,
   a. Parse X/package.json, and look for "main" field.
   b. let M = X + (json main field)
   c. LOAD_AS_FILE(M)
2. If X/index.js is a file, load X/index.js as JavaScript text.  STOP
3. If X/index.json is a file, parse X/index.json to a JavaScript object. STOP
4. If X/index.node is a file, load X/index.node as binary addon.  STOP

LOAD_NODE_MODULES(X, START)
1. let DIRS=NODE_MODULES_PATHS(START)
2. for each DIR in DIRS:
   a. LOAD_AS_FILE(DIR/X)
   b. LOAD_AS_DIRECTORY(DIR/X)

NODE_MODULES_PATHS(START)
1. let PARTS = path split(START)
2. let I = count of PARTS - 1
3. let DIRS = []
4. while I >= 0,
   a. if PARTS[I] = "node_modules" CONTINUE
   c. DIR = path join(PARTS[0 .. I] + "node_modules")
   b. DIRS = DIRS + DIR
   c. let I = I - 1
5. return DIRS
```

------

Modules are cached based on their resolved filename. Since modules may resolve to a different filename based on the location of the calling module (loading from node_modules folders), it is not a guarantee that require('foo') will always return the exact same object, if it would resolve to different files.

------

NodeJS内部架构图：

![node-internal-arch](./media/nodejsarch.png)

## 值得关注

- [Node-Webkit](github.com/rogerwang/node-webkit)
- [apiDoc](https://github.com/apidoc/apidoc)
- [Docco](http://jashkenas.github.io/docco/)
- [libuv](http://libuv.org/)
- [Async.js](https://github.com/caolan/async)
- [lodash](https://lodash.com)

## 推荐阅读

- [libuv中文教程](http://luohaha.github.io/Chinese-uvbook/)
- [You-Dont-Know-JS Series](https://github.com/getify/You-Dont-Know-JS)
- [awesome-electron](https://github.com/sindresorhus/awesome-electron)
- [An Inside Look at the Architecture of NodeJS](http://mcgill-csus.github.io/student_projects/Submission2.pdf) √
- [Node.js Style Guide](https://github.com/felixge/node-style-guide) √
- [Anatomy of an HTTP Transaction](https://nodejs.org/en/docs/guides/anatomy-of-an-http-transaction/) √
- [module best practices](https://github.com/mattdesl/module-best-practices) √
- [stream-handbook](https://github.com/substack/stream-handbook) √ 值得关注使用
- [Mastering the filesystem in Node.js](https://medium.com/@yoshuawuyts/mastering-the-filesystem-in-node-js-4706b7cb0801#.dermpbiul)√
- [Node.js: Style and structure](http://caolan.org/posts/nodejs_style_and_structure/) √ 非常赞！


