# Constraint neighborhoods
The <dfn data-lt="neighborhood-constraint">neighborhood of a focus node for a constraint</dfn> are those triples from the [=data graph=] that show that the focus node satisfies the constraint. Which triples are contained in such a neighborhood depends on the [=kind=] of the constraint. In the remainder of this section, we give definitions of neighborhoods for the different constraint kinds in [SHACL core](https://www.w3.org/TR/shacl/#core-components).

## Value type constraint components
_Subsection about neighborhoods for constraints of kinds: `sh:ClassConstraintComponent`, `sh:DatatypeConstraintComponent`, `sh:NodeKindConstraintComponent`_

The <dfn data-lt="neighborhood-value-type">neighborhood of a focus node for a value type constraint</dfn> contains the triples from the data graph that make the focus node satisfy the constraint. For a value type constraint that is part of a [=property shape=], i.e., a shape with a [=property path=], the neighborhood of a focus node `x` contains, for each of the shape's [=value nodes=] `y`, the [=path triples=] between `x` and `y` for the shape's property path. 

Additionaly, the <dfn data-lt="neighborhood-class">neighborhood of a focus node for a class constraint</dfn> (`sh:ClassConstraintComponent`) contains, for each [=value node=] `y`, those triples in the [=data graph=] that declare the `y` has class the required class `c`. These are the [=path triples=] between the `y` and `c` for property path `rdf:type/rdfs:subClassOf*`.


<aside id="ex-class" title="Example with a class constraint" class="example">
<pre class="ex-input">
:Alice a :Person .
:Bob :address [ 
  a :PostalAddress ; 
  :city :Berlin ] .
:Carol :address [ 
  :city :Cairo ] .
</pre>
<pre class="ex-shape">
:ClassExampleShape a sh:NodeShape ;
  sh:targetNode :Bob, :Alice, :Carol ;
  sh:property [
    sh:path  :address ;
    sh:class :PostalAddress ; ] .
</pre>
<pre class="ex-output">
:Bob :address [ a :PostalAddress ] .
</pre>
</aside>

<aside id="ex-datatype" title="Example with a datatype constraint" class="example">
<pre class="ex-input">
:Alice :age "23"^^xsd:integer .
:Bob   :age "twenty two" .
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
</aside>

<aside id="ex-nodeKind" title="Example with a node kind constraint" class="example">
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
</aside>

## Cardinality constraint components
_Subsection about neighborhoods for constraints of kinds: `sh:MinCountConstraintComponent`, `sh:MaxCountConstraintComponent`_

The <dfn data-lt="neighborhood-cardinality">neighborhood of a focus node for a cardinality constraint</dfn> contains those triples that show that the focus node has the required amount of value nodes. For a cardinality constraint that is part of a [=property shape=], i.e., a shape with a [=property path=], the neighborhood of a focus node `x` contains the [=path triples=] between `x` and `y` for the property path for all value nodes `y`.

<aside id="ex-minCount" title="Example with a minimum count constraint" class="example">
Example with `sh:minCount`:
<pre class="ex-input">
:Alice :name "Alice" .
:Bob :givenName "Bob"@en .
</pre>
<pre class="ex-shape">
:minCountExampleShape a sh:PropertyShape ;
  sh:targetNode :Alice, :Bob ;
  sh:path :name ;
  sh:minCount 1 .
</pre>
<pre class="ex-output">
:Alice :name "Alice" .
</pre>
</aside>

## Value range constraint components
_Subsection about neighborhoods for constraints of kinds: `sh:MinExclusiveConstraintComponent`, `sh:MinInclusiveConstraintComponent`, `sh:MaxExclusiveConstraintComponent`, `sh:MaxInclusiveConstraintComponent`_

The <dfn data-lt="neighborhood-value-range">neighborhood of a focus node for a value range constraint</dfn> contains those triples that show that the focus node's value nodes lie in the required range. For a value range constraint that is part of a [=property shape=], i.e., a shape with a [=property path=], the neighborhood of a focus node `x` includes the [=path triples=] between `x` and `y` for the property path for all value nodes `y`.

<aside id="ex-minInclusive" title="Example with a minimum and maximum value constraint" class="example">
<pre class="ex-input">
:Bob   :age 23 .
:Alice :age 220 .
:Ted   :age "twenty one" .
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
</aside>

## String-based constraint components
_Subsection about neighborhoods for constraints of kinds: `sh:MinLengthConstraintComponent`, `sh:MaxLengthConstraintComponent`, `sh:PatternConstraintComponent`, `sh:LanguageInConstraintComponent`, `sh:UniqueLangConstraintComponent`_

The <dfn data-lt="neighborhood-string-based">neighborhood of a focus node for a string-based constraint</dfn> contains those triples that show that the focus node's value nodes match the required string values. For a string-based constraint that is part of a [=property shape=], i.e., a shape with a [=property path=], the neighborhood of a focus node `x` includes the [=path triples=] between `x` and `y` for the property path for all value nodes `y`.

<aside id="ex-minLength" title="Example with a minimum and maximum string length constraint" class="example">
<pre class="ex-input">
:Alice :password "1234567890ABC" .
:Bob   :password "123456789" .
</pre>
<pre class="ex-shape">
:PasswordExampleShape a sh:NodeShape ;
  sh:targetNode :Alice, :Bob ;
  sh:property [
    sh:path :password ;
    sh:minLength 8 ;
    sh:maxLength 10 ; ] .
</pre>
<pre class="ex-output">
:Bob :password "123456789" .
</pre>
</aside>

<aside id="ex-pattern" title="Example with a pattern constraint" class="example">
<pre class="ex-input">
:Alice :bCode "B102" .
:Bob   :bCode "b101" .
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
:Alice :bCode "B102" .
:Bob   :bCode "b101" .
</pre>
</aside>

<aside id="ex-languageIn" title="Example with a language tag constraint" class="example">
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
</aside>

<aside id="ex-uniqueLang" title="Example with a unique language constraint" class="example">
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
</aside>

## Property pair constraint components
_Subsection about neighborhoods for constraints of kinds: `sh:EqualsConstraintComponent`, `sh:DisjointConstraintComponent`, `sh:LessThanConstraintComponent`, `sh:LessThanOrEqualsConstraintComponent`_

Except constraints of kind equals, the <dfn data-lt="neighborhood-property-pair">neighborhood of a focus node for property pair constraint</dfn> is empty.

The <dfn data-lt="equals-property-pair">neighborhood of a focus for an equals constraint</dfn> contains all triples in the data graph with subject `x` and the predicate specified after `sh:equals`. Additionaly, for an equals constraint that is part of a [=property shape=], i.e., a shape with a [=property path=], the neighborhood of a focus node `x` includes the [=path triples=] between `x` and `y` for the property path for all value nodes `y`.

<aside id="ex-equals" title="Example with an equals constraint" class="example">
<pre class="ex-input">
:Alice :firstName "Alice" ;
       :givenName "Allie" .
:Bob   :firstName "Bob" ;
       :givenName "Bob" .
</pre>
<pre class="ex-shape">
:EqualExampleShape a sh:NodeShape ;
  sh:targetNode :Alice, :Bob ;
  sh:property [
    sh:path   :firstName ;
    sh:equals :givenName ] .
</pre>
<pre class="ex-output">
:Bob :firstName "Bob" ;
     :givenName "Bob" .
</pre>
</aside>

## Logical constraint components (excluding negation)
_Subsection about neighborhoods for constraints of kinds: `sh:AndConstraintComponent`, `sh:OrConstraintComponent`, `sh:XoneConstraintComponent`_

The <dfn data-lt="neighborhood-logical">neighborhood of a focus node for a logical constraint of kind "and", "or" or "xone"</dfn> are those triples in the data graph that show that the focus node satisfies the logical constraint. Such a neighborhood contains, for each shape `s` specified in the parameter list of the logical constraint and for each value node `y` of the focus node, the [=neighborhood-shape|(shape) neighborhood=] of `y` for `s`. Additionaly, for a logical constraint that is part of a [=property shape=], i.e., a shape with a [=property path=], the neighborhood of a focus node `x` includes, for each value node `y`, the [=path triples=] between `x` and `y` for the shape's property path.

<aside id="ex-and" title="Example with an and constraint" class="example">
<pre class="ex-input">
:ValidInstance   :property "One" .
:InvalidInstance :property "One" ;
                 :property "Two" .
</pre>
<pre class="ex-shape">
:SuperShape a sh:NodeShape ;
  sh:property [
  sh:path :property ;
  sh:minCount 1 ; ] .
:ExampleAndShape a sh:NodeShape ;
  sh:targetNode :ValidInstance, :InvalidInstance ;
  sh:and ( :SuperShape
           [ sh:path :property ;
             sh:maxCount 1 ; ] ) .
</pre>
<pre class="ex-output">
:ValidInstance :property "One" .
</pre>
</aside>

<aside id="ex-or" title="Example with an or constraint" class="example">
<pre class="ex-input">
:Bob :firstName "Robert" .
</pre>
<pre class="ex-shape">
:OrConstraintExampleShape a sh:NodeShape ;
  sh:targetNode :Bob ;
  sh:or ( [ sh:path :firstName ;
            sh:minCount 1 ; ]
          [ sh:path :givenName ;
            sh:minCount 1 ; ] ) .
</pre>
<pre class="ex-output">
:Bob :firstName "Robert" .
</pre>
</aside>

<aside id="ex-xone" title="Example with an exactly one constraint" class="example">
<pre class="ex-input">
:Bob a :Person ;
  :firstName "Robert" ;
  :lastName  "Coin" .
:Carla a :Person ;
  :fullName "Carla Miller" .
:Dory a :Person ;
  :firstName "Dory" ;
  :lastName  "Dunce" ;
  :fullName  "Dory Dunce" .
</pre>
<pre class="ex-shape">
:XoneConstraintExampleShape a sh:NodeShape ;
  sh:targetClass :Person ;
  sh:xone ( [ sh:property [
                sh:path :fullName ;
                sh:minCount 1 ; ] ; ]
            [ sh:property [
                sh:path :firstName ;
                sh:minCount 1 ; ] ;
            [ sh:property [
                sh:path :lastName ;
                sh:minCount 1 ; ] ; ] ) .
</pre>
<pre class="ex-output">
:Bob a :Person ;
  :firstName "Robert" ;
  :lastName  "Coin" .
:Carla a :Person ;
  :fullName "Carla Miller" .
</pre>
</aside>

## Shape-based constraint components
_Subsection about neighborhoods for constraints of kinds: `sh:NodeConstraintComponent`, `sh:PropertyShapeComponent`, `sh:QualifiedMinCountConstraintComponent`, `sh:QualifiedMaxCountConstraintComponent`_

The <dfn data-lt="neighborhood-node-property">neighborhood of a focus node for a node or property constraint</dfn> (`sh:node`, `sh:property`) contains the triples from the [=data graph=] that show that the focus node satisfies the shape-based constraint. In particular, these neighborhoods include those triples from the data graph that show that each of the [=value nodes=] satisfies the shape specified after `sh:node` or `sh:property`. Concretely, for each value node `y`, the [=neighborhood-shape|(shape) neighborhood=] of `y` for the shape specified after `sh:node`/`sh:property` is included in the neighborhood of a focus node for  a node or property constraint. 
Additionaly, for a node or property constraint that is part of a [=property shape=], i.e., a shape with a [=property path=], the neighborhood of a focus node `x` includes, for each value node `y`, the [=path triples=] between `x` and `y` for the shape's property path.

<aside id="ex-node" title="Example with a node constraint" class="example">
<pre class="ex-input">
:Bob a :Person ;
    :address :BobsAddress .
:BobsAddress :postalCode "1234" .
:Reto a :Person ;
    :address :RetosAddress .
:RetosAddress :postalCode 5678 .
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
:BobsAddress :postalCode "1234" .
</pre>
</aside>

The <dfn data-lt="neighborhood-qualifiedMinCount">neighborhood of a focus node for a qualified min count constraint</dfn> contains those triples from the [=data graph=] that show that the [=focus node=] has the required amount of [=value nodes=] that satisfy the specified shape. 
A neighborhood of a focus node `x` for a qualified min count constraint with qualified shape `s` contains, for each value node `y`:

- (Only if the constraint is part of a [=property shape=], i.e., a shape with a [=property path=],) the [=path triples=] between `x` and `y` for the property path .
- (Only if `y` satisfies `s`,) the [=neighborhood-shape|(shape) neighborhood=] of `y` for `s`.

<aside id="ex-qualifiedMinCount" title="Example with a qualified min count constraint" class="example">
<pre class="ex-input">
:QualifiedValueShapeExampleValidResource
  :parent :John ;
  :parent :Jane .
:John :gender :male .
:Jane :gender :female .
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
:Jane :gender :female .
</pre>
</aside>

The <dfn data-lt="neighborhood-qualifiedMaxCount">neighborhood of a focus node for a qualified max count constraint</dfn> contains those triples from the [=data graph=] that show that the [=focus node=] has no more than the allowed amount of [=value nodes=] that satisfy the specified shape. 
A neighborhood of a focus node `x` for a qualified max count constraint with qualified shape `s` contains, for each value node `y`:

- (Only if the constraint is part of a [=property shape=], i.e., a shape with a [=property path=],) the [=path triples=] between `x` and `y` for the property path .
- (Only if `y` satisfies the [=negation=] of `s`,) the [=neighborhood-shape|(shape) neighborhood=] of `y` for `s`.

## Other constraints
_Subsection about neighborhoods for constraints of kinds: sh:ClosedConstraintComponent, sh:HasValueConstraintComponent, sh:InConstraintComponent_

The <dfn data-lt="neighborhood-closed">neighborhood of a focus node for a closed constraint</dfn> is empty. Such a constraint (like all other constraints) influences whether the focus node satisfies the shape of which the constraint is part, and therefore influences whether the [=neighborhood-shape|neighborhood of the focus node for that shape=] is empty, but the constraint itself has does not contribute to the neighborhood for that shape. 

<aside title="Example with a closed constraint" id="ex-closed" class="example">
Example with `sh:closed`: 
<pre class="ex-input">
:Alice :firstName "Alice" .
:Bob :firstName "Bob" ;
     :middleInitial "J" .
</pre>
<pre class="ex-shape">
:ClosedShapeExampleShape a sh:NodeShape ;
  sh:targetNode :Alice, :Bob ;
  sh:closed true ;
  sh:ignoredProperties (rdf:type) ;
  sh:property [ sh:path :firstName ; ] ;
  sh:property [ sh:path :lastName ; ] .
</pre>
<pre class="ex-output">
:Alice :firstName "Alice" .
</pre>
</aside>

The <dfn data-lt="neighborhood-hasValue">neighborhood of a focus node for a has value constraint</dfn> contains those triples from the [=data graph=] that show that at least one of the [=focus node=]'s [=value nodes=] has the required value.
For a has value constraint that has required value `v`  and that is part of a [=property shape=], i.e., a shape that has a [=property path=], the neighborhood of a focus node `x` contains, for each value node `y` _that is equal to `v`_, the [=path triples=] between start node `x` and end node `y` for the property shape's property path.
<aside title="Example with a has value constraint" id="ex-hasValue" class="example">
<pre class="ex-input">
:Alice :alumniOf :Harvard ;
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
:Alice :alumniOf :Stanford .
</pre>
</aside>

The <dfn data-lt="neighborhood-in">neighborhood of a focus node for an in constraint</dfn> contains those triples from the [=data graph=] that show that all of the [=focus node=]'s [=value nodes=] have one of the required values.
For in constraints that are part of a [=property shape=], i.e., a shape that has a [=property path=], the neighborhood of a focus node `x` contains, for each value node `y`, the [=path triples=] between start node `x` and end node `y` for the property shape's property path.
<aside title="Example with a has value constraint" id="ex-hasValue" class="example">
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
</aside>