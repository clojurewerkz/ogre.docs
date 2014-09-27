---
title: "Side effects in Ogre, a graph query library"
layout: article
---

## Side Effect

At this stage in Ogre's development, most side effect steps immediately
return various data structures about the query. 

### get-grouped-by!

Takes in a key function and processing function. Returns all of the
processed objects grouped by the value of the key function.

```clojure
(q/query (g/get-vertices)
         (q/get-grouped-by! (q/prop :lang)
                            identity))
;= {nil [#<TinkerVertex v[2]> #<TinkerVertex v[1]> 
;=      #<TinkerVertex v[6]> #<TinkerVertex v[4]>], 
;=  "java" [#<TinkerVertex v[3]> #<TinkerVertex v[5]>]}

(q/query (g/get-vertices)
         (q/get-grouped-by! (q/prop :lang)
                            (q/prop :name)))
;= {nil ["vadas" "marko" "peter" "josh"], "java" ["lop" "ripple"]}


(q/query (g/get-vertices)
         (q/get-grouped-by! (q/prop :lang)
                            #(q/query % q/out q/into-vec!)))
;= {nil [[] 
;=       [#<TinkerVertex v[2]> #<TinkerVertex v[4]> #<TinkerVertex v[3]>]
;=       [#<TinkerVertex v[3]>] 
;=       [#<TinkerVertex v[5]> #<TinkerVertex v[3]>]], 
;=  "java" [[] []]}
```

### get-group-count!

Takes in a key function, and optionally, a counting function. Returns
the count of the objects grouped by the key function.

```clojure 
(q/query (g/get-vertices)
         (q/get-group-count! (q/prop :lang)))
;= {nil 4, "java" 2}
(q/query (g/get-vertices)
         (q/get-group-count! (q/prop :lang)
                             (fn [a b] (+ b (count (.getProperty a "name"))))))
;= {nil 19, "java" 9}
```

### get-table!

Returns a list of maps that correspond to the values of the named
steps. Optional arguments are a list of functions that are applied
round robin style.

```clojure 
(q/query (g/get-vertices)
         (q/property :name)
         (q/as "name")
         (q/back 1)
         (q/property :age)
         (q/as "age")
         (q/get-table!))
;= ({:name "lop", :age nil} {:name "vadas", :age 27} 
;=  {:name "marko", :age 29} {:name "peter", :age 35} 
;=  {:name "ripple", :age nil} {:name "josh", :age 32})         

(q/query (g/get-vertices)
         (q/property :name)
         (q/as "name")
         (q/back 1)
         (q/property :age)
         (q/as "age")
         (q/get-table! count #(or % 18)))
;= ({:name 3, :age 18} {:name 5, :age 27} 
;=  {:name 5, :age 29} {:name 5, :age 35} 
;=  {:name 6, :age 18} {:name 4, :age 32})
```

### get-tree!

Returns a tree of the objects encountered taken while executing the
query. Key functions can be supplied as well.

```clojure
(q/query (g/find-by-id 1)
         q/-->
         q/-->
         (q/get-tree! (q/prop :name)))
;= {:value "marko", 
;=  :children [{:value "josh", 
;=              :children [{:value "lop"} 
;=                         {:value "ripple"}]}]}         
```

### side-effect

Execute some side-effect.

```clojure
(let [lst (atom [])
      elem (g/find-by-id 1)
      names (q/query elem
                  q/-->
                  (q/side-effect (partial swap! lst conj))
                  (q/property :name)
                  q/into-vec!)]
      [@lst names])
;= [[#<TinkerVertex v[2]> #<TinkerVertex v[4]> #<TinkerVertex v[3]>] 
;=  ["vadas" "josh" "lop"]]      
```

### Conclusion

You should [read the conclusion next](/articles/conclusion.html). 
