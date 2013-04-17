---
title: "Annotations in Ogre, a graph query library"
layout: article
---

## Annotations

So far, we've had great success traversing the graph. If you've
understood most of everything up to this point, you know how to do
sorts of neat things with graphs now. There's another level of traversal
that we can attain though. We can annotate and traverse the pipeline
itself, which lets us do all sorts of fancy tricks. 

### back

Go back to the results from n-steps ago.

```
(q/query (g/find-by-id 1)
         q/-->      
         (q/back 1) 
         q/into-vec!)
;;[#<TinkerVertex v[1]>]
```

### as/back-to

`as` lets you name a step that you can later return to with `back-to`. 

```
(q/query (g/find-by-id 1)
         (q/as "here")
         q/-->      
         (q/back-to "here")         
         q/into-vec!)
;;[#<TinkerVertex v[1]>]         
```


### select

Get a list of named steps, with optional functions for post processing
round robin style. (This will be returned as a map when Pipes is
upgraded to 2.3.0).

```clojure
(q/query (g/find-by-id 1)
         (q/as "a")
         (q/--> :knows)
         (q/as "b")
         q/select
         q/all-into-vecs!)
;;([#<TinkerVertex v[1]> #<TinkerVertex v[2]>] 
;; [#<TinkerVertex v[1]> #<TinkerVertex v[4]>])         

(q/query (g/find-by-id 1)
         (q/as "a")
         (q/--> :knows)
         (q/as "b")
         (q/select (q/prop :name))
         q/all-into-vecs!)
;;(["marko" "vadas"] ["marko" "josh"])

(q/query (g/find-by-id 1)
         (q/as "a")
         (q/--> :knows)
         (q/as "b")
         (q/select (q/prop :name) g/get-id)
         q/all-into-vecs!)
;;(["marko" "2"] ["marko" "4"])
```

### select-only

Select the named steps to emit, with round robin style function
processing again. 

```clojure
(q/query (g/find-by-id 1)
         (q/as "a")
         q/-->
         (q/as "b")
         q/-->
         (q/as "c")       
         (q/select-only ["a" "b"])
         q/all-into-vecs!)
;;([#<TinkerVertex v[1]> #<TinkerVertex v[4]>] 
;; [#<TinkerVertex v[1]> #<TinkerVertex v[4]>])

(q/query (g/find-by-id 1)
         (q/as "a")
         q/-->
         (q/as "b")
         q/-->
         (q/as "c")       
         (q/select-only ["a" "c"] (q/prop :name))
         q/all-into-vecs!)
;;(["marko" "ripple"] ["marko" "lop"])

(q/query (g/find-by-id 1)
         (q/as "a")
         (q/--> :knows)
         (q/as "b")
         q/-->
         (q/as "c")                
         (q/select-only ["a" "c"] (q/prop :name) g/get-id)
         q/all-into-vecs!)
;;(["marko" "5"] ["marko" "3"])
```

### loop

Loop over a particular set of steps in the pipeline. The first
argument is the number of steps back. The second argument is a
predicate that takes three objects: the current object, the current
path, and the number of loops thus far. While the predicate evaluates
true, the loop continues on it's merry way. 

```clojure
(q/query (g/find-by-id 1)
         (q/-->)
         (q/loop 1
                 (fn [l o p] (< l 3)))
         (q/property :name)
         (q/into-vec!))                           
;;["ripple" "lop"]
```                  
                
### loop-to

`loop-to` is just like loop, but it travels back to a named step
instead. 

```clojure
(q/query (g/find-by-id 1)
         (q/as "here")
         (q/-->)
         (q/loop-to "here"
                 (fn [l o p] (< l 3)))
         (q/property :name)
         (q/into-vec!))                           
;;["ripple" "lop"]
```                  

*** 

### Side effects are next

You should [read about side effects next](/articles/sideeffects.html). 

