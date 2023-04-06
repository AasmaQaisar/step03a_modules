Modules in TypeScript
Modules
JavaScript has a long history of different ways to handle modularizing code. TypeScript having been around since 2012, has implemented support for a lot of these formats, but over time the community and the JavaScript specification has converged on a format called ES Modules (or ES6 modules). You might know it as the import/export syntax.

ES Modules was added to the JavaScript spec in 2015, and by 2020 had broad support in most web browsers and JavaScript runtimes.

For focus, the handbook will cover both ES Modules and its popular pre-cursor CommonJS module.exports = syntax, and you can find information about the other module patterns in the reference section under Modules.

How JavaScript Modules are Defined
In TypeScript, just as in ECMAScript 2015, any file containing a top-level import or export is considered a module.

Conversely, a file without any top-level import or export declarations is treated as a script whose contents are available in the global scope (and therefore to modules as well).

Modules are executed within their own scope, not in the global scope. This means that variables, functions, classes, etc. declared in a module are not visible outside the module unless they are explicitly exported using one of the export forms. Conversely, to consume a variable, function, class, interface, etc. exported from a different module, it has to be imported using one of the import forms.

Non-modules
Before we start, it’s important to understand what TypeScript considers a module. The JavaScript specification declares that any JavaScript files without an export or top-level await should be considered a script and not a module.

Inside a script file variables and types are declared to be in the shared global scope, and it’s assumed that you’ll either use the outFile compiler option to join multiple input files into one output file, or use multiple <script> tags in your HTML to load these files (in the correct order!).

If you have a file that doesn’t currently have any imports or exports, but you want to be treated as a module, add the line:

export {};
Try
which will change the file to be a module exporting nothing. This syntax works regardless of your module target.

Modules in TypeScript
Additional Reading:
Impatient JS (Modules)
MDN: JavaScript Modules
There are three main things to consider when writing module-based code in TypeScript:

Syntax: What syntax do I want to use to import and export things?
Module Resolution: What is the relationship between module names (or paths) and files on disk?
Module Output Target: What should my emitted JavaScript module look like?
ES Module Syntax
A file can declare a main export via export default:

// @filename: hello.ts
export default function helloWorld() {
  console.log("Hello, world!");
}
Try
This is then imported via:

import helloWorld from "./hello.js";
helloWorld();
Try
In addition to the default export, you can have more than one export of variables and functions via the export by omitting default:

// @filename: maths.ts
export var pi = 3.14;
export let squareTwo = 1.41;
export const phi = 1.61;
 
export class RandomNumberGenerator {}
 
export function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}
Try
These can be used in another file via the import syntax:

import { pi, phi, absolute } from "./maths.js";
 
console.log(pi);
const absPhi = absolute(phi);
        
const absPhi: number
Try
Additional Import Syntax
An import can be renamed using a format like import {old as new}:

import { pi as π } from "./maths.js";
 
console.log(π);
           
(alias) var π: number
import π
Try
You can mix and match the above syntax into a single import:

// @filename: maths.ts
export const pi = 3.14;
export default class RandomNumberGenerator {}
 
// @filename: app.ts
import RandomNumberGenerator, { pi as π } from "./maths.js";
 
RandomNumberGenerator;
         
(alias) class RandomNumberGenerator
import RandomNumberGenerator
 
console.log(π);
           
(alias) const π: 3.14
import π
Try
You can take all of the exported objects and put them into a single namespace using * as name:

// @filename: app.ts
import * as math from "./maths.js";
 
console.log(math.pi);
const positivePhi = math.absolute(math.phi);
          
const positivePhi: number
Try
You can import a file and not include any variables into your current module via import "./file":

// @filename: app.ts
import "./maths.js";
 
console.log("3.14");
Try
In this case, the import does nothing. However, all of the code in maths.ts was evaluated, which could trigger side-effects which affect other objects.

TypeScript Specific ES Module Syntax
Types can be exported and imported using the same syntax as JavaScript values:

// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number };
 
export interface Dog {
  breeds: string[];
  yearOfBirth: number;
}
 
