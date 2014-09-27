---
title: "Executing queries in Ogre, a graph query library"
layout: article
---

## Query execution 

Ogre cannot do everything for you. Specifically, it does not figure
out the types of Java objects that are returned from some arbitrary
query and convert them automatically into Clojure objects. So, with
that in mind, Ogre includes several functions that execute the
pipeline and then do conversions into specific Clojure data
structures.

### count!

`q/count!` returns the number of objects currently in the pipeline. 

``` clojure
(q/query (g/find-by-id 1)
         q/-->
         q/count!)
;= 3
``` 

### into-lazy-seq!

Gets the objects and returns them inside a lazy sequence.
``` clojure
(type (q/query (g/find-by-id 1)
         q/-->
         q/into-lazy-seq!))
;= clojure.lang.Cons

(first (q/query (g/find-by-id 1)
         q/-->
         q/into-lazy-seq!))
;= #<TinkerVertex v[2]>
```

### into-list!

Gets the objects and sticks them inside of a list. 

``` clojure
(q/query (g/find-by-id 1)
         q/-->
         q/into-list!)
;= (#<TinkerVertex v[3]> #<TinkerVertex v[4]> #<TinkerVertex v[2]>)
``` 

### into-vec!

Gets the objects and sticks them inside of a vector. 

``` clojure
(q/query (g/find-by-id 1)
         q/-->
         q/into-vec!)
;= [#<TinkerVertex v[2]> #<TinkerVertex v[4]> #<TinkerVertex v[3]>]
``` 

### into-set!

Gets the objects and sticks them inside of a set. 

``` clojure
(q/query (g/find-by-id 1)
         q/-->         
         q/into-set!)
;= #{#<TinkerVertex v[2]> #<TinkerVertex v[3]> #<TinkerVertex v[4]>}
``` 

### first-of!

Gets the first object of the returned list. 

``` clojure
(q/query (g/find-by-id 1)
         q/first-of!)
;= #<TinkerVertex v[1]>
``` 

### first-into-vec!

Gets the first object of the returned list and puts it into a vector. 

``` clojure
(q/query (g/find-by-id 1)
         (q/property :name)
         q/path
         q/into-vec!)
;= [#<ArrayList [v[1], marko]>]         

(q/query (g/find-by-id 1)
         (q/property :name)
         q/path
         q/first-into-vec!)
;= [#<TinkerVertex v[1]> "marko"]
``` 

### first-into-set!

Gets the first object of the returned list and puts it into a set. 

``` clojure
(q/query (g/find-by-id 1)
         q/-->
         q/id
         q/gather
         q/first-into-set!)
;= #{"2" "3" "4"}         
```

`gather` collects all objects up to that step; see
[reduce-like functions](/articles/reduce.html) for more examples.

### first-into-map!

Gets the first object of the returned list and puts it into a set. 

``` clojure
(q/query (g/find-by-id 1)
          q/map
          q/first-into-map!)
;= {:name "marko", :age 29}
```

### all-into-vecs!

Gets the list of returned objects and maps vec across all of the
objects.

``` clojure
(q/query (g/find-by-id 1)
         q/-->
         (q/path (q/prop :age)
                 (q/prop :name))
         q/all-into-vecs!)
;= ([29 "vadas"] [29 "josh"] [29 "lop"])
```                        

### all-into-sets!

Gets the list of returned objects and maps set across all of the
objects.

``` clojure
(q/query (g/find-by-id 1)
         q/-->
         (q/path (q/prop :age)
                 (q/prop :name))
         q/all-into-sets!)
;= (#{"vadas" 29} #{"josh" 29} #{"lop" 29})
```                        

### all-into-maps!

Gets the list of returned objects and maps set across all of the
objects.

``` clojure
(q/query (g/find-by-id 1)
         q/<->
         q/<->
         q/map
         q/all-into-maps!)
;= ({:name "marko", :age 29} 
;=  {:name "marko", :age 29} 
;=  {:name "ripple", :lang "java"} 
;=  {:name "lop", :lang "java"} 
;=  {:name "marko", :age 29} 
;=  {:name "josh", :age 32} 
;=  {:name "peter", :age 35})         
```                        

### Reduce-like functions are next

You should [read about reduce-like functions next](/articles/reduce.html). 
