---
title: Squint Testing
toc:
  depth: 3
---


## Using Squint from JS

- The [squint library](https://github.com/squint-cljs/squint) is available precompiled [from NPM](https://www.npmjs.com/package/squint-cljs) and is importable as `squint_core`.
- This is the CLJS standard library but using vanilla JS objects.
- You can use JavaScript's `eval` to

```js echo
import * as squint_core from 'npm:squint-cljs/core.js';
// this works
eval(`squint_core.println("I'm sittin' on the dock of the bay ....");`)
// or you can invoke larger code blocks within functions
eval(
  `(async function() {
      squint_core.println(
        "I'm sittin' on the dock of the bay ....")})()
`)
```

<br>
<br>

## Compiling CLJ-> JS with Squint

- The squint COMPILER is what converts CLJS into JS.
- To execute the compiled JS you will need the `squint_core` library.
- this example just shows using CLJS syntax and yielding a JS Object.

```js echo
import * as squint from 'npm:squint-cljs';
import * as squint_core from 'npm:squint-cljs/core.js';

// Showing Arrows and Assoc work as per CLJS
const code = `
  (-> {}
    (assoc :greatline "... I'm sittin' on the dock of the bay ....")
    (assoc :another "... Watching the tide roll away ..."))`;

// compiler action is there
// note: I am getting rid of imports (first line).
// Leave this if you are in Node.
const compilerState = squint.compileStringEx(
  `(do ${code})`, {repl: false, context: 'return', "elide-exports": true}, null);
const js = compilerState.javascript;
const import_index = js.indexOf('\n');
const newString2 = js.substring(import_index + 1);
const result = {value: eval(`(async function() { ${newString2} })()`)};
view(await (result).value);
```

<br>
<br>


## Squint + Text Box

- Since we are in Observable Framework, let's try a [TextArea](https://observablehq.com/@observablehq/input-textarea)
- To execute the compiled JS you will need the `squint_core` library.
- this example just shows using CLJS syntax and yielding a JS Object.
- Type some CLJS code and hit submit to view the output.
- Try the code I've provided below.

```js echo
import * as squint from 'npm:squint-cljs';
import * as squint_core from 'npm:squint-cljs/core.js';

function compile(code) {
  let compilerState = squint.compileStringEx(
    `(do ${code})`, {repl: false, context: 'return', "elide-exports": true}, null);
  let js = compilerState.javascript;
  let import_index = js.indexOf('\n');
  let newString2 = js.substring(import_index + 1);
  let result = {value: eval(`(async function() { ${newString2} })()`)};
  console.log(result.value);
  return result.value
}

const essay = view(Inputs.textarea({label: "Squint CLJS Eval", rows: 3, submit: true, placeholder: "(+ 1 10 100)"}));
view(compile(essay))
```

### Compiled Code Here

```js
view(await compile(essay))
```

### Example Code


```{clj}
;; cut/paste this above
(defn nodize
  "Transforms a hiccup-like vector [:name props children] into a node map."
  [form]
  (let [[name-val props-val children-val] form]
    (cond-> {"kind" name-val}
      (not (empty? props-val)) (assoc "params" props-val)
      (seq children-val) (assoc "children" (mapv nodize children-val)))))

(nodize
  [:root {:meta nil} [
    [:child1 {:awesome "yes"} nil ]
    [:child2 {:awesome "less_so"} nil]]])
```


<br>
<br>
<br>


## CodeMirror + Mol*

- This takes the same principle as above but extends it to use the output JSON as input to another component.
- We are using the [Mol*](https://molstar.org) Viewer, a browser viewer for molecular structures.
- We are targetting the [MolViewSpec](https://molstar.org/mol-view-spec/), a Mol* extension and declarative visualization standard.
- The standard uses node trees in JSON.
- Here we are going to use a single cljs FN to compress the node tree.
- Hit ALT+Enter To evaluate the code.
- try changing `blue` to `green` and reevaluating.

```html
 <div id="editor"
     class="rounded-md mb-0 py-2 text-sm monospace overflow-auto relative border shadow-lg bg-white"
     style="border: 2px dotted lightgray;
     border-radius: 0.375rem;
     margin-bottom: 0;
     padding-top: 0.5rem;
     padding-bottom: 0.5rem;
     font-size: 0.875rem;
     font-family: monospace;
     overflow: auto;
     position: relative;
     box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
     background-color: white;">
</div>

<link rel="stylesheet" type="text/css" href="npm:molstar/build/viewer/molstar.css">
<div id="molstar" style="position: relative; width: 400px; height: 400px;">

```

```js
// Code From here: https://github.com/nextjournal/clojure-mode/blob/main/public/squint/js/demo.mjs
import { default_extensions, complete_keymap } from 'npm:@nextjournal/clojure-mode';
import { EditorView, drawSelection, keymap } from  'npm:@codemirror/view';
import { EditorState } from  'npm:@codemirror/state';
import { history, historyKeymap } from 'npm:@codemirror/commands';
import { syntaxHighlighting, defaultHighlightStyle, foldGutter } from 'npm:@codemirror/language';
import { extension as eval_ext, cursor_node_string, top_level_string } from 'npm:@nextjournal/clojure-mode/extensions/eval-region';

let example_code = `(defn nodize
  "Transforms a hiccup-like vector [:name props children] into a node map."
  [form]
  (let [[name-val props-val children-val] form]
    (cond-> {"kind" name-val}
      (not (empty? props-val)) (assoc "params" props-val)
      (seq children-val) (assoc "children" (mapv nodize children-val)))))

  (def state-01
    (nodize
      [:root {}
        [[:download {:url "https://files.wwpdb.org/download/1cbs.cif"}
          [[:parse {:format "mmcif"}
            [[:structure {:type "model"}
              [[:component {:selector "all"}
                [[:representation {:type "cartoon"}
                  [[:color {:color "blue"} nil]]]]]]]]]]]]]))

  {:kind "single"
  :root state-01
  :metadata
  {:version "1.4"
  :timestamp "2025-04-14T19:04:58.549065+00:00"}
}
`

let theme = EditorView.theme({
  ".cm-content": {whitespace: "pre-wrap",
                  padding: "10px 0",
                  flex: "1 1 0"},

  "&.cm-focused": {outline: "0 !important"},
  ".cm-line": {"padding": "0 9px",
               "line-height": "1.6",
               "font-size": "16px",
               "font-family": "var(--code-font)"},
  ".cm-matchingBracket": {"border-bottom": "1px solid var(--teal-color)",
                          "color": "inherit"},
  ".cm-gutters": {background: "transparent",
                  border: "none"},
  ".cm-gutterElement": {"margin-left": "5px"},
  // only show cursor when focused
  ".cm-cursor": {visibility: "hidden"},
  "&.cm-focused .cm-cursor": {visibility: "visible"}
});
```



```js
// use this to hold compiled outputs from CodeMirror
const codemirror_out = Mutable({ value: null })


// the MVSJ Viewer
const app = display(document.querySelector("#molstar"));

const cljs_viewer = molstar.Viewer.create(app, {
  layoutIsExpanded: false,
  layoutShowControls: false,
  layoutShowRemoteState: false,
  layoutShowSequence: true,
  layoutShowLog: false,
  layoutShowLeftPanel: false,
  viewportShowExpand: true,
  viewportShowSelectionMode: false,
  viewportShowAnimation: false,
  pdbProvider: "rcsb",
  emdbProvider: "rcsb"
}).then((viewer) => {
  return viewer;
});

const evalCell = async (opts) => {
  let code = opts.state.doc.toString();
  console.log("Evaluating Cell Code");

  try {
    // Await the compiled result
    const result = await compile(code);
    const viewer = await cljs_viewer;

    codemirror_out.value = result;
    viewer.loadMvsData(JSON.stringify(result), "mvsj", {replaceExisting: true} )

  } catch (error) {
    console.error("Compilation error:", error);
    codemirror_out = {
      pending: false,
      error: error
    };
  }
  return true;
}

let squintExtension = ( opts ) => {
  return keymap.of([
    {key: "Alt-Enter", run: evalCell},
])}

let extensions = [
  history(), theme, foldGutter(),
  syntaxHighlighting(defaultHighlightStyle), drawSelection(),
  keymap.of(complete_keymap), keymap.of(historyKeymap),
  ...default_extensions, eval_ext({modifier: "Meta"}),
  squintExtension({modifier: "Meta"})
];


let state = EditorState.create(
  {doc:example_code, extensions: extensions});
let editorElt = document.querySelector('#editor');
let editor = new EditorView({state: state,
                             parent: editorElt,
                             extensions: extensions });
```


```js
import "npm:molstar/build/viewer/molstar.js";
const molstar = window.molstar;
```
