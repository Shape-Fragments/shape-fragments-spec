# Negation
_Subsection about neighborhoods for constraints of kinds: `sh:NotConstraintComponent`_

<dfn data-lt="negation">Negated shapes</dfn> are shapes which a [=focus node=] should *not* satisfy. In SHACL, they are defined using `sh:not` or `sh:qualifiedMaxCount`; the latter says that enough of the focus node's [=value nodes=] do *not* satisfy the qualified shape. 

The <dfn>neighborhood of a focus node for a negated shape</dfn> depends on the constraints that compose the shape being negated. Neighborhood definitions for five [=kinds=] of constraints in the next five paragraphs are given; for other constraint kinds they are empty. Note: shapes with multiple constraints are treated as conjunctions, so the negation of such a shape should be treated like a negated conjunction (negation of `sh:and`).

For some constraint types, we define the neighborhood of their negation by giving an equivalent shape.  An engine could implement this for example using shape rewriting. The rewriting is guaranteed to end; it is always possible to reach a negation normal form where the neighborhood of every negated constraint is clearly defined.

## Negation of cardinality constraints
The negation of cardinality constraints is given by changing the cardinality constraint’s direction. As shown per example:

<aside title="Negation of a min count constraint" id="ex-not-minCount" class="example">
Following shapes are equivalent by pushing the negation (`sh:not`) inside the min count constraint (`sh:minCount`):
<pre class="ex-shape">
:negatedMinCountShape
  sh:not [ 
    sh:path :path ;
    sh:minCount 4 ] .
</pre>
<pre class="ex-shape">
:maxCountShape
  sh:path :path ;
  sh:maxCount 3 .
</pre>
</aside>

<aside title="Negation of a max count constraint" id="ex-not-maxCount" class="example">
Following shapes are equivalent by pushing the negation (`sh:not`) inside the max count constraint (`sh:maxCount`):
<pre class="ex-shape">
:negatedMaxCountShape
  sh:not [ 
    sh:path :path ;
    sh:maxCount 4 ] .
</pre>
<pre class="ex-shape">
:maxCountShape
  sh:path :path ;
  sh:minCount 5 .
</pre>
</aside>

## Negation of shape-based constraints
The neighborhood of negated shape-based constraints is given by pushing the negation inside those constraints and changing the quantifier. The laws we use to define the pushing inside are based on negations of quantified predicates. We illustrate the pushing inside by example for negated node shape constraints, negated qualified min count constraints and negated qualified max count constraints:

<aside title="Negation of a node constraint" id="ex-not-node" class="example">
Following shapes are equivalent by pushing the negation (`sh:not`) inside the node shape constraint (`sh:node`):
<pre class="ex-shape">
:negatedNodeShape
  sh:not [ 
    sh:path :path ;
    sh:node :shape ] .
</pre>
<pre class="ex-shape">
:minOneCounterExampleShape
  sh:path :path ; 
  sh:qualifiedMinCount 1 ;
  sh:qualifiedValueShape [ 
    sh:not :shape ] .
</pre>
</aside>

<aside title="Negation of a qualified min count constraint" id="ex-not-qualifiedMinCount" class="example">
Following shapes are equivalent by pushing the negation (`sh:not`) inside the qualified min count constraint (`sh:qualifiedMinCount`) and changing the qualified cardinality accordingly:
<pre class="ex-shape">
:negatedQualifiedMinCountShape
  sh:not [ 
    sh:path :path ;
    sh:qualifiedMinCount 4 ;
    sh:qualifiedValueShape :shape ] .
</pre>
<pre class="ex-shape">
:maxCountNegationShape
  sh:path :path ;
  sh:qualifiedMaxCount 3 ;
  sh:qualfiedValueShape [
    sh:not :shape ] .
</pre>
</aside>