// @filename: app.ts
import { Cat, Dog } from "./animal.js";
type Animals = Cat | Dog;
Try
TypeScript has extended the import syntax with two concepts for declaring an import of a type:

import type
Which is an import statement which can only import types:

// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number };
'createCatName' cannot be used as a value because it was imported using 'import type'.
export type Dog = { breeds: string[]; yearOfBirth: number };
export const createCatName = () => "fluffy";
 
// @filename: valid.ts
import type { Cat, Dog } from "./animal.js";
export type Animals = Cat | Dog;
 
// @filename: app.ts
import type { createCatName } from "./animal.js";
const name = createCatName();
Try
Inline type imports
TypeScript 4.5 also allows for individual imports to be prefixed with type to indicate that the imported reference is a type:

// @filename: app.ts
import { createCatName, type Cat, type Dog } from "./animal.js";
 
export type Animals = Cat | Dog;
const name = createCatName();
Try
Together these allow a non-TypeScript transpiler like Babel, swc or esbuild to know what imports can be safely removed.

ES Module Syntax with CommonJS Behavior
TypeScript has ES Module syntax which directly correlates to a CommonJS and AMD require. Imports using ES Module are for most cases the same as the require from those environments, but this syntax ensures you have a 1 to 1 match in your TypeScript file with the CommonJS output:

import fs = require("fs");
const code = fs.readFileSync("hello.ts", "utf8");
Try
You can learn more about this syntax in the modules reference page.

CommonJS Syntax
CommonJS is the format which most modules on npm are delivered in. Even if you are writing using the ES Modules syntax above, having a brief understanding of how CommonJS syntax works will help you debug easier.

Exporting
Identifiers are exported via setting the exports property on a global called module.

function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}
 
module.exports = {
  pi: 3.14,
  squareTwo: 1.41,
  phi: 1.61,
  absolute,
};
Try
Then these files can be imported via a require statement:

const maths = require("maths");
maths.pi;
      
any
Try
Or you can simplify a bit using the destructuring feature in JavaScript:

const { squareTwo } = require("maths");
squareTwo;
   
const squareTwo: any
Try
CommonJS and ES Modules interop
There is a mis-match in features between CommonJS and ES Modules regarding the distinction between a default import and a module namespace object import. TypeScript has a compiler flag to reduce the friction between the two different sets of constraints with esModuleInterop.

TypeScript’s Module Resolution Options
Module resolution is the process of taking a string from the import or require statement, and determining what file that string refers to.

TypeScript includes two resolution strategies: Classic and Node. Classic, the default when the compiler option module is not commonjs, is included for backwards compatibility. The Node strategy replicates how Node.js works in CommonJS mode, with additional checks for .ts and .d.ts.

There are many TSConfig flags which influence the module strategy within TypeScript: moduleResolution, baseUrl, paths, rootDirs.

For the full details on how these strategies work, you can consult the Module Resolution.

TypeScript’s Module Output Options
There are two options which affect the emitted JavaScript output:

target which determines which JS features are downleveled (converted to run in older JavaScript runtimes) and which are left intact
module which determines what code is used for modules to interact with each other
Which target you use is determined by the features available in the JavaScript runtime you expect to run the TypeScript code in. That could be: the oldest web browser you support, the lowest version of Node.js you expect to run on or could come from unique constraints from your runtime - like Electron for example.

All communication between modules happens via a module loader, the compiler option module determines which one is used. At runtime the module loader is responsible for locating and executing all dependencies of a module before executing it.

For example, here is a TypeScript file using ES Modules syntax, showcasing a few different options for module:

import { valueOfPi } from "./constants.js";
 
export const twoPi = valueOfPi * 2;
Try
ES2020
import { valueOfPi } from "./constants.js";
export const twoPi = valueOfPi * 2;
 
Try
CommonJS
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
exports.twoPi = void 0;
const constants_js_1 = require("./constants.js");
exports.twoPi = constants_js_1.valueOfPi * 2;
 
