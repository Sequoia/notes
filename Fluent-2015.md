# [Chrome Dev Tools](http://conferences.oreilly.com/fluent/javascript-html-us/public/schedule/detail/46324) 
* Cmd+P to fuzzyfind files in source
* Cmd+shift+O to find function names
* In call stack, to hide parts of stack from libs (jquery, async, etc): right click &rarr; "Blackbox script"
* "When you see a box in chrome devtools &rarr; click it!"
* Wrap console logs in a function `console.group('label for that function');` `console.groupEnd();`
* `console.log('%cHere's Some Text', 'color:red; cssproperty:cssvalue')` &larr; format console messages
* `console.table` for *collections* (array of similar objects) *(Note: this works better in Firefox, it is more flexible)*
  * `console.table(collection, listOfFields)` filter properties
* click and **hold** reload button: dropdown of different types of reload
* Network tab: throttling dropdown: simulate slower network
  * click the camera: start **recording** &rarr; get **screenshots** of page state at different times during loading :100:
## Experiments (settings &rarr; experiments)
* enable custom UI themes :sunglasses:
## Stupid Devtools Tricks
* `document.body.contentEditable = true` &larr; turn page into wysiwyg editor

# Graph-based recommendation engine
* Users generate recommendations (i.e. we use historical data of user behavior)
```js
// https://gist.github.com/keithwhor/edd064b6de0adc9d982f
let joe = {type: 'node', properties: {name: 'joe'}, input: [], output: []};
let likes = {type: 'edge', properties: {name: 'likes'}, input: null, output: null};
let minecraft = {type: 'node', properties: {name: 'minecraft'}, input: [], output: []};

joe.output.push(likes);
likes.input = joe;
likes.output = minecraft;
minecraft.input.push(likes);
```

# [Stranger danger: Addressing the security risk in npm dependencies](http://conferences.oreilly.com/fluent/javascript-html-us/public/schedule/detail/48353)
[Danny Grander](https://twitter.com/grander) | [Guy Podjarny](https://twitter.com/guypod)
* Most of the lines of code in "your application" are actually in dependencies.  Lots of space for vulns in 3rd party code
## Snyk
* `npm install -g snyk`
* Run snyk on your codebase, **checks your deps against db of known vulns**

### Vulns it found
* `url/%2e%2e/` &larr; urlencoded dot dot (vuln from [st](https://www.npmjs.com/package/st))
* "Regular Expression Denial of Service" &larr; regex can take lots of time to
  process, if you can give a service a really complicated regex or a string it
  can use a lot of CPU
  1. Send super long string (numbers ideally)
  2. Make it not match (takes longer to test nonmatching)
* RegexDOS is a problem in every env., but **especially significant in node
  because you block the eventloop**
  * :question: can you time & exit to avoid this?
* To hit the buffer String/NUmber vuln:
  1. Submit as JSON
  2. Send a number where a strnig should be
  3. Get unititialized memory :
* :bulb: `repeat` command in &larr; linux: try this out

## Snyk Wizard
* Like `snyk`, also recommends fixes & lets you pick & do it.
* Tries to remediate using one of the following:
  1. Upgrade immediate dep
  2. Upgrade parent dep (if it's a dep's dep that's vulnerable)
  3. Apply patchfile created created by snyk
* If you ignore, it asks for reason (writes to audit file?)
* `snyk test` : for CI, fails build on new vuln
* `snyk protect`: takes snapshot of all your code & uploads to snyk.io so they
  can alert you on new stuff
