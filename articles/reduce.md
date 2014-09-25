---
title: "Executing queries in Ogre, a graph query library"
layout: article
---

## Reduce like functions

These functions sort of act like `clojure.core/reduce`.

### order

Order the items in the stream according to the provided function. If no
function is provided, then a default sort order is used.

```clojure
(q/query (g/get-vertices)
         (q/property :name)
         q/into-vec!)                         
;= ["lop" "vadas" "marko" "peter" "ripple" "josh"]

(q/query (g/get-vertices)
         (q/property :name)
         q/order
         q/into-vec!)                         
;= ["josh" "lop" "marko" "peter" "ripple" "vadas"]         

(q/query (g/get-vertices)
         (q/property :name)
         (q/order (fn [a b] (compare b a)))
         q/into-vec!)                         
;= ["vadas" "ripple" "peter" "marko" "lop" "josh"]
```

### gather

Collect all objects up to that step and, optionally, process the
gathered list with the provided function.

``` clojure
(q/query (g/find-by-id 1)
         q/-->
         q/id
         q/gather
         q/first-into-vec!)
;= ["2" "3" "4"]         

(q/query (g/find-by-id 1)
         q/-->
         q/id
         (q/gather count)
         q/into-vec!)
;= [3]
```

### Filters are next

You should [read about filters next](/articles/filter.html). 