Try
UMD
(function (factory) {
    if (typeof module === "object" && typeof module.exports === "object") {
        var v = factory(require, exports);
        if (v !== undefined) module.exports = v;
    }
    else if (typeof define === "function" && define.amd) {
        define(["require", "exports", "./constants.js"], factory);
    }
})(function (require, exports) {
    "use strict";
    Object.defineProperty(exports, "__esModule", { value: true });
    exports.twoPi = void 0;
    const constants_js_1 = require("./constants.js");
    exports.twoPi = constants_js_1.valueOfPi * 2;
});
 
ECMAScript Modules in Node.js
For the last few years, Node.js has been working to support running ECMAScript modules (ESM). This has been a very difficult feature to support, since the foundation of the Node.js ecosystem is built on a different module system called CommonJS (CJS).

Interoperating between the two module systems brings large challenges, with many new features to juggle; however, support for ESM in Node.js is now implemented in Node.js, and the dust has begun to settle.

That’s why TypeScript brings two new module and moduleResolution settings: node16 and nodenext.

{
    "compilerOptions": {
        "module": "nodenext",
    }
}
These new modes bring a few high-level features which we’ll explore here.

type
in
package.json
and New Extensions
Node.js supports a new setting in package.json called type. "type" can be set to either "module" or "commonjs".

{
    "name": "my-package",
    "type": "module",
    "//": "...",
    "dependencies": {
    }
}
This setting controls whether .js and .d.ts files are interpreted as ES modules or CommonJS modules, and defaults to CommonJS when not set. When a file is considered an ES module, a few different rules come into play compared to CommonJS:

import/export statements and top-level await can be used
relative import paths need full extensions (e.g we have to write import "./foo.js" instead of import "./foo")
imports might resolve differently from dependencies in node_modules
certain global-like values like require() and __dirname cannot be used directly
CommonJS modules get imported under certain special rules
We’ll come back to some of these.

To overlay the way TypeScript works in this system, .ts and .tsx files now work the same way. When TypeScript finds a .ts, .tsx, .js, or .jsx file, it will walk up looking for a package.json to see whether that file is an ES module, and use that to determine:

how to find other modules which that file imports
and how to transform that file if producing outputs
When a .ts file is compiled as an ES module, ECMAScript import/export syntax is left alone in the .js output; when it’s compiled as a CommonJS module, it will produce the same output you get today under module: commonjs.

This also means paths resolve differently between .ts files that are ES modules and ones that are CJS modules. For example, let’s say you have the following code today:

// ./foo.ts
export function helper() {
    // ...
}
// ./bar.ts
import { helper } from "./foo"; // only works in CJS
helper();
This code works in CommonJS modules, but will fail in ES modules because relative import paths need to use extensions. As a result, it will have to be rewritten to use the extension of the output of foo.ts - so bar.ts will instead have to import from ./foo.js.

// ./bar.ts
import { helper } from "./foo.js"; // works in ESM & CJS
helper();
This might feel a bit cumbersome at first, but TypeScript tooling like auto-imports and path completion will typically just do this for you.

One other thing to mention is the fact that this applies to .d.ts files too. When TypeScript finds a .d.ts file in package, whether it is treated as an ESM or CommonJS file is based on the containing package.

New File Extensions
The type field in package.json is nice because it allows us to continue using the .ts and .js file extensions which can be convenient; however, you will occasionally need to write a file that differs from what type specifies. You might also just prefer to always be explicit.

Node.js supports two extensions to help with this: .mjs and .cjs. .mjs files are always ES modules, and .cjs files are always CommonJS modules, and there’s no way to override these.

In turn, TypeScript supports two new source file extensions: .mts and .cts. When TypeScript emits these to JavaScript files, it will emit them to .mjs and .cjs respectively.

Furthermore, TypeScript also supports two new declaration file extensions: .d.mts and .d.cts. When TypeScript generates declaration files for .mts and .cts, their corresponding extensions will be .d.mts and .d.cts.

Using these extensions is entirely optional, but will often be useful even if you choose not to use them as part of your primary workflow.

CommonJS Interop
Node.js allows ES modules to import CommonJS modules as if they were ES modules with a default export.

// @filename: helper.cts
export function helper() {
    console.log("hello world!");
}
 
// @filename: index.mts
import foo from "./helper.cjs";
 
