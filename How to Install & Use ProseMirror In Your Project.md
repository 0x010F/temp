Prosemirror is an incredibly well-written rich text library. But, lacks proper *prose* on how to set it up in a project for beginner / occasional JS developers. By the time you’ve finished reading this you’d learn:

    + How to install prosemirror into your JS codebase (that isn't using fancy modern tools like Webpack, Babel, etc currently).
    + Why the installation is difficult.
    + Some modern JS practices.


This piece is intended for non full-time JS developers who were familiar with Javascript in its jQuery and &lt;script&gt; tag and ES5 days. If you are already familiar with ES6, module-bundling and imports you’d find no problem dealing with Prosemirror anyway.

---------------


The conventional way of using an external library was to include the JS file in your index.html in a script tag. This will attach a global namespace/variable for that library which will expose the library’s APIs and methods. 

For example:
You’ll include &lt;script src=”jquery.js”&gt; in your index.html
After that you’ll access all jQuery APIs via `window.jQuery` or `window.$` namespace.

Good News: Things have changed now. Most libraries are now written as ES6/CommonJS Modules which means your global namespace remains pristine. 

Bad News: Things have changed now. There are extra steps that you’d need to make before incorporating these libraries into your JS code. 

---------------------


I’m assuming you are either familiar with ES5 JS and / or have a legacy JS codebase. To use Prosemirror in your codebase, you’d have to find a way to export necessary Prosemirror APIs into the global namespace. 

Basically, build a ProseMirror.js file that you can use like jquery.js

--------------------


Create a folder & file structure as below:

```
prosemirror/
       /src
       /src/index.js
	   /public
	   /public/index.html
       package.json
       rollup.config.js
```

Since ProseMirror is written using ES6, and we want to use it with plain old ES5 Javascript, we need to transpile ProseMirror library along with its global APIs to ES5. 

We are going to do that with [Babel](https://babeljs.io/) & Use [Rollup](https://rollupjs.org/) as our module bundler. 

Create a `package.json` file
```
cd  prosemirror
```

Put the below content into package.json
```js
{
  "name": "setup-prosemirror",

  "dependencies": {
    "prosemirror-example-setup": "^1.0.1",
    "prosemirror-model": "^1.6.4",
    "prosemirror-schema-basic": "^1.0.0",
    "prosemirror-state": "^1.2.2",
    "prosemirror-view": "^1.6.8"
  },
  "devDependencies": {
    "@babel/core": "^7.2.2",
    "@babel/preset-env": "^7.2.3",
    "babel-loader": "^8.0.5",
    "rollup": "^1.1.0",
    "rollup-plugin-babel": "^4.3.1",
    "rollup-plugin-commonjs": "^9.2.0",
    "rollup-plugin-node-resolve": "^4.0.0"
  }
}

```
The package.json file declares all the dependencies to build a final `ProseMirror.js` file, including all the tools needed. 

Now it's time to use this `package.json` and install the dependencies. Run:
```
bash$ npm install
```
After the dependencies are installed, create a `rollup.config.js` file in the prosemirror directory and put below code in it.
This tells Rollup to take our src file from `src/index.js` and spit out a `ProseMirror.js` into `public/ProseMirror.js`

```js
import babel from 'rollup-plugin-babel';
import resolve from 'rollup-plugin-node-resolve';
import commonJS from 'rollup-plugin-commonjs'


export default {
  input: './src/index.js',
  output: {
    format: 'iife',
    file: 'public/ProseMirror.js',
    name: 'ProseMirror'
  },
  plugins: [
      babel(), 
      resolve(), 
      commonJS({
        include: 'node_modules/**'
      })
    ],
};
```

The configuration is mostly self-explanatory. We are telling Rollup to first convert all fancy ES6 code to ES5, then resolve all dependency libraries from `node_modules` folder and spit out a final `ProseMirror.js` as a self executing module format.


Now's the most important step – Writing the code to import all ProseMirror APIs and exporting them to `window` namespace.

Put below code into `src/index.js`
```
export { EditorState } from "prosemirror-state";
export { EditorView } from "prosemirror-view";
export { Schema, DOMParser, Node } from "prosemirror-model";
export { schema as basicSchema } from "prosemirror-schema-basic";
export { exampleSetup } from "prosemirror-example-setup";
```

Without Rollup we would have had to manually attach each API to window ourselves. Like: 
```js
window.ProseMirror = {}
window.ProseMirror.EditorState = EditorState;
```
But luckily we have a better way. Rollup can just parse this file and do that work us. 
Just run:
```
bash$ rollup -c
```

You will see a `ProseMirror.js` file being created at `public/ProseMirror.js`

All that is left is to use this compiled JS and initialize a ProseMirror editable area.
We'll include this js file in `public/index.html` and test out a small demo. 
Put the below code in `public/index.html`

```html
<!DOCTYPE html> <meta charset="utf8" />
<html>
<head>
    <link rel="stylesheet" href="https://prosemirror.net/css/editor.css" />
</head>
<body>
<div id="editor" style="margin-bottom: 23px"></div>
<script src="index.js"></script>
<script>
// window.ProseMirror is active by now.
var plugins = ProseMirror.exampleSetup({ schema: ProseMirror.basicSchema });
  
var view = new ProseMirror.EditorView(document.querySelector("#editor"), {
    state: ProseMirror.EditorState.create({
        schema: ProseMirror.basicSchema,
        plugins: plugins
    })
});
</script>
</body>
</html>
```

Open up `index.html` in a browser and you should see a nice little rich text editor out there!


DISCLAIMER:
However convenient this seems, this is a non-scalable way of using ProseMirror. For example anytime you want to use another API from ProseMirror, you'd have to edit `src/index.js`to export that API for you. Also, polluting the global namespace is a thing of the past. Moving your project to a good module system (like ES6 Modules) would be more beneficial than the cost involved. 