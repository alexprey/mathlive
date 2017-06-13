#MathLive Contributor Guide

This guide describes the structure of the MathLive project if you are 
interested in contributing to it, or if you want to better understand how
it works. If you simply want to use MathLive with your web content, see the
 [MathLive UsageGuide](USAGE_GUIDE.md).

## Code structure
The MathLive library consists of the following key directories:
* `css/` the stylesheets and fonts used by MathLive
* `src/core` the core Javascript code needed to render math. This module depends on the `css/` module.
* `src/editor` the Javascript code needed for the editor. This module depends
on the `src/core` module.
* `src/addons` some optional modules that provide additional functionality

## Language
MathLive is written in Javascript, using the ES2016 dialect. This includes
in particular features such as:
* `let` and `const` instead of `var`
* `Array.prototype.includes()`
* arrow functions (to a limited extent, there appears to be issues with transpilers)
* template strings
* `for...of` iterators
* `Object.assign()`

Features that have not been adopted include:
* classes. The syntax doesn't seem to offer that much benefit and forces 
utility functions to be separated from methods that use them.
* getters/setters: would probably be a good idea
* destructuring: probably somes opportunities to simply some code
* default parameters: would clean up some code
* rest/spread
* generators

Before publishing, the code is ran through [Babel](https://babeljs.io), which 
transpiles it to ES5 for compatibility with browsers that don't support all
those features

The code is written using 4-spaces and follow the linting described in the 
.eslintrc.json file.


## Compatibility
MathLive is designed for the modern web. Supporting older browsers complicates
the effort involved in building new features, but it is also an insecure 
practice that should not be encouraged. In this context, _modern_ means the
latest release of Chrome, IE, Safari, Firefox. Both desktop and mobile are
supported.

## Architecture
The core of MathLive is a renderer that can display complex math content
using HTML and CSS. This renderer is using the TeX layout algorithms. Given
the same input, it will render pixel for pixel what TeX would have rendered.
To do so, it make use of a web version of the fonts used by TeX and which are
included in the `css/fonts/` directory.

 Here are some of the key concepts used throughtout the code base.

 ### Span

A span is an object that is used to represent an element displayed in the page:
a symbol such as _x_ or _=_, an open brace, a line separating the numerator 
and denominator of a fraction, etc...

The basic layout strategy is to calculate the vertical placement of the spans and 
position them accordingly, while letting the HTML rendering engine position
and display the horizontal items. An exception is when some adjustment needs
to be made, such as additional space between items, in which case CSS adjustments
are made.

**Spans** can be rendered to HTML markup before being displayed on the page.

 ### Math Atom
An atom is an object encapsulating a mathematical unit, indepdent of its 
graphical representation. It can be of several types such as `mbin` (binary 
operator), `mrel` (relational operator), `genfrac` (generalized fraction).

 ### Math List
A **math list** is simply an array of Atom. Although it's a common data 
structure there are no classes to represent them: they are simply an Array() of MathAtom.

### Lexer
The **lexer** converts a string of TeX code into tokens that can be digested
by the parser.

### Parser
The **parser** turns a stream of tokens generated by the lexer into **math atoms**. Those atoms then can be rendered into **spans**, or back into 
LaTeX or into spoken text.

### Editable Math List
An **Editable Math List** is a class specific to the editor and that
encapsulates the manipulations that can be done to a math list, including 
adding and removing content and keeping track of and modifying an insertion 
point and selection.






## Naming Convention

Those naming conventions are particularly important for objects that are exposed
as part of the public API, such as MathField.

* variables and function names that begin with `_` are private and should not
 be used.
* functions that end in '_' are selectors and should not be invoked directly.
Instead, a `MathField.perform()` call should be made. Note that the perform call
does not include the `_`, so you would call `MathField.perform('selectAll')`.
* functions that neither begin nor end with an `_` are public and can be called
directly.