<aside title="Negation of a qualified max count constraint" id="ex-not-qualifedMaxCount" class="example">
Following shapes are equivalent by pushing the negation (`sh:not`) inside the qualified max count constraint (`sh:qualifiedMaxCount`) and changing the qualified cardinality accordingly:
<pre class="ex-shape">
:negatedQualifiedMaxCountShape
  sh:not [ 
    sh:path :path ;
    sh:qualifiedMaxCount 4 ;
    sh:qualifiedValueShape :shape ] .
</pre>
<pre class="ex-shape">
:minCountNegationShape
  sh:path :path ;
  sh:qualifiedMinCount 5 ;
  sh:qualfiedValueShape [
    sh:not :shape ] .
</pre>
</aside>

## Negation of logical constraints
The neighborhood of negated logical constraints is given by pushing the negation inside those constraints using logical laws. Negating another negation removes both negations (double negative). For the other logical constraints, [De Morgan’s laws](https://en.wikipedia.org/wiki/De_Morgan%27s_laws) are used.

<aside title="Negation of an and constraint" id="ex-not-and" class="example">
Following shapes are equivalent by pushing the negation (`sh:not`) inside the conjunction (`sh:and`) using De Morgan’s law:
<pre class="ex-shape">
:negatedConjunctionShape
  sh:not [ 
    sh:and ( :shape1 :shape2) ] .
</pre>
<pre class="ex-shape">
:disjunctedNegationShape
  sh:or ( 
    [ sh:not :shape1 ]
    [ sh:not :shape2 ] ) .
</pre>
</aside>

## Negation of property pair constraints
The neighborhoods of negated property pair constraints contain those triples that show the property pair constraint is not satisfied. To define these neighborhoods, we will call use the name `E` to refer to the constraint’s property path expression and we will use `p` to refer to the property specified as after the property pair constraint’s parameter (`sh:equals`/`sh:disjoint`/`sh:lessThan`/`sh:lessThanOrEquals`).

The neighborhood of a focus node `x` for a negated equals constraint contains:
- The [=path triples=] for start node `x` and end node `y` for `E`, for those nodes `y` that are reachable from `x` through an `E`-path, but not through a `p`-path.
- The triples in the data graph that have subject `x`, predicate `p`, and object `y`, on the condition that `y` is not also reachable from `x` through an `E`-path.

The neighborhood of a focus node `x` for a negated disjointness constraint contains, for all nodes `y` that are both reachable from `x` through an `E`-path and a `p`-path:
- The [=path triples=] for start node `x` and end node `y` for `E`.
- The triple `(x, p, y)`.

For all nodes `y`, `z` where these three conditions hold: (i) `y` is not less than `z` (resp. `y` is not less than or equals) `z`, (ii) `y` is reachable from `z` through an `E`-path and (iii) `z` is reachable from `x` through a `p`-path, the neighborhood of a focus node `x` for a negated less than constraint (resp. less than or equal) contains:
- The [=path triples=] for start node `x` and end node `y` for `E`.
- The triple `(x, p, y)`.

## Negation of closure
The neighborhood of a node and a negated closed constraint component contains all triples in the data graph that (i) have the given node as subject and (ii) have a predicate that is neither specified in any property shape of non-closed shape, nor in the ignored property list.

<aside title="Negation of a closed constraint" id="ex-not-and" class="example">
<pre class="ex-input">
:Alice :firstName     "Alice" .
:Bob   :firstName     "Bob" ;
       :middleInitial "J" .
</pre>
<pre class="ex-shape">
:ClosedShapeExampleShape a sh:NodeShape ;
    sh:targetNode ex:Alice, ex:Bob ;
    sh:not [ sh:closed true ] ;
    sh:ignoredProperties (rdf:type) ;
    sh:property [ sh:path:firstName ] ;
    sh:property [ sh:path:lastName  ] .
</pre>
<pre class="ex-output">
:Bob :firstName "Bob" ;   # from the first property constraint’s neighborhood
     :middleInitial "J" . # from the negated closure’s neighborhood
</pre>
</aside>