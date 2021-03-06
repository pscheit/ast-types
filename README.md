AST Types
===

This module provides an efficient, modular,
[Esprima](https://github.com/ariya/esprima)-compatible implementation of
the [abstract syntax
tree](http://en.wikipedia.org/wiki/Abstract_syntax_tree) type hierarchy
pioneered by the [Mozilla Parser
API](https://developer.mozilla.org/en-US/docs/SpiderMonkey/Parser_API).

[![Build Status](https://travis-ci.org/benjamn/ast-types.png?branch=master)](https://travis-ci.org/benjamn/ast-types)

Installation
---

From NPM:

    npm install ast-types

From GitHub:

    cd path/to/node_modules
    git clone git://github.com/benjamn/ast-types.git
    cd ast-types
    npm install .

Basic Usage
---
```js
var assert = require("assert");
var n = require("ast-types").namedTypes;
var b = require("ast-types").builders;

var fooId = b.identifier("foo");
var ifFoo = b.ifStatement(fooId, b.blockStatement([
    b.expressionStatement(b.callExpression(fooId, []))
]));

assert.ok(n.IfStatement.check(ifFoo));
assert.ok(n.Statement.check(ifFoo));
assert.ok(n.Node.check(ifFoo));

assert.ok(n.BlockStatement.check(ifFoo.consequent));
assert.strictEqual(
    ifFoo.consequent.body[0].expression.arguments.length,
    0);

assert.strictEqual(ifFoo.test, fooId);
assert.ok(n.Expression.check(ifFoo.test));
assert.ok(n.Identifier.check(ifFoo.test));
assert.ok(!n.Statement.check(ifFoo.test));
```

Custom AST Node Types
---

The `ast-types` module was designed to be extended. To that end, it
provides a readable, declarative syntax for specifying new AST node types,
based primarily upon the `require("ast-types").Type.def` function:
```js
var types = require("ast-types");
var def = types.Type.def;
var string = types.builtInTypes.string;
var b = types.builders;

// Suppose you need a named File type to wrap your Programs.
def("File")
    .bases("Node")
    .build("name", "program")
    .field("name", string)
    .field("program", def("Program"));

// Prevent further modifications to the File type (and any other
// types newly introduced by def(...)).
types.finalize();

// The b.file builder function is now available. It expects two
// arguments, as named by .build("name", "program") above.
var main = b.file("main.js", b.program([
    // Pointless program contents included for extra color.
    b.functionDeclaration(b.identifier("succ"), [
        b.identifier("x")
    ], b.blockStatement([
        b.returnStatement(
            b.binaryExpression(
                "+", b.identifier("x"), b.literal(1)
            )
        )
    ]))
]));

assert.strictEqual(main.name, "main.js");
assert.strictEqual(main.program.body[0].params[0].name, "x");
// etc.

// If you pass the wrong type of arguments, or fail to pass enough
// arguments, an AssertionError will be thrown.

b.file(b.blockStatement([]));
// ==> AssertionError: {"body":[],"type":"BlockStatement","loc":null} does not match type string

b.file("lib/types.js", b.thisExpression());
// ==> AssertionError: {"type":"ThisExpression","loc":null} does not match type Program
```
The `def` syntax is used to define all the default AST node types found in
https://github.com/benjamn/ast-types/blob/master/lib/core.js,
https://github.com/benjamn/ast-types/blob/master/lib/es6.js,
https://github.com/benjamn/ast-types/blob/master/lib/mozilla.js,
https://github.com/benjamn/ast-types/blob/master/lib/e4x.js, and
https://github.com/benjamn/ast-types/blob/master/lib/xjs.js, so you have
no shortage of examples to learn from.
