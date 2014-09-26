---
title: "Getting Started with Ogre, a graph query library"
layout: article
---


## Getting Started 

### Introduction to Ogre

[Ogre](http://github.com/clojurewerkz/ogre) is a domain specific
language for traversing property graphs in
[Clojure](http://clojure.org/). Ogre wraps
[Gremlin](https://github.com/tinkerpop/gremlin/wiki), a library which
enables
[all sorts of groovy ways to work with graphs](http://gremlindocs.com/).
The documentation and samples presented here attempt to stay current
with the most current, stable release of Ogre. Please join the
[Titanium users group](https://groups.google.com/forum/#!forum/clojure-titanium)
for Ogre related discussions. Please use the
[Ogre issue page](https://github.com/clojurewerkz/ogre/issues) for
reporting bugs and discussing features. For any errors or corrections
with this documentation, please use the
[issue page](https://github.com/clojurewerkz/ogre.docs). Pull requests
will be celebrated, scrutinized, and hopefully accepted. Ogre
currently powers
[Archimedes](https://github.com/clojurewerkz/archimedes), a Clojure
library for
[Blueprints](https://github.com/tinkerpop/blueprints/wiki), and
[Titanium](https://github.com/clojurewerkz/titanium), a Clojure
library built on top of Archimedes for working with
[Titan](http://thinkaurelius.github.com/titan/).

### The Tinkerpop stack

Before going down the rabbit hole, we offer the briefest of warnings:
the [Tinkerpop folks](https://github.com/tinkerpop?tab=members) have
been working on Gremlin,
[Pipes](https://github.com/tinkerpop/pipes/wiki), and
[Blueprints](https://github.com/tinkerpop/blueprints/wiki) for a few
years now and the stack has become incredibly intertwined. Ogre and
[Archimedes](https://github.com/clojurewerkz/archimedes) try to hide
all of this from you at some hide level, but every abstraction leaks.
Your stack traces will speak of `com.tinkerpop.blueprints.Vertex` and
`com.tinkerpop.gremlin.GremlinPipeline`, and there is nothing we can
(well, should) do about it. The Tinkerpop stack is fantastic and
really well done, but we wanted to warn you that the following is just
the tip of the iceberg in terms of what you will need to know to
understand what is really going on when you use Ogre.

Thankfully, it's a joy to work with the stack. So, let's get started!

### leiningen

The version scheme for Ogre is as follows:
`[Full Gremlin version].[Ogre major version]`. Philosophically, the
authors of Gremlin are the ones who will be causing most of the
changes to Ogre to hapepn. Ogre is a mere wrapper around their work
and the version scheme acknowledges that directly. Thus, the current
release is `2.3.0.2`, meaning that the current release uses Gremlin
`2.3.0` and has undergone two major versions itself so far since
release.

To get started with Ogre, include the following dependency for
leiningen: `[clojurewerkz/ogre "2.3.0.2"]`.

### The TinkerGraph

Unless otherwise noted, all samples reference `clojurewerkz.ogre.tinkergraph` and
`clojurewerkz.ogre.core` as follows:

```clojure 
(require [clojurewerkz.ogre.tinkergraph :as g]) 
(require [clojurewerkz.ogre.core :as q]) 

(g/use-new-tinker-graph!)
```

`g/use-new-tinker-graph!` creates the following graph and secretly
squirrels it away
[_somewhere_](https://github.com/clojurewerkz/ogre/blob/master/src/ogre/tinkergraph.clj#L10)
(image from
[here](http://github.com/tinkerpop/blueprints/wiki/Property-Graph-Model)):

<img src="https://github.com/tinkerpop/blueprints/raw/master/doc/images/graph-example-1.jpg"></img>

We recommend that you open this image up into a
[new tab](https://github.com/tinkerpop/blueprints/raw/master/doc/images/graph-example-1.jpg).
It will serve as the main reference for the majority of the examples below. 

### So what does Ogre actually do? 

At a high level, Ogre let's you easily ask complex questions about
certain types of graphs and get back answers. That's it.

At a low level, Ogre is a library that takes in Blueprint Vertices and
Edges and let's you build up GremlinPipeline objects that ask
questions about those objects in the language of traversals,
transformations, filters, and branching on the graph. Ogre allows you
to annotate various steps of the pipeline to allow for incredibly
useful queries in a few terse lines. Ogre also carefully deals with some of
the side effects that the Gremlin library can launch. 

At the lowest level, Ogre is probably equivalent to some crazy Turing
machine. Some poor grad student has probably had to try and write the
JVM as a Turing machine. Poor gal.

### Reading this documentation

This series of guide is organized to be read mostly linearly. That
means that you can probably read it from start to finish and
understand what is going on. We start from the basics, with
traversals, transformations, query executions, and filters. Then we
transition into the more advanced topics of annotations and side
effects. At the same time, this documentation is meant to serve as a
reference for anyone using the library. These examples were are
developed at the command line or inside emacs with a REPL, so they are
meant to be run and experimented with.


### Building queries 

Ogre let's you build up Gremlin queries from scratch. The main method
for doing this is `q/query`. Here is a simple query on the
Tinkergraph. It takes in the vertex with id `1`, finds the vertices
that the starting vertex points out to, and then returns the result in
a vector.

``` clojure
(require [clojurewerkz.ogre.tinkergraph :as g]) 
(require [clojurewerkz.ogre.core :as q]) 

(g/use-new-tinker-graph!)

(q/query (g/find-by-id 1)
         q/-->
         q/into-vec!)
;= [#<TinkerVertex v[2]> #<TinkerVertex v[4]> #<TinkerVertex v[3]>]
```

Let's break this down: 

* `q/query` is a
  [simple macro combining](https://github.com/clojurewerkz/ogre/blob/master/src/ogre/util.clj#L13)
  of `->` and `(GremlinPipeline.)`. It takes in a single element or a
  Collection and creates a new pipeline around them.
* `g/find-by-id` is a function that goes and asks the vertex for the
  element of id 1. 
* `q/-->` is a function which adds on an outwards traversal step to
  the pipe. This means that the Gremlin query will take all the
  vertices it is currently thinking about and then think about all the
  vertices that the vertices in it's mind pointed to.
* `q/into-vec!` executes the query and returns the results inside of a
  vector. Before this call, the Gremlin query hasn't actually done
  anything yet. Only when a function that ends with a bang is passed
  in does anything actually happen beyond just a GremlinPipeline
  getting built up. 

So far so good. But, I wonder, who is the dashing rogue behind
`#<TinkerVertex v[2]>`?

``` clojure
(q/query (g/find-by-id 1)
         q/-->
         q/into-vec!
         first
         ((q/prop :name)))
;= "vadas"
```

`q/query` isn't just about running Gremlin queries. Remember, it's
really just a glorified `->`, and a bunch of provided helper
functions. That means we can stick a `first` in there to get the first
vertex of the vector. `(q/prop :name)` takes a property key and
returns a function which takes a vertex and returns the given
property. Thus, `"vadas"` is the charming face of `#<TinkerVertex
v[2]>`.

### Traversals are next

You should [read about traversals next](/articles/traversals.html). 