// prints "hello world!"
foo.helper();
Try
In some cases, Node.js also synthesizes named exports from CommonJS modules, which can be more convenient. In these cases, ES modules can use a “namespace-style” import (i.e. import * as foo from "..."), or named imports (i.e. import { helper } from "...").

// @filename: helper.cts
export function helper() {
    console.log("hello world!");
}
 
// @filename: index.mts
import { helper } from "./helper.cjs";
 
// prints "hello world!"
helper();
Try
There isn’t always a way for TypeScript to know whether these named imports will be synthesized, but TypeScript will err on being permissive and use some heuristics when importing from a file that is definitely a CommonJS module.

One TypeScript-specific note about interop is the following syntax:

import foo = require("foo");
In a CommonJS module, this just boils down to a require() call, and in an ES module, this imports createRequire to achieve the same thing. This will make code less portable on runtimes like the browser (which don’t support require()), but will often be useful for interoperability. In turn, you can write the above example using this syntax as follows:

// @filename: helper.cts
export function helper() {
    console.log("hello world!");
}
 
// @filename: index.mts
import foo = require("./foo.cjs");
 
foo.helper()
Try
Finally, it’s worth noting that the only way to import ESM files from a CJS module is using dynamic import() calls. This can present challenges, but is the behavior in Node.js today.

You can read more about ESM/CommonJS interop in Node.js here.

package.json
Exports, Imports, and Self-Referencing
Node.js supports a new field for defining entry points in package.json called "exports". This field is a more powerful alternative to defining "main" in package.json, and can control what parts of your package are exposed to consumers.

Here’s an package.json that supports separate entry-points for CommonJS and ESM:

// package.json
{
    "name": "my-package",
    "type": "module",
    "exports": {
        ".": {
            // Entry-point for `import "my-package"` in ESM
            "import": "./esm/index.js",
            // Entry-point for `require("my-package") in CJS
            "require": "./commonjs/index.cjs",
        },
    },
    // CJS fall-back for older versions of Node.js
    "main": "./commonjs/index.cjs",
}
There’s a lot to this feature, which you can read more about on the Node.js documentation. Here we’ll try to focus on how TypeScript supports it.

With TypeScript’s original Node support, it would look for a "main" field, and then look for declaration files that corresponded to that entry. For example, if "main" pointed to ./lib/index.js, TypeScript would look for a file called ./lib/index.d.ts. A package author could override this by specifying a separate field called "types" (e.g. "types": "./types/index.d.ts").

The new support works similarly with import conditions. By default, TypeScript overlays the same rules with import conditions - if you write an import from an ES module, it will look up the import field, and from a CommonJS module, it will look at the require field. If it finds them, it will look for a colocated declaration file. If you need to point to a different location for your type declarations, you can add a "types" import condition.

// package.json
{
    "name": "my-package",
    "type": "module",
    "exports": {
        ".": {
            // Entry-point for `import "my-package"` in ESM
            "import": {
                // Where TypeScript will look.
                "types": "./types/esm/index.d.ts",
                // Where Node.js will look.
                "default": "./esm/index.js"
            },
            // Entry-point for `require("my-package") in CJS
            "require": {
                // Where TypeScript will look.
                "types": "./types/commonjs/index.d.cts",
                // Where Node.js will look.
                "default": "./commonjs/index.cjs"
            },
        }
    },
    // Fall-back for older versions of TypeScript
    "types": "./types/index.d.ts",
    // CJS fall-back for older versions of Node.js
    "main": "./commonjs/index.cjs"
}
The "types" condition should always come first in "exports".

It’s important to note that the CommonJS entrypoint and the ES module entrypoint each needs its own declaration file, even if the contents are the same between them. Every declaration file is interpreted either as a CommonJS module or as an ES module, based on its file extension and the "type" field of the package.json, and this detected module kind must match the module kind that Node will detect for the corresponding JavaScript file for type checking to be correct. Attempting to use a single .d.ts file to type both an ES module entrypoint and a CommonJS entrypoint will cause TypeScript to think only one of those entrypoints exists, causing compiler errors for users of the package.

TypeScript also supports the "imports" field of package.json in a similar manner (looking for declaration files alongside corresponding files), and supports packages self-referencing themselves. These features are generally not as involved, but are supported.