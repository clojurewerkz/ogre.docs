---
title: "Map like functions in Ogre, a graph query library"
layout: article
---

## Map like functions

The following functions are conceptually similar in scope to
`clojure.core/map` and so are grouped together. They all take in some
object and perform a transformation on it.

### id

Gets the unique identifier of the element.

``` clojure
(q/query (g/find-by-id 1)
         q/id
         q/into-vec!)
;;["1"]

(q/query (g/find-by-id 1)
         q/-->
         q/id
         q/into-vec!)
;;["2" "4" "3"]

(q/query (g/find-by-id 1)
         q/id
         q/-->         
         q/into-vec!)
;;ClassCastException java.lang.String cannot be cast to  com.tinkerpop.blueprints.Vertex  
;;com.tinkerpop.gremlin.pipes.transform.VerticesVerticesPipe.processNextStart 
;;(VerticesVerticesPipe.java:37)
```

### property

Get the property value of an element. 

``` clojure
(q/query (g/find-by-id 1)
         (q/property :name)
         q/into-vec!)
;;["marko"]

(q/query (g/find-by-id 1)
         q/-->
         (q/property :name)
         q/into-vec!)
;;["vadas" "josh" "lop"]
```


### label

Get the label of an edge.

``` clojure
(q/query (g/find-by-id 1)
         q/--E>
         q/label
         q/into-vec!)
;;["knows" "knows" "created"]
``` 

### map

Get the property map of the graph element.

``` clojure
(q/query (g/find-by-id 1)
         q/map
         q/into-vec!)
;;[#<HashMap {name=marko, age=29}>]

(q/query (g/find-by-id 1)
         q/map
         q/first-into-map!)
;;{:name "marko", :age 29}

(q/query (g/find-by-id 1)
         q/-->
         q/map
         q/all-into-maps!)
;;({:name "vadas", :age 27} {:name "josh", :age 32} {:name "lop", :lang "java"})
``` 

We now see two new functions in addition to `q/map`: `first-into-map!`
and `all-into-maps!`. As you see in the first example, Gremlin doesn't
return Clojure data structures. The new functions execute the Gremlin
query and then call the correct conversion methods to ensure that you
can work with the returned objects without too much hassle.

### path

Gets the path through the pipeline up to this point. If functions are
provided, they are applied round robin to each of the objects in the
path. 

``` clojure
(q/query (g/find-by-id 1)
         q/<->
         q/<->
         q/path
         q/all-into-vecs!)
;; ([#<TinkerVertex v[1]> #<TinkerVertex v[2]> #<TinkerVertex v[1]>] 
;;  [#<TinkerVertex v[1]> #<TinkerVertex v[4]> #<TinkerVertex v[1]>] 
;;  [#<TinkerVertex v[1]> #<TinkerVertex v[4]> #<TinkerVertex v[5]>] 
;;  [#<TinkerVertex v[1]> #<TinkerVertex v[4]> #<TinkerVertex v[3]>] 
;;  [#<TinkerVertex v[1]> #<TinkerVertex v[3]> #<TinkerVertex v[1]>] 
;;  [#<TinkerVertex v[1]> #<TinkerVertex v[3]> #<TinkerVertex v[4]>] 
;;  [#<TinkerVertex v[1]> #<TinkerVertex v[3]> #<TinkerVertex v[6]>])         

(q/query (g/find-by-id 1)
         q/<->
         q/<->
         (q/path (q/prop :name))
         q/all-into-vecs!)
;;(["marko" "vadas" "marko"] 
;; ["marko" "josh" "marko"] 
;; ["marko" "josh" "ripple"] 
;; ["marko" "josh" "lop"] 
;; ["marko" "lop" "marko"] 
;; ["marko" "lop" "josh"] 
;; ["marko" "lop" "peter"])

(q/query (g/find-by-id 1)
         q/<->
         q/<->
         (q/path (q/prop :name) (fn [v] (count (.getProperty v "name"))))
         q/all-into-vecs!)
;;(["marko" 5 "marko"] 
;; ["marko" 4 "marko"] 
;; ["marko" 4 "ripple"] 
;; ["marko" 4 "lop"] 
;; ["marko" 3 "marko"] 
;; ["marko" 3 "josh"] 
;; ["marko" 3 "peter"])         
``` 

Note that again we have introduced a new function `all-into-vecs!`.
This function takes in an ArrayList of ArrayLists and produces a list
of vectors.

### transform

Transform applies a function to each object. 

``` clojure
(q/query (g/find-by-id 1)
         (q/transform (q/prop :name))
         q/first-of!)
;;"marko"         

(q/query (g/find-by-id 1)
          q/--E>
          q/label
          (q/transform count)
          q/into-vec!)         
;;[5 5 7]          
``` 

`first-of!` executes the query and gets the first element from the
list. Don't shoot yourself in the foot with this. 

### Query execution is next

You should [read about the various ways to execute queries next](/articles/query.html). 
