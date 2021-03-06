# Code Painter

[![Build Status][]](http://travis-ci.org/jedhunsaker/codepainter)

Code Painter is a JavaScript beautifier that can transform JavaScript files
into the coding style of your choice. Style settings can be supplied via
predefined styles, a custom JSON file, command line settings, [EditorConfig][]
settings or it can even be inferred from one or more sample files. For example,
you could provide a code snippet from the same project with which the new code
is intended to integrate.

It uses the excellent [Esprima parser][] by [Ariya Hidayat][] and his
[contributors][] � thanks!

The name is inspired by Word's Format Painter, which does a similar job for
rich text.


## Installation

    $ npm install codepainter

To access the command globally, do a global install:

    $ npm install -g codepainter

*nix users might also have to add the following to their .bashrc file:

    PATH=$PATH:/usr/local/share/npm/bin


## CLI Usage

You can see the usage in the CLI directly by typing `codepaint` or
`codepaint --help`.

```
$ codepaint --help

  Code Painter beautifies JavaScript.

  Usage: codepaint [options] <command>

  Commands:

    infer [options] <globs>...  Infer coding style from file(s)
    xform [options] <globs>...  Transform file(s) to specified style

  Options:

    -h, --help     output help information
    -V, --version  output version information


$ codepaint infer --help

  Infer coding style from file(s)

  Usage: infer [options] <globs>...

  Options:

    -h, --help     output help information
    -d, --details  give a detailed report with trend scores

  Examples:

    $ codepaint infer "**/*.js"
    $ codepaint infer "**/*view.js" "**/*model.js"
    $ codepaint infer %s "**/*.js" -m
    $ codepaint infer %s "**/*.js" -e inferred.json


$ codepaint xform --help

  Transform file(s) to specified style

  Usage: xform [options] <globs>...

  Options:

    -h, --help                 output help information
    -i, --infer <glob>         code sample(s) to infer
    -p, --predef <name>        cascade predefined style (e.g., idiomatic)
    -j, --json <path>          cascade JSON style over predef style
    -s, --style <key>=<value>  cascade explicit style over JSON
    -e, --editor-config        cascade EditorConfig style over all others

  Examples:

    $ codepaint xform "**/*.js"
    $ codepaint xform "**/*view.js" "**/*model.js"
    $ codepaint xform %s "**/*.js" -i sample.js
    $ codepaint xform %s "**/*.js" -p idiomatic
    $ codepaint xform %s "**/*.js" -j custom.json
    $ codepaint xform %s "**/*.js" -s quote_type=null
    $ codepaint xform %s "**/*.js" -s indent_style=space -s indent_size=4
    $ codepaint xform %s "**/*.js" -e
```


## Library Usage

`.infer(<path|glob|globs|ReadableStream>[,options][,callback])`

`.transform(<path|glob|globs|ReadableStream>[,options])`

Library usage is intended to be every bit the same as CLI usage, so you can
expect the same options and arguments that the CLI requires.

The following example infers coding style from `foo.js` and uses that inferred
style to transform all .js files under the current directory.

```js
var codepainter = require('codepainter');
codepainter.infer('foo.js', function(inferredStyle) {
    codepainter.xform('**/*.js', {json: inferredStyle});
});

```

'foo.js' could also be an array or any readable stream. `xform` is an alias
for the `transform` method. You can use either one.

Great, so that's all nice and simple, but maybe you want to do something with
the output. We start by creating an instance of the Transformer class.

```js
var Transformer = require('codepainter').Transformer;
var transformer = new Transformer();
```

Now, we can listen to any of the following events:

### cascade

Every time one style cascades over another.

```js
transformer.on('cascade', cascade);
function cascade(styleBefore, styleToMerge, styleType) {
    // code here
}
```

### transform

Every time a file is transformed.

```js
transformer.on('transform', function(transformed, path) {
    // code here
}
```

### error

```js
transformer.on('error', function(err, inputPath) {
    // code here
}
```

### end

When all transformations have taken place.

```js
transformer.on('end', function(err, transformed, skipped, errored) {
    // code here
}
```

Of course, none of these events will fire if you don't perform the transform:

`transformer.transform(globs, options);`


## CLI Examples

    $ codepaint infer "**/*.js"

Infers coding style from all .js files under the current directory into a
single JSON object, which you can pipe out to another file if you want. It can
then be used in a transformation (below).

    $ codepaint xform "**/*.js"

This doesn't transform any files, but it does show you how many files would be
affected by the glob you've provided. Globs absolutely *must* be in quotes or
you will experience unexpected behavior!

    $ codepaint xform -i infer.js "**/*.js"

Transforms all .js files under the current directory with the style inferred
from infer.js

    $ codepaint xform -p idiomatic "**/*.js"

Transforms all .js files under the current directory with a Code Painter
pre-defined style. In this case, Idiomatic. The only other pre-defined styles
available at this time are mediawiki and hautelook.

    $ codepaint xform -j custom.json "**/*.js"

Transforms all .js files under the current directory with a custom style in
JSON format.

    $ codepaint xform -s indent\_style=space -s indent\_size=4 "**/*.js"

Transforms all .js files under the current directory with 2 settings:
`indent_style=space` and `indent_size=4`. You can specify as many settings as
you want and you can set values to `null` to disable them.

    $ codepaint xform -e /usr/local/bin/editorconfig "**/*.js"

Transforms all .js files under the current directory with the EditorConfig
settings defined for each individual file.

Refer to [EditorConfig Core Installation][] for installation instructions and
[EditorConfig][] for more information, including how to define and use
`.editorconfig` files.

    $ codepaint xform -i infer.js -p idiomatic -j custom.json
    -s end_of_line=null -e /usr/local/bin/editorconfig "**/*.js"

As you can see, you can use as many options as you want. Code Painter will
cascade your styles and report how the cascade has been performed, like so:

```
  Inferred style:
   + indent_style = tab
   + insert_final_newline = true
   + quote_type = auto
   + space_after_anonymous_functions = false
   + space_after_control_statements = false
   + spaces_around_operators = false
   + trim_trailing_whitespace = false
   + spaces_in_brackets = false

  hautelook style:
   * indent_style = space
   + indent_size = 4
   * trim_trailing_whitespace = true
   + end_of_line = lf
   = insert_final_newline = true
   = quote_type = auto
   * spaces_around_operators = true
   = space_after_control_statements = false
   = space_after_anonymous_functions = false
   * spaces_in_brackets = hybrid

  Supplied JSON file:
   * space_after_control_statements = true
   = indent_style = space
   * indent_size = 3

  Inline styles:
   x end_of_line = null

  Editor Config:
   + applied on a file-by-file basis

  ...........................

  REPORT: 27 files transformed
```


## Supported Style Properties

1.  **codepaint**: *false*

    Tells CodePainter to skip the file (no formatting). This property really
    only makes sense if you are using the `--editor-config` CLI option. This
    allows you to, for example, skip a vendor scripts directory.

1.  EditorConfig properties: **indent\_style**, **indent\_size**,
    **trim\_trailing\_whitespace** and **insert\_final\_newline**. Refer to
    [EditorConfig's documentation][] for more information.

1.  **quote\_type**: *single*, *double*, *auto*

    Specifies what kind of quoting you would like to use for string literals:

    `console.log("Hello world!")` -> `console.log('Hello world!')`

    Adds proper escaping when necessary, obviously.

    `console.log('Foo "Bar" Baz')` -> `console.log("Foo \"Bar\" Baz")`

    The *auto* setting infers the quoting with a precedence toward *single*
    mode.

    `console.log("Foo \"Bar\" Baz")` -> `console.log('Foo "Bar" Baz')` or
    `console.log('Foo \'Bar\' Baz')` -> `console.log("Foo 'Bar' Baz")`

1.  **space\_after\_control\_statements**: *true*, *false*

    Specifies whether or not there should be a space between if/for/while and
    the following open paren.

    `if(x === 4)` -> `if (x === 4)` or `while (foo()) {` -> `while(foo()) {`

1.  **space\_after\_anonymous\_functions**: *true*, *false*

    Specifies whether or not there should be a space between the `function`
    keyword and the following parens in anonymous functions.

    `function(x) { }` -> `function (x) { }`

1.  **spaces\_around\_operators**: *true*, *false*, *hybrid*

    Specifies whether or not there should be spaces around operators such as
    `+,=,+=,>=,!==`.

    `var x = 4;` -> `var x=4;` or `a>=b` -> `a >= b` or `a>>2` -> `a >> 2`

    *Hybrid* mode is mostly like the *true* setting, except it behaves as
    *false* on operators `*,/,%` and unary operators `!,~,+,-`, not to be
    confused with the `+` and `-` addition and subtraction operators.

    `var x = 4 * 2 + 1 / 7;` -> `var x = 4*2 + 1/7;`

1.  **spaces\_in\_brackets**: *true*, *false*, *hybrid*

    Specifies whether or not there should be spaces inside brackets, which
    includes `(),[],{}`. Empty pairs of brackets will always be shortened.

    `(x===4)` -> `( x===4 )` or `( )` -> `()`

    The *hybrid* setting mostly reflects Idiomatic style. Refer to
    [Idiomatic Style Manifesto][].


## Recommended Use

It is highly recommended that you use the EditorConfig approach to painting
your code. These are the steps you would follow to do so:

1.  Place an `.editorconfig` file at your project root. Refer to this
    project's [.editorconfig][] file for a point of reference as to how this
    might look. You can also scatter `.editorconfig` files elsewhere
    throughout your project to prevent CodePainter from doing any
    transformations (e.g., your vendor scripts folders). In this case, the
    `.editorconfig` file would simply read: `codepaint = false`.

1.  Install CodePainter for global use with `npm install -g codepainter`.

1.  You *could* run `codepaint` manually every time you want to do it, but you
    might find this next `.bashrc` shortcut more useful. The idea is to run
    this `gc` alias to a newly-defined `codepaint_git_commit` function. This,
    you do instead of running `git commit`. The caveat is that you need to
    stage your changes with `git add` before doing so. This is because the
    command runs `codepaint` only on staged `.js` files. Aside from this
    caveat, you can commit things mostly the same as you were used to before.
    Now, `gc` can paint your code before a commit and bail-out of the commit
    if there are issues with the process (e.g., JavaScript parse errors). The
    idea of formatting code before a commit is definitely controversial, but
    if you choose to do so anyway, here's the neat trick to put in your
    `.bashrc` file:

```bash
alias gc=codepaint_git_commit
codepaint_git_commit() {
    # 1. Gets a list of .js files in git staging and sends the list to CodePainter.
    # 2. CodePainter with the -e flag applies rules defined in your EditorConfig file(s).
    # 3. After CodePainter is done, your args are passed to the `git commit` command.
    codepaint -e $(git diff --name-only --cached | egrep '\.js$') && git commit $@
}
```

Any tips on how to improve this process are greatly appreciated. See the
[package.json][] file for contact information.


## License

Released under the MIT license.

[Build Status]: https://secure.travis-ci.org/jedhunsaker/codepainter.png?branch=master
[Esprima parser]: http://esprima.org/
[Ariya Hidayat]: http://ariya.ofilabs.com/
[contributors]: https://github.com/ariya/esprima/graphs/contributors
[EditorConfig]: http://editorconfig.org/
[EditorConfig's documentation]: http://editorconfig.org/
[EditorConfig Core Installation]: /editorconfig/editorconfig-core#installation
[Idiomatic Style Manifesto]: /rwldrn/idiomatic.js/#whitespace
[.editorconfig]: .editorconfig
[package.json]: package.json
