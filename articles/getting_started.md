---
title: "Getting Started with Ogre, a graph query library"
layout: article
---


## Getting Started

[Ogre](http://github.com/clojurewerkz/ogre) is a Clojure wrapper for the
[Gremlin](https://tinkerpop.apache.org/gremlin.html)  graph traversal language of 
[Apache TinkerPop&trade;](https://tinkerpop.apache.org). This documentation assumes that
its audience has an existing knowledge of TinkerPop concepts and the Gremlin language syntax. 

<a href="http://tinkerpop.apache.org/gremlin.html">
  <img src="https://raw.githubusercontent.com/apache/tinkerpop/master/docs/static/images/gremlin-running.png" style="width: 120px" alt="Gremlin"/>
</a>

### Installing

Orge artifacts are [released to Clojars](https://clojars.org/clojurewerkz/ogre). Maven users should add the 
following repository definition to your pom.xml:

```xml
<repository>
  <id>clojars.org</id>
  <url>http://clojars.org/repo</url>
</repository> 
```

With Leiningen:

```clojure
[clojurewerkz/ogre "3.2.4.0"]
```

With Maven:

```xml
<dependency>
  <groupId>clojurewerkz</groupId>
  <artifactId>ogre</artifactId>
  <version>3.2.4.0</version>
</dependency>
```

### Introduction to Ogre

To get an idea of how Ogre helps make Gremlin easier to work with in 
Clojure, let's convert a basic Gremlin traversal into an Ogre traversal.
Consider the following Gremlin traversal written in Java:

```groovy
g.V().has("name","gremlin").         // Get the vertex with the name "gremlin"
  out("knows").                      // Traverse to people who Gremlin knows
  out("knows").                      // Traverse to the people those people know
  values("name")                     // Get the names of those people
```
  
In Ogre this traversal is written as:

```clojure
(traverse g V (has :name "gremlin")  ;; Get the vertex with the name "gremlin"
            (out :knows)             ;; Traverse to people who Gremlin knows
            (out :knows)             ;; Traverse to the people those people know
            (values :name))          ;; Get the names of those people
```

The flow of Ogre maps quite naturally to that of Gremlin written in Java and 
this approach is quite typical of Ogre in most cases though there are a few 
areas of minor step naming differences to be better in line with Clojure.

### The REPL

Gremlin is commonly used in the [Gremlin Console](http://tinkerpop.apache.org/docs/current/tutorials/the-gremlin-console/)
which is a Groovy-based REPL. Gremlin is quite at home in that style of environment 
so this documentation will be make heavy use of the Clojure REPL to explain Ogre
usage.

```text
user=> (load "clojurewerkz/ogre/core") 
nil
user=> (in-ns 'clojurewerkz.ogre.core)
#object[clojure.lang.Namespace 0x228e9715 "clojurewerkz.ogre.core"]
clojurewerkz.ogre.core=> (import '[org.apache.tinkerpop.gremlin.tinkergraph.structure TinkerGraph TinkerFactory])
org.apache.tinkerpop.gremlin.tinkergraph.structure.TinkerGraph
```

Note that [TinkerGraph](http://tinkerpop.apache.org/docs/current/reference/#tinkergraph-gremlin)
was imported as a sample graph for purpose of this documentation, but obviously any TinkerPop-enabled
graph would suffice.

Unless otherwise noted, the code in the following sections will assume that the
REPL was initialized as shown above. 

### Getting a Graph Instance

Getting a `Graph` instance is the first step to working with Gremlin and 
therefore is the first step to working with Ogre. Ogre provides a wrapper around
TinkerPop's `GraphFactory`, which provides a provider-agnostic way to create 
graphs. The following code will create a TinkerGraph instance with Ogre:

```text
clojurewerkz.ogre.core=> (def graph (open-graph {(Graph/GRAPH) (.getName TinkerGraph)}))
#'clojurewerkz.ogre.core/graph
```

In the above example `open-graph` takes a `Map` of arguments which is the configuration
object that `GraphFactory` accepts in its `openGraph()` method. Obviously, it would also
be possible to simply instantiate the TinkerGraph directly as follows:

```text
clojurewerkz.ogre.core=> (def graph (TinkerGraph/open))
#'clojurewerkz.ogre.core/graph
```

While the second method is more straightforward it is specific to TinkerGraph. Not all
`Graph` instances will be instantiated that way. Unless otherwise noted, this documentation 
will focus on traversals that work with the [modern graph](http://tinkerpop.apache.org/docs/current/tutorials/the-gremlin-console/#toy-graphs)
which is a small sample dataset that is provided by TinkerPop. This data can be loaded 
into the `graph` instances as follows:

```text
clojurewerkz.ogre.core=> (TinkerFactory/generateModern graph)
nil
clojurewerkz.ogre.core=> graph
#object[org.apache.tinkerpop.gremlin.tinkergraph.structure.TinkerGraph 0x6e362190 "tinkergraph[vertices:6 edges:6]"]
```

For purpose of this documentation and experimentation all of the above could be simplified
to:

```text
clojurewerkz.ogre.core=> (def graph (TinkerFactory/createModern))    ;; graph = TinkerFactory.createModern()
#'clojurewerkz.ogre.core/graph
clojurewerkz.ogre.core=> graph
#object[org.apache.tinkerpop.gremlin.tinkergraph.structure.TinkerGraph 0x75eeb446 "tinkergraph[vertices:6 edges:6]"]
```

### The Traversal

With a `Graph` instance and some data in place it is now possible to execute some traversals. 
As with Gremlin, Ogre needs a `TraversalSource` object. A basic `TraversalSource` can be 
constructed with:

```text
clojurewerkz.ogre.core=> (def g (traversal graph))    ;; graph.traversal()
#'clojurewerkz.ogre.core/g
```

The familiar `g` becomes the cornerstone of traversing that Gremlin users are familiar with. Consider
the following Gremlin Java based traversal:

```java
g.V().values("name")
```

The above traversal can be written in Ogre as:

```text
clojurewerkz.ogre.core=> (traverse g V (values "name") (into-seq!))
("marko" "vadas" "lop" "josh" "ripple" "peter")
clojurewerkz.ogre.core=> (traverse g V (values :name) (into-seq!))
("marko" "vadas" "lop" "josh" "ripple" "peter")
```

The `traverse` function takes a `TraversalSource` and the sequence of traversal steps that make
up the traversal. There are two important things to note. First, property keys can be specified 
as a string, as in `"name"` or as a keyword `:name`. The latter can be a bit nicer in Clojure.
Second, the statement is concluded with a terminator, in this case `(into-seq!)` which iterates
the traversal into a Clojure sequence. There are a number of such terminator functions available
including:

* `into-seq!`
* `into-list!`
* `into-vec!`
* `into-set!`
* `iterate!`
* `next!`

The point with these terminators is that just like Gremlin in any other language, Ogre does 
not iterate a `Traversal` for you. It is an `Iterator` ultimately and it is up to the developer
to decide how it should be treated.

The previous example showed that Ogre can use keywords for property keys, but they can be 
also be used in other areas as well:

```text
clojurewerkz.ogre.core=> (traverse g V (has :name "marko") (out :created) (values :name) (into-seq!))
("lop")
clojurewerkz.ogre.core=> (traverse g V (has :name "marko") (as :a) (out :knows) (as :b) (select :a :b) (by :name) (into-seq!))
({"a" "marko", "b" "vadas"} {"a" "marko", "b" "josh"})
```

Gremlin has a number of [steps](http://tinkerpop.apache.org/docs/current/reference/#graph-traversal-steps) 
all of which are supported by Ogre. As the documentation of Ogre grows it may come to cover all of those 
steps, but for now it will be best to look at the Ogre [unit tests](https://github.com/clojurewerkz/ogre/tree/master/test/clojure/clojurewerkz/ogre/suite)
when syntax isn't clear. As Ogre implements the TinkerPop Process Test Suite, it will always have a complete
body of examples to examine.

### Versioning

The version scheme for Ogre is as follows:
`[Full TinkerPop version].[Ogre major version]`. Philosophically, releases of
TinkerPop will typically trigger releases in Ogre. Ogre is a mere wrapper 
around that body of work and the version scheme acknowledges that dependency 
directly. Thus, an Ogre release of `3.2.4.2`, would mean that the current 
release uses Gremlin `3.2.4` and has undergone two major versions itself so far 
since release.
