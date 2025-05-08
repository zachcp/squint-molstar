---
title: Squint Testing
toc:
  depth: 3
---



## Using Squint from JS

The squint library is available precompiled from NPM and is importable as `squint_core`. This is the CLJS standard library but using vanilla JS objects.

```js echo
import * as squint_core from 'npm:squint-cljs/core.js';
// this one will print in the console
eval(`(async function() {
  squint_core.println("I'm just sitting on the dock of the bay ....")})()
`).value
```

## Compiling CLJ-> JS with Squint

The squint compiler will convert CLJS to JS. But you will need `squint_core` as above to use it.

```js echo
import * as squint from 'npm:squint-cljs';
import * as squint_core from 'npm:squint-cljs/core.js';

// Showing Arrows and Assoc work as per CLJS
const code = `
  (-> {}
    (assoc :greatline "...I'm just sitting on the dock of the bay....")
    (assoc :another "... Watching as that ships roll in..."))`;

// compiler action is there
// note: that I am getting rid of imports (first line).
// Leave this if you are in Node.
const compilerState = squint.compileStringEx(
  `(do ${code})`, {repl: false, context: 'return', "elide-exports": true}, null);
const js = compilerState.javascript;
const import_index = js.indexOf('\n');
const newString2 = js.substring(import_index + 1);
const result = {value: eval(`(async function() { ${newString2} })()`)};
view(await (result).value);
```

## Squint + Text Box

Type some CLJS code and  hit submit to view to output.

```js
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

const essay = view(Inputs.textarea({label: "Squint CLJS Eval", rows: 6, minlength: 10, submit: true}));
```

### Compiled Code Here

```js
view(await compile(essay))
```

### Example Code

Cut/paste this code into the box above and hit Enter.

```
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



## Squint + CodeMirror

Hit CMD+Enter To Evaluate.

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
  :timestamp: "2025-04-14T19:04:58.549065+00:00"}
}
`

let theme = EditorView.theme({
  ".cm-content": {whitespace: "pre-wrap",
                  passing: "10px 0",
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
const codemirror_out = Mutable({
  value: null
})


// the MVSJ Viewer
const app2 = display(document.querySelector("#molstar2"));

const cljs_viewer = molstar.Viewer.create(app2, {
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
    console.log("compiled");
    console.log(result);

    // Update with completed value
    codemirror_out.value = result;
    // update the msvj with the new data
    console.log(result);
    console.log(JSON.stringify(result));
    console.log(codemirror_out);

    console.log(cljs_viewer);
    const viewer = await cljs_viewer;
    console.log(viewer);
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
  return keymap.of([{key: "Alt-Enter", run: evalCell},
                    // {key: opts.modifier + "-Enter",
                    //  run: evalAtCursor,
                    //  shift: evalToplevel
                    // }
  ])}

let extensions = [
  history(),
  theme,
  foldGutter(),
  syntaxHighlighting(defaultHighlightStyle),
  drawSelection(),
  keymap.of(complete_keymap),
  keymap.of(historyKeymap),
  ...default_extensions,
  eval_ext({modifier: "Meta"}),
  squintExtension({modifier: "Meta"})
];




let state = EditorState.create(
  {doc:example_code, extensions: extensions});
let editorElt = document.querySelector('#editor');
let editor = new EditorView({state: state,
                             parent: editorElt,
                             extensions: extensions });
console.log(editor);
```


```js echo
view(editor.state)
```

```js echo
view(editor.state.doc.text)
```


```js echo
view(editor.state.doc.text.join(""))
```

```js echo
view(await compile(editor.state.doc.text.join("")))
```

## CodeMirror Output

This updates on change.

```js
view(codemirror_out)
```



## Load Molstar


```html echo
<link rel="stylesheet" type="text/css" href="npm:molstar/build/viewer/molstar.css">
<div id="molstar" style="position: relative; width: 400px; height: 400px;">
```

```js
import "npm:molstar/build/viewer/molstar.js";
const molstar = window.molstar;
```


### Example MSVJ


```js
const example_mvsj = {
  "kind": "single",
  "root": {
    "kind": "root",
    "children": [
      {
        "kind": "download",
        "params": {
          "url": "https://files.wwpdb.org/download/1cbs.cif"
        },
        "children": [
          {
            "kind": "parse",
            "params": {
              "format": "mmcif"
            },
            "children": [
              {
                "kind": "structure",
                "params": {
                  "type": "model"
                },
                "children": [
                  {
                    "kind": "component",
                    "params": {
                      "selector": "all"
                    },
                    "children": [
                      {
                        "kind": "representation",
                        "params": {
                          "type": "cartoon"
                        },
                        "children": [
                          {
                            "kind": "color",
                            "params": {
                              "color": "blue"
                            }
                          }
                        ]
                      }
                    ]
                  }
                ]
              }
            ]
          }
        ]
      }
    ]
  },
  "metadata": {
    "timestamp": "2025-04-14T19:04:58.549065+00:00",
    "version": "1.4"
  }
}

console.log(example_mvsj);
console.log(JSON.stringify(example_mvsj));
view(example_mvsj)
```



### Example MSVJ from CLJS



```js
const app = display(document.querySelector("#molstar"));

molstar.Viewer.create(app, {
  layoutIsExpanded: false,
  layoutShowControls: false,
  layoutShowRemoteState: false,
  layoutShowSequence: true,
  layoutShowLog: false,
  layoutShowLeftPanel: true,
  viewportShowExpand: true,
  viewportShowSelectionMode: false,
  viewportShowAnimation: false,
  pdbProvider: "rcsb",
  emdbProvider: "rcsb"
}).then((viewer) => {
  viewer.loadMvsData(JSON.stringify(example_mvsj), "mvsj");
  return viewer;
});


```


```html echo
<div id="molstar2" style="position: relative; width: 400px; height: 400px;">
```

```js



// const editor_val = Generators.observe(change => {
//     change(editor.getValue());
//     editor.on('change', (cm, val) => {
//       change(editor.getValue());
//     });
// });

// console.log(editor_val);
```
