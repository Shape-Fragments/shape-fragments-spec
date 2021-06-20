# Extracting subgraphs with shapes {#shape-fragments}

A **shapes graph fragment** is a subgraph derived from a data graph for a given shapes graph. A shapes graph fragment is the union of the shape fragments of the individual shapes in the shapes graph. For each shape in the shapes graph, the **shape fragment** contains the neighborhoods for that shape and each of its [targets](https://www.w3.org/TR/shacl/#dfn-target).

The **neighborhood for a shape and a node** is a subgraph of the data graph. If the node does not conform to the shape, the neighborhood is empty. If the node does conform to the shape, the neighborhood for the shape and the node is the union of:

- The neighborhoods for the node and each of the shape’s constraints.
- The neighborhoods for the node and each of the shape’s target declarations. 

The definition of a neighborhood for a node and a constraint or target declaration depends on the [kind of the constraint](https://www.w3.org/TR/shacl/#dfn-constraint-component) or [target declaration](https://www.w3.org/TR/shacl/#dfn-target-declarations). In the remainder of this section, we give definitions of neighborhoods for the different target declarations and constraint components in [SHACL core](https://www.w3.org/TR/shacl/#core-components).

## Correctness property
Shape fragments are designed to be a “correct” interpretation of SHACL. By “correct” is meant that: (i) a shapes graph fragment contains all the targets from the data graph that conform to the shape in the shape graph they are a target for and (ii) the shape fragment conforms to the shapes graph.

<figure>
  <img src="./resources/images/extraction-validation.png" alt="Target structure"/>
  <figcaption>Subgraph extraction with shape fragments returns the validated subgraph of a graph instead of a validation report.</figcaption>
</figure>

## Targets
The neighborhood for a [target declaration](https://www.w3.org/TR/shacl/#dfn-target-declarations) and a node intuitively contains the triples from the data graph that make the node a target of the target declaration. If the node is not a [target node](https://www.w3.org/TR/shacl/#dfn-target) of the target declaration, the neighborhood is empty. If the node *is* a target node of the target declaration, the neighborhood defined as follows, for the four kinds of target declarations:

- The neighborhood for a node and a `sh:targetClass` target declaration are the triples in the data graph that declare the node has the specified class. This is similar to extracting class relations.
- The neighborhood for a node and a `sh:targetSubjectsOf` target declaration are the triples in the data graph with as subject the node and as predicate the specified predicate.
- The neighborhood for a node and a `sh:targetObjectsOf` target declaration are the triples in the data graph with as object the node and as predicate the specified predicate.
- The neighborhood for a node and a `sh:targetNode` target declaration is empty.

### Examples

Example with `sh:targetClass`:

<pre class="ex-input">
:Alice 
  a :Person .
:Bob 
  a :Person .
:NewYork 
  a :City .
</pre>
<pre class="ex-shape">
:PersonShape
  a sh:NodeShape ;
  sh:targetClass :Person .
</pre>
<pre class="ex-output">
:Alice 
  a :Person .
:Bob 
  a :Person .
</pre>



Example with `sh:targetSubjectsOf`:

<pre class="ex-input">
:Alice 
  :knows :Bob .
:Bob 
  :livesIn
    :NewYork .
</pre>
<pre class="ex-shape">
:TSOExampleShape
  a sh:NodeShape ;
  sh:targetSubjectsOf 
    :knows .
</pre>
<pre class="ex-output">
:Alice 
  :knows :Bob .
</pre>



## Value type constraint components
_Subsection about neighborhoods for constraints of kinds: `sh:ClassConstraintComponent`, `sh:DatatypeConstraintComponent`, `sh:NodeKindConstraintComponent`_

The neighborhood for a node and a constraint of a [value type](https://www.w3.org/TR/shacl/#core-components-value-type) kind contains the triples from the data graph that make the node have the specified type of value.

If the value type constraint is kind `sh:ClassConstraintComponent`, the neighborhood contains the triples in the data graph that declare the node has the class specified with `sh:class`. These are the triples on paths in the data graph that match the property path `$value rdf:type/rdfs:subClassOf* $class`.

If a value type constraint is used in a property shape, the neighborhood for a node x and this constraint also includes the [path triples](#path-triples) defined by the property shape’s property path expression, start node x and end node y, for all nodes y that conform to the constraint.
### Examples
Example with `sh:class`:

<pre class="ex-input">
:Alice a :Person .
:Bob :address [ a :PostalAddress ; :city :Berlin ] .
:Carol :address [ :city :Cairo ] .
</pre>
<pre class="ex-shape">
:ClassExampleShape a sh:NodeShape ;
  sh:targetNode :Bob, :Alice, :Carol ;
  sh:property [
    sh:path :address ;
    sh:class :PostalAddress ; ] .
</pre>
<pre class="ex-output">
:Bob :address [ a :PostalAddress ] .
</pre>

Example with `sh:datatype`:
<pre class="ex-input">
:Alice :age "23"^^xsd:integer .
:Bob :age "twenty two" .
:Carol :age "23"^^xsd:int .
</pre>
<pre class="ex-shape">
:DatatypeExampleShape a sh:NodeShape ;
  sh:targetNode :Alice, :Bob, :Carol ;
  sh:property [
    sh:path :age ;
    sh:datatype xsd:integer ; ] .
</pre>
<pre class="ex-output">
:Alice :age "23"^^xsd:integer .
</pre>

Example with `sh:nodeKind`:
<pre class="ex-input">
:Bob :knows :Alice .
:Alice :knows "Bob" .
</pre>
<pre class="ex-shape">
:NodeKindExampleShape a sh:NodeShape ;
  sh:targetObjectsOf :knows ;
  sh:nodeKind sh:IRI .
</pre>
<pre class="ex-output">
:Bob :knows :Alice .
</pre>

## Cardinality constraint components
_Subsection about neighborhoods for constraints of kinds: `sh:MinCountConstraintComponent`, `sh:MaxCountConstraintComponent`_

The neighborhood for cardinality constraints contains those triples that show that nodes conform to this constraint. For both minimum and maximum cardinality constraints, the neighborhood of a node `x` contains the [path triples](#path-triples) defined by the constraint’s property path expression, start node `x` and end node `y`, for all nodes `y` reachable from `x` through paths that match the property path expression.
### Examples
Example with `sh:minCount`:
<pre class="ex-input">
:Alice :name "Alice" .
:Bob :givenName "Bob"@en .
</pre>
<pre class="ex-shape">
:MinCountExampleShape a sh:PropertyShape ;
  sh:targetNode :Alice, :Bob ;
  sh:path :name ;
  sh:minCount 1 .
</pre>
<pre class="ex-output">
:Alice :name "Alice" .
</pre>

## Value range constraint components
_Subsection about neighborhoods for constraints of kinds: `sh:MinExclusiveConstraintComponent`, `sh:MinInclusiveConstraintComponent`, `sh:MaxExclusiveConstraintComponent`, `sh:MaxInclusiveConstraintComponent`_

If a value range constraint is used in a property shape, the neighborhood for a node `x` and this constraint includes the [path triples](#path-triples) defined by the property shape’s property path expression, start node `x` and end node `y`, for all nodes `y` reachable from `x` through a path matching the property path expression.
### Examples
Example with `sh:minInclusive` and `sh:maxInclusive`: 
<pre class="ex-input">
:Bob :age 23 .
:Alice :age 220 .
:Ted :age "twenty one" .
</pre>
<pre class="ex-shape">
:NumericRangeExampleShape a sh:NodeShape ;
  sh:targetNode :Bob, :Alice, :Ted ;
  sh:property [
    sh:path :age ;
    sh:minInclusive 0 ;
    sh:maxInclusive 150 ; ] .
</pre>
<pre class="ex-output">
:Bob :age 23 .
</pre>
## String-based constraint components
_Subsection about neighborhoods for constraints of kinds: `sh:MinLengthConstraintComponent`, `sh:MaxLengthConstraintComponent`, `sh:PatternConstraintComponent`, `sh:LanguageInConstraintComponent`, `sh:UniqueLangConstraintComponent`_

If a string-based constraint is used in a property shape,  the neighborhood for a node `x` and this constraint includes the [path triples](#path-triples) defined by the property shape’s property path expression, start node `x` and end node `y`, for all nodes `y` reachable from `x` through a path matching the property path expression.
Examples
Example with `sh:minLength` and `sh:maxLength`: 
<pre class="ex-input">
:Bob :password "123456789" .
:Alice :password "1234567890ABC" .


</pre>
<pre class="ex-shape">
:PasswordExampleShape a sh:NodeShape ;
  sh:targetNode :Bob, :Alice ;
  sh:property [
    sh:path :password ;
    sh:minLength 8 ;
    sh:maxLength 10 ; ] .


</pre>
<pre class="ex-output">
:Bob :password "123456789" .
</pre>

Example with `sh:pattern`: 
<pre class="ex-input">
:Bob :bCode "b101" .
:Alice :bCode "B102" .
:Carol :bCode "C103" .


</pre>
<pre class="ex-shape">
:PatternExampleShape
  a sh:NodeShape ;
  sh:targetNode :Bob, :Alice, :Carol ;
  sh:property [
    sh:path :bCode ;
    sh:pattern "^B" ;    # starts with 'B'
    sh:flags "i" ;       # Ignore case
  ] .


</pre>
<pre class="ex-output">
:Bob :bCode "b101" .
:Alice :bCode "B102" .
</pre>

Example with `sh:languageIn`: 
<pre class="ex-input">
:Mountain
  :prefLabel "Mountain"@en ;
  :prefLabel "Hill"@en-NZ ;
  :prefLabel "Maunga"@mi .

:Berg
  :prefLabel "Berg" ;
  :prefLabel "Berg"@de ;
  :prefLabel :BergLabel .
</pre>
<pre class="ex-shape">
:NewZealandLanguagesShape a sh:NodeShape ;
  sh:targetNode :Mountain, :Berg ;
  sh:property [
    sh:path :prefLabel ;
    sh:languageIn ( "en" "mi" ) ; ] .
</pre>
<pre class="ex-output">
:Mountain
  :prefLabel "Mountain"@en ;
  :prefLabel "Hill"@en-NZ ;
  :prefLabel "Maunga"@mi .
</pre>

Example with `sh:UniqueLang`: 
<pre class="ex-input">
:Alice
  :label "Alice" ;
  :label "Alice"@en ;
  :label "Alice"@fr .

:Bob
  :label "Bob"@en ;
  :label "Bobby"@en .
</pre>
<pre class="ex-shape">
:UniqueLangExampleShape a sh:NodeShape ;
  sh:targetNode :Alice, :Bob ;
  sh:property [
    sh:path :label ;
     sh:uniqueLang true ; ] .
</pre>
<pre class="ex-output">
:Alice
  :label "Alice" ;
  :label "Alice"@en ;
  :label "Alice"@fr ."
</pre>

## Property pair constraint components
_Subsection about neighborhoods for constraints of kinds: `sh:EqualsConstraintComponent`, `sh:DisjointConstraintComponent`, `sh:LessThanConstraintComponent`, `sh:LessThanOrEqualsConstraintComponent`_

Except for equals constraint components, the neighborhoods for property pair constraint components are empty.

The neighborhood for a node `x` and an equals constraint includes both (i) [path triples](#path-triples) defined by the property shape’s property path expression, start node `x` and end node `y` for all nodes `y`, and (ii) triples in the data graph with subject `x` and the predicate specified after `sh:equals`.
### Examples
Example with `sh:equals`: 
<pre class="ex-input">
:Bob
  :firstName "Bob" ;
  :givenName "Bob" .
</pre>
<pre class="ex-shape">
:EqualExampleShape a sh:NodeShape ;
  sh:targetNode :Bob ;
  sh:property [
    sh:path :firstName ;
    sh:equals :givenName ; ] .
</pre>
<pre class="ex-output">
:Bob
  :firstName "Bob" ;
  :givenName "Bob" .
</pre>

## Logical constraint components (excluding negation)
_Subsection about neighborhoods for constraints of kinds: `sh:AndConstraintComponent`, `sh:OrConstraintComponent`, `sh:XoneConstraintComponent`_

The neighborhood for a node and a logical constraint of kind "and", "or" or "xone" is the union of the neighborhoods of the shapes that are contained in the list specified in the constraint parameter.
### Examples

Example with `sh:and`: 
<pre class="ex-input">
:ValidInstance :property "One" .

# Invalid: more than one property
:InvalidInstance
  :property "One" ;
  :property "Two" .
</pre>
<pre class="ex-shape">
:SuperShape a sh:NodeShape ;
  sh:property [
  sh:path :property ;
  sh:minCount 1 ; ] .

:ExampleAndShape a sh:NodeShape ;
  sh:targetNode :ValidInstance, :InvalidInstance ;
  sh:and (
    :SuperShape
    [
       sh:path :property ;
       sh:maxCount 1 ; ] ) .
</pre>
<pre class="ex-output">
:ValidInstance :property "One" .
</pre>

Example with `sh:or`: 
<pre class="ex-input">
:Bob :firstName "Robert" .
</pre>
<pre class="ex-shape">
:OrConstraintExampleShape a sh:NodeShape ;
  sh:targetNode :Bob ;
  sh:or (
    [
       sh:path :firstName ;
       sh:minCount 1 ; ]
    [
       sh:path :givenName ;
       sh:minCount 1 ; ] ) .
</pre>
<pre class="ex-output">
:Bob :firstName "Robert" .
</pre>

Example with `sh:xone`: 
<pre class="ex-input">
:Bob a :Person ;
  :firstName "Robert" ;
  :lastName "Coin" .

:Carla a :Person ;
  :fullName "Carla Miller" .
        
:Dory a :Person ;
  :firstName "Dory" ;
  :lastName "Dunce" ;
  :fullName "Dory Dunce" .
</pre>
<pre class="ex-shape">
:XoneConstraintExampleShape a sh:NodeShape ;
  sh:targetClass :Person ;
  sh:xone (
    [
       sh:property [
         sh:path :fullName ;
         sh:minCount 1 ; ] ; ]
    [
       sh:property [
         sh:path :firstName ;
         sh:minCount 1 ; ] ;
       sh:property [
         sh:path :lastName ;
         sh:minCount 1 ; ] ; ] ) .
</pre>
<pre class="ex-output">
:Bob a :Person ;
  :firstName "Robert" ;
  :lastName "Coin" .

:Carla a :Person ;
  :fullName "Carla Miller" .
</pre>

## Shape-based constraint components
_Subsection about neighborhoods for constraints of kinds: `sh:NodeConstraintComponent`, `sh:PropertyShapeComponent`, `sh:QualifiedMinCountConstraintComponent`, `sh:QualifiedMaxCountConstraintComponent`_

The neighborhood for a node and a shape-based constraint depends on the type of shape-based constraint.

For node and property constraint components (`sh:node`, `sh:property`), the neighborhood of a node contains all the triples in the neighborhood for that node and the shape specified after `sh:node`/`sh:property`. If the constraint is used in a property shape, the neighborhood for a node `x` and this constraint also includes the [path triples](#path-triples) defined by the property shape’s property path expression, start node `x` and end node `y`, for all nodes `y` reachable from `x` through a path matching the property path expression.

For qualified min count constraint components, the neighborhood of a node `x` contains following triples for each node `y` that both (i) conforms to the shape specified after `sh:qualifiedValueShape` and (ii) is reachable from `x` through a path matching the property path expression:
- all [path triples](#path-triples) defined by the shape’s property path expression, start node `x` and end node ``y
- the neighborhood for node `y` and the shape specified after `sh:qualifiedValueShape`.

For qualified max count constraint components, the neighborhood of a node `x` contains following triples for each node `y` that both (i) conforms to the negation of the shape specified after `sh:qualifiedValueShape` and (ii) is reachable from `x` through a path matching the property path expression:
- all [path triples](#path-triples) defined by the shape’s property path expression, start node `x` and end node `y`
- the neighborhood for node `y` and the negation of the shape specified after `sh:qualifiedValueShape`.
### Examples
Example with `sh:node`: 
<pre class="ex-input">
:Bob a :Person ;
  :address :BobsAddress .
        
:BobsAddress
  :postalCode "1234" .

:Reto a :Person ;
  :address :RetosAddress .

:RetosAddress
  :postalCode 5678 .
</pre>
<pre class="ex-shape">
:AddressShape a sh:NodeShape ;
  sh:property [
    sh:path :postalCode ;
    sh:datatype xsd:string ;
    sh:maxCount 1 ; ] .

:PersonShape a sh:NodeShape ;
  sh:targetClass :Person ;
  sh:property [   # _:b1
    sh:path :address ;
    sh:minCount 1 ;
    sh:node :AddressShape ;  ] .
</pre>
<pre class="ex-output">
:Bob a :Person ;
  :address :BobsAddress .
        
:BobsAddress
  :postalCode "1234" .
</pre>

Example with `sh:qualifiedMinCount`: 
<pre class="ex-input">
:QualifiedValueShapeExampleValidResource
  :parent :John ;
  :parent :Jane .

:John
  :gender :male .

:Jane
  :gender :female .
</pre>
<pre class="ex-shape">
:QualifiedValueShapeExampleShape a sh:NodeShape ;
  sh:targetNode :QualifiedValueShapeExampleValidResource ;
  sh:property [
    sh:path :parent ;
    sh:minCount 2 ;
    sh:maxCount 2 ;
    sh:qualifiedValueShape [
       sh:path :gender ;
       sh:hasValue :female ; ] ;
    sh:qualifiedMinCount 1 ; ] .
</pre>
<pre class="ex-output">
:QualifiedValueShapeExampleValidResource
  :parent :John ;
  :parent :Jane .

:Jane
  :gender :female .
</pre>

## Other constraint components
_Subsection about neighborhoods for constraints of kinds: sh:ClosedConstraintComponent, sh:HasValueConstraintComponent, sh:InConstraintComponent_

The neighborhood for “other” constraints is empty. These constraints are used as a condition on whether the neighborhood for the shape they are part of is empty for a given node, but do not return anything themselves.
### Examples
Example with `sh:closed`: 
<pre class="ex-input">
:Alice
  :firstName "Alice" .

:Bob
  :firstName "Bob" ;
  :middleInitial "J" .
</pre>
<pre class="ex-shape">
:ClosedShapeExampleShape a sh:NodeShape ;
  sh:targetNode :Alice, :Bob ;
  sh:closed true ;
  sh:ignoredProperties (rdf:type) ;
  sh:property [
    sh:path :firstName ; ] ;
  sh:property [
    sh:path :lastName ; ] .
</pre>
<pre class="ex-output">
:Alice
  :firstName "Alice" .
</pre>

Example with `sh:hasValue`: 
<pre class="ex-input">
:Alice
  :alumniOf :Harvard ;
  :alumniOf :Stanford .
</pre>
<pre class="ex-shape">
:StanfordGraduate a sh:NodeShape ;
  sh:targetNode :Alice ;
  sh:property [
    sh:path :alumniOf ;
    sh:hasValue :Stanford ; ] .
</pre>
<pre class="ex-output">
:Alice
  :alumniOf :Stanford .
</pre>

Example with `sh:in`: 
<pre class="ex-input">
:RainbowPony :color :Pink .
</pre>
<pre class="ex-shape">
:InExampleShape a sh:NodeShape ;
  sh:targetNode :RainbowPony ;
  sh:property [
    sh:path :color ;
    sh:in ( :Pink :Purple ) ; ] .
</pre>
<pre class="ex-output">
:RainbowPony :color :Pink .
</pre>

## Path triples {#path-triples}
We introduce the term [path triples](#path-triples) to refer to triples in the data graph that lay on a path. Path triples are defined by a property path expression, a start node, and an end node. Given a property path expression, start node and end node, the [path triples](#path-triples) are exactly those triples in the data graph on paths that (i) start in the start node, (ii) match the property path and (iii) end in the end node.
### Examples
In the following example, the first triple in the input matches the above criteria and is present in the shape fragment. The second triple in the input is not returned since its end node does not conform to the example constraints: it is an IRI instead of a literal. The third triple in the input is not returned since its predicate does not match the path expression

<pre class="ex-input">
:Alice 
 :name “Alice” .
:Bob :name :bobsName .
:Carol :hasName “Carol” . 
</pre>
<pre class="ex-shape">
:PersonShape
  a sh:PropertyShape ;
  sh:targetNode :Alice, :Bob, :Carol ;
  sh:path :name ;
  sh:nodeKind sh:Literal .
</pre>
<pre class="ex-output">
:Alice 
 :name “Alice” .
</pre>

Following example shape will extract a fragment containing transitive friends of `:Alice` and `:Xander` (if any).

<pre class="ex-input">
:Alice 
  :friend :Bob .
:Bob 
  :friend :Carol . 
:Xander :likes :Yolanda .
</pre>
<pre class="ex-shape">
:PersonShape
  a sh:PropertyShape ;
  sh:targetNode :Alice, :Xander .
  sh:path [ 
    sh:zeroOrMorePath :friend ] ;
  sh:minCount 1 .
</pre>
<pre class="ex-output">
:Alice 
  :friend :Bob .
:Bob 
  :friend :Carol . 
</pre>

## Negation
_Subsection about neighborhoods for constraints of kinds: `sh:NotConstraintComponent`_

The neighborhood for a negative constraint (defined with `sh:not`) depends on the shape being negated. We give the neighborhood definitions for five types of constraints in the next five paragraphs. For other constraints they are empty. 

For some constraint types, we define the neighborhood of their negation by giving an equivalent shape.  An engine could implement this for example using shape rewriting. The rewriting is guaranteed to end; it is always possible to reach a negation normal form where the neighborhood of every negated constraint is clearly defined.

Note: we treat shapes with multiple constraints as conjunctions of their constraints, so the negation of such a shape should be treated like a negated conjunction.
### Negation of cardinality constraints
The negation of cardinality constraints is given by changing the cardinality constraint’s direction. As shown per example.
#### Examples
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

### Negation of shape-based constraints
The neighborhood of negated shape-based constraints is given by pushing the negation inside those constraints and changing the quantifier. The laws we use to define the pushing inside are based on negations of quantified predicates. We illustrate the pushing inside by example for negated node shape constraints, negated qualified min count constraints and negated qualified max count constraints:
#### Examples
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

### Negation of logical constraints
The neighborhood of negated logical constraints is given by pushing the negation inside those constraints using logical laws. Negating another negation removes both negations (double negative). For the other logical constraints, De Morgan’s laws are used.
#### Examples
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

### Negation of property pair constraints
The neighborhoods of negated property pair constraints contain those triples that show the property pair constraint is not satisfied. To define these neighborhoods, we will call use the name `E` to refer to the constraint’s property path expression and we will use `p` to refer to the property specified as after the property pair constraint’s parameter (`sh:equals`/`sh:disjoint`/`sh:lessThan`/`sh:lessThanOrEquals`).

The neighborhood of a node `x` and a negated equals constraint component contains:
- The [path triples](#path-triples) defined by path expression `E`, start node `x` and end node `y`, for those nodes `y` that are reachable from `x` through an `E`-path, but not through a `p`-path.
- The triples in the data graph that have subject `x`, predicate `p`, and object `y`, on the condition that `y` is not also reachable from `x` through an `E`-path.

The neighborhood of a node `x` and a negated disjointness constraint component contains for all nodes `y` that are both reachable from `x` through an `E`-path and a `p`-path:
- The [path triples](#path-triples) defined by `E`, start node `x` and end node `y`.
- The triple `(x, p, y)`.

For all nodes `y`, `z` where these three conditions hold: (i) `y` is not less than `z` (resp. `y` is not less than or equals) `z`, (ii) `y` is reachable from `z` through an `E`-path and (iii) `z` is reachable from `x` through a `p`-path, the neighborhood of a node `x` and a negated less than (resp. less than or equal) constraint component contains:
- The [path triples](#path-triples) defined by `E`, start node `x` and end node `y`.
- The triple `(x, p, y)`.

### Negation of closure
The neighborhood of a node and a negated closed constraint component contains all triples in the data graph that (i) have the given node as subject and (ii) have a predicate that is neither specified in any property shape of the closed shape, nor in the ignored property list.
#### Example

<pre class="ex-input">
:Alice
  :firstName "Alice" .

:Bob
  :firstName "Bob" ;
  :middleInitial "J" .
</pre>
<pre class="ex-shape">
:ClosedShapeExampleShape
    a sh:NodeShape ;
    sh:targetNode ex:Alice, ex:Bob ;
    sh:not [ sh:closed true ] ;
    sh:ignoredProperties (rdf:type) ;
    sh:property [
        sh:path:firstName ;
    ] ;
    sh:property [
        sh:path:lastName ;
    ] .
</pre>
<pre class="ex-output">
:Bob
  :firstName "Bob" ;   # from the first property constraint’s neighborhood
  :middleInitial "J" . # from the negated closure’s neighborhood
</pre>