---
layout: default
title: Fez
---

Fez is an über fast general purpose build tool based on [tup][1] and engineered for Javascript. Build specs are defined as sets of transformational relationships between files. This set of rules is used to construct a build graph which is efficiently traversable, enabling Fez to do only the work which needs to be done, and to do work in parallel where appropriate.

<ul class="overview-list">
  <li>Fast</li>
  <li>Powerful</li>
  <li>Succinct</li>
  <li>Cool</li>
</ul>

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
[roadmap](/contribute.html#roadmap)) and process appropriate transformations in separate processes.

[1]: http://gittup.org/tup/