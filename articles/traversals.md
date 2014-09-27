---
title: "Traversals via Ogre, a graph query library"
layout: article
---

## Traversals

Traversal functions allow you to explore around the graph and see how
vertices are connected. 

### out / -->

`-->` or `out` gets the out adjacent vertices (the functions do
exactly the same thing, one just looks cooler). Additionally, a list
of labels can be supplied that so that the query only traverses edges
with labels in the provided collection. This also applies to the other
traversal functions where it makes sense (any function that is named
with arrows).

``` clojure
(q/query (g/find-by-id 4)
         q/-->
         q/into-vec!)
;= [#<TinkerVertex v[5]> #<TinkerVertex v[3]>]

(q/query (g/find-by-id 4)
         q/out
         q/into-vec!)
;= [#<TinkerVertex v[5]> #<TinkerVertex v[3]>]

(q/query (g/find-by-id 4)
         (q/--> [:created])
         q/into-vec!)
;= [#<TinkerVertex v[5]> #<TinkerVertex v[3]>]

(q/query (g/find-by-id 4)
         (q/--> [:hates])
         q/into-vec!)
;= []

(q/query (g/find-by-id 4)
         (q/--> [:created :hates])
         q/into-vec!)
;= [#<TinkerVertex v[5]> #<TinkerVertex v[3]>]
```

### out-edges / -E>

Get the outgoing edges of the vertex.

``` clojure
(q/query (g/find-by-id 4)
         q/-E>
         q/into-vec!)
;= [#<TinkerEdge e[10][4-created->5]> #<TinkerEdge e[11][4-created->3]>]

(q/query (g/find-by-id 4)
         q/out-edges
         q/into-vec!)
;= [#<TinkerEdge e[10][4-created->5]> #<TinkerEdge e[11][4-created->3]>]
```

### out-vertex

Get the outgoing tail vertex of the edge.

``` clojure
(q/query (g/find-by-id 4)
         q/-E>
         q/out-vertex
         q/into-vec!)
;= [#<TinkerVertex v[4]> #<TinkerVertex v[4]>]
```

Conceptually, this might seem same strange at first. Why does it
return the same vertex twice? The answer lies in the example queries
for `-E>`. Those queries return two edges. The current query is the
same as the `-E>` query except we are asking for the `out-vertex`.
That means, by the time we are asking for the `out-vertex`, we have
two objects "in the pipeline". Thus, we get two objects back. 

### in / <--

Get the adjacent vertices pointing to the vertex.

``` clojure
(q/query (g/find-by-id 3)
         q/<--
         q/into-vec!)
;= [#<TinkerVertex v[1]> #<TinkerVertex v[4]> #<TinkerVertex v[6]>]
```

### in-edges / <E-

Get the incoming edges of the vertex.

``` clojure
(q/query (g/find-by-id 3)
         q/<E-
         q/into-vec!)
;= [#<TinkerEdge e[9][1-created->3]> 
;=  #<TinkerEdge e[11][4-created->3]> 
;=  #<TinkerEdge e[12][6-created->3]>]
```

### in-vertex

Get incoming head vertex of the edge.

``` clojure
(q/query (g/find-by-id 3)
         q/<E-
         q/in-vertex
         q/into-vec!)
;= [#<TinkerVertex v[3]> #<TinkerVertex v[3]> #<TinkerVertex v[3]>]
```

### both / <->

Get any vertices that are connected to the given vertex. 

``` clojure
(q/query (g/find-by-id 4)
         q/<->
         q/into-vec!)
;= [#<TinkerVertex v[1]> #<TinkerVertex v[5]> #<TinkerVertex v[3]>]
```

### both-edges / `<E>`

Get both incoming and outgoing edges of the vertex.

``` clojure
(q/query (g/find-by-id 4)
         q/<E>
         q/into-vec!)
;= [#<TinkerEdge e[8][1-knows->4]> 
;=  #<TinkerEdge e[10][4-created->5]> 
;=  #<TinkerEdge e[11][4-created->3]>]
```

### both-vertices

Get both incoming and outgoing vertices of the edge.

``` clojure
(q/query (g/find-by-id 4)
         q/<E>
         q/both-vertices
         q/into-vec!)
;= [#<TinkerVertex v[1]> #<TinkerVertex v[4]> 
;=  #<TinkerVertex v[4]> #<TinkerVertex v[5]> 
;=  #<TinkerVertex v[4]> #<TinkerVertex v[3]>]
```

### Map like functions are next

You should [read about map-like functions next](/articles/map.html). 


