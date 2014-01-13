---
layout: default
title: Fez - Documentation
---

Documentation
=============

* <a href="#intro">Introduction</a>
* <a href="#using">Using</a>
  * <a href="#understanding">Understanding rule and rule.each</a>
* <a href="#ops">Operations</a>
  * <a href="#op-basics">The Basics</a>
  * <a href="#op-writing">Writing Your Own</a>
* <a href="#utilities">Utilities</a>
  * <a href="#mapping">Mapping Functions</a>

<a name="intro"></a>
Introduction
------------

Fez is an über fast general purpose build tool based on [tup][2] and engineered for Javascript. Build specs are defined as sets of transformational relationships between files. This set of rules is used to construct a build graph which is efficiently traversable, enabling Fez to do only the work which needs to be done, and to do work in parallel where appropriate.

Let's look at the following ruleset:

    *.less → %f.css
    *.css → %f.min.css
    *.min.css → dist.min.css

And a few input files on our file system:

![](https://dl.dropboxusercontent.com/u/10832827/before.svg)

Fez takes  the ruleset and  the input files and  generates a
build graph:

![](https://dl.dropboxusercontent.com/u/10832827/after.svg)

Generating the build graph like this does a few things. First, it makes sure
everything that *needs* to be built *is* built. Less intelligent build systems
may skip over necessary files, or offer less flexibility in rule definitions to
avoid potential problems. Second, the build graph speeds things up. Rather than
making multiple passes to make sure every file is handled, Fez works its way up
(with a topological sort) from the source nodes of the graph (files which
already exist on the file system) to the sink nodes (files which have no further
outputs). Third, it gives us an opportunity to introduce concurrency (see the
roadmap below) and process appropriate transformations in separate processes.

<a name="using"></a>
Using
-----

Fez build specs are simple Javascript files, usually called `fez.js` in a
project's root directory. There is no command line Fez tool; instead, Fez build
specs are self-executable.  The basic outline of a `fez.js` file usually
something like this:

{% highlight javascript %}
var fez = require("fez");

exports.build = function(rule) {
  ...    		  
};

exports.default = exports.build;

fez(module);
{% endhighlight %}

Let's step through this line by line.

{% highlight javascript %}
var fez = require("fez");
{% endhighlight %}

Requiring `fez` isn't strictly necessary, but is needed to make `fez.js`
self-executable. The only case in which you might not want it is if you have
child build specs which are never run on their own. Even then, `fez` comes with
a number of utility functions useful for crafting rulesets.

{% highlight javascript %}
exports.build = function(rule) {
{% endhighlight %}

Rulesets (a variation on tasks, or targets) are specified by adding fields to
`module.exports`.  A ruleset takes the form of a function which uses its
argument, `rule`, to define a set of rules to be used in generating the build
graph. We'll talk more about crafting rulesets later on. For now, just know that
every ruleset is a function, and is named.

{% highlight javascript %}
exports.default = exports.build;
{% endhighlight %}

The ruleset named `default` has special meaning in Fez. When `fez.js` is run
without any command line options (i.e `node fez.js`), the `default` ruleset is
run.  We specify the `default` ruleset just like any other: by adding a field
named `default` to `module.exports`.

{% highlight javascript %}
fez(module);
{% endhighlight %}

This is the tiny bit of magic that makes `fez.js` self-executable.  The `fez`
function takes a module (almost always the current module) as an argument. If
that module is the *main* module (i.e the file which was run with `node
fez.js`), Fez will parse command line options and run the builds generated from
the rulesets in the module. This line should be in every build spec unless you
are sure you will never want to run the build spec on its own. It i→s safe (and
recommended) to include the line even in build specs which will be used
primarily as child specs. Using child specs will be discussed in more detail
below.

<a name="understanding"></a>
### Understanding `rule` and `rule.each`

Rulesets  are  defined  with  a   combination  of  the  `rule`  and  `rule.each`
functions.  They  each   have  semantic  differences  which   are  important  to
understand. Both functions have the same basic signature:

{% highlight javascript %}
rule(inputs, output, operation)
rule.each(input, output, operation)
{% endhighlight %}

The `rule` function is used to define one to one and many to one
relationships. The `inputs` argument is in the form of a string, which can be a
glob, or an array of strings, which can be globs. All of the globs are expanded
in the build graph and the complete set of inputs is passed to the
operation. For example, imagine the following rule:

    *.js → dist.min.js

and the files:

    a.js
    b.js
    c.js

The build graph would end up looking this like this:

    a.js
         ↘
    b.js  ⇒ dist.min.js
         ↗
    c.js

Each of the inputs is a node, and all the nodes share a common ouput.

The `rule.each` function is a little different.  It exclusively defines one to
one relationships. It is used in cases where the inputs are unknown (i.e a glob)
and the output is a function of the selected input. It's easiest to explain with
an example. Let's use a similar rule as above:

    *.js → %f.min.js

and the files:

    a.js
    b.js
    c.js

Notice how rather than having a single output name in our rule, the output is
based on the input.  The `%f` operator is replaced with the input's filename
minus its extension. The `rule.each` function *must* have a function as the
output argument. Fez has a handful of utility function generators for building
the output function. Perhaps the most useful is `fez.mapFile`. There is also
`fez.patsubst` which is a lot like the [make][1] function by the same name. Read
below for more details on Fez's utilities.

The rule and files above  will result in the following build
graph:

    a.js → a.min.js
    b.js → b.min.js
    c.js → c.min.js

Unlike `rule`, `rule.each` produces a unique output for each input.  Cool! With
`rule` and `rule.each`, we can define almost any transformational relationship.

The only thing missing is one to many and many to many relationships. For now,
that seems like a pretty uncommon use case.  If you do need to define a
relationship like that, it would be great for you to submit your use case in a
bug report!.

[1]: https://www.gnu.org/software/make/
[2]: http://gittup.org/tup/
