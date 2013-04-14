---
title: "Filters in Ogre, a graph query library"
layout: article
---

## Filter

Here be functions which filter out objects.

### range

A range filter that emits the objects within a range.

``` clojure 
(q/query (g/find-by-id 1)
         (q/-->)
         (q/into-vec!))
;;[#<TinkerVertex v[2]> #<TinkerVertex v[4]> #<TinkerVertex v[3]>]         


(q/query (g/find-by-id 1)
         (q/-->)
         (q/range 0 1)
         (q/into-vec!))
;;[#<TinkerVertex v[2]> #<TinkerVertex v[4]>]         
```

### dedup

Filter out repeated objects. A function can be supplied that provides
the values that the pipeline will consider when filtering 

``` clojure
(q/query (g/get-vertices)
         q/<->
         (q/property :name)
         (q/into-vec!))         
;;["marko" "josh" "peter" "marko" "vadas" "josh" "lop" "lop" "josh" "marko" "ripple" "lop"]

(q/query (g/get-vertices)
         q/<->
         q/dedup
         (q/property :name)
         (q/into-vec!))
;;["marko" "josh" "peter" "vadas" "lop" "ripple"]                         

(q/query (g/get-vertices)
          q/<->
          (q/dedup (partial g/get-property :lang))
          (q/property :name)
          (q/into-vec!))
;;["marko" "lop"]          
```                         

Note that `g/get-vertices` retrieves all of the vertices of the graph
and provides them in a list.

### except

Filter out the provided objects.

``` clojure
(q/query (g/find-by-id 1)
         q/-->
         q/<--
         (q/except [(g/find-by-id 1)])
         (q/into-vec!))         
;;[#<TinkerVertex v[4]> #<TinkerVertex v[6]>]         
```                         

### filter

Uses a predicate to decide whether an object should pass.


``` clojure
(q/query (g/get-vertices)
         (q/filter (fn [v] (= "java" (.getProperty v "lang"))))
         q/map
         (q/all-into-maps!))         
;;({:name "lop", :lang "java"} {:name "ripple", :lang "java"})         
```                         


### has

Allows an element if it has a particular property. The standard
Clojure operations for comparisons can also be supplied:
`>`,`>=`,`<`,`<=`,`=`,`not=`. 

```clojure
(q/query (g/get-vertices)
         (q/has :name "marko")                    
         (q/into-vec!))
;;[#<TinkerVertex v[1]>]         

(q/query (g/get-vertices)
         (q/has :age > (int 30))                    
         (q/into-vec!))
[#<TinkerVertex v[6]> #<TinkerVertex v[4]>]         
```                      

### has-not

Allows an element if it does not have a particular property. 

```clojure
(q/query (g/get-vertices)
         (q/has-not :name "marko")                    
         (q/into-vec!))
;;[#<TinkerVertex v[3]> #<TinkerVertex v[2]> 
;; #<TinkerVertex v[6]> #<TinkerVertex v[5]> 
;; #<TinkerVertex v[4]>]

(q/query (g/get-vertices)
         (q/has-not :age > (int 30))                    
         (q/into-vec!))
;;[#<TinkerVertex v[2]> #<TinkerVertex v[1]>]         
```                      

### interval

Allow elements to pass that have their property in the provided start
and end interval.

```clojure
(q/query (g/find-by-id 1)
         (q/--E>)
         (q/interval :weight 0 0.6)
         (q/in-vertex)
         (q/into-vec!))
;;[#<TinkerVertex v[2]> #<TinkerVertex v[3]>]         
```         
### random

Emits the incoming objects, each with the supplied chance.

```clojure
;; Results may vary
(q/query (g/get-vertices)
         (q/random 0.5)
         (q/into-vec!))
[#<TinkerVertex v[6]> #<TinkerVertex v[4]>]

(q/query (g/get-vertices)
         (q/random 0.5)
         (q/into-vec!))
[#<TinkerVertex v[3]> #<TinkerVertex v[1]> #<TinkerVertex v[6]> #<TinkerVertex v[4]>]
```
### retain

Only allows elements in the supplied collection to pass.

```clojure
(q/query (g/find-by-id 1)
         (q/-->)      
         (q/retain [(g/find-by-id 2)])
         (q/into-vec!))
;;[#<TinkerVertex v[2]>]         
```

### Annotating pipes is next

You should [read about annotating pipes next](/articles/annotation.html). 

