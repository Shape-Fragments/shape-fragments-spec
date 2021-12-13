# Extracting subgraphs with shapes {#shape-fragments}

A <dfn>shapes graph fragment</dfn> is a subgraph derived from a data graph for a given shapes graph. A shapes graph fragment is the union of the [=shape fragments=] of the individual shapes in the shapes graph. For each shape in the shapes graph, that shape's <dfn>shape fragment</dfn> contains the [=neighborhood-shape|neighborhood for that shape=] of each focus node in the shape's [=target=].

<figure>
  <img src="./resources/images/extraction-validation.png" alt="Diagram showing that Shape Fragments outputs a subgraph based on the input data and shapes graph."/>
  <figcaption>Subgraph extraction with shape fragments returns the validated subgraph of a graph instead of a validation report.</figcaption>
</figure>

## Shape neighborhoods

The <dfn data-lt="neighborhood-shape">neighborhood of a focus node for a shape</dfn> is a subgraph of the data graph. If a focus node does not conform to a shape, the neighborhood of that node for that shape is defined as being empty. If a focus node *does* conform to a shape, the neighborhood of that focus node for that shape is defined as containing exactly:

- The triples of the data graph that make the focus node part of the shape's target: the [=target triples=]. (This is empty if the focus node is not part of the shape's target.)
- The triples of the data graph that make the focus node satisfy the shape's [=constraints=]: for each of the shape's constraints, the [=neighborhood-constraint|neighborhood of the focus node for the constraint=].

## Target triples
The <dfn>target triples</dfn> of a focus node for a shape intuitively are those triples in the data graph that make the focus node part of the [=target=] of that shape. 
The target triples of a focus node for a shape are defined as the union of the target triples for that focus node and each of the shape's [=target declarations=]: 

- The target triples of a focus node for a `sh:targetClass` target declaration are the triples in the data graph that declare the node has the specified class. These are the [=path triples=] between the focus node and the target class node for property path `rdf:type/rdfs:subClassOf*`. (See [[[#ex-targetClass]]].) This definition is (intentionally) similar to the definition of [=neighborhood-class|the neighborhood of a class constraint=]. 
- The target triples of a focus node for a `sh:targetSubjectsOf` target declaration are the triples in the data graph with as subject the node and as predicate the specified predicate. (See [[[#ex-targetSubjectsOf]]].)
- The target triples of a focus node for a `sh:targetObjectsOf` target declaration are the triples in the data graph with as object the node and as predicate the specified predicate.
- The target triples of a focus node for a `sh:targetNode` target declaration is empty.

If a node is not in the [=target=] of a shape, there are no target triples of that focus node for that shape. 

<aside class="example" title="Example with target class" id="ex-targetClass">
<pre class="ex-input">
:Alice   a :Person .
:Bob     a :Teacher .
:Teacher rdfs:subclassOf :Person .
:NewYork a :City .
</pre>
<pre class="ex-shape">
:PersonShape a sh:NodeShape ;
  sh:targetClass :Person .
</pre>
<pre class="ex-output">
:Alice   a :Person .
:Bob     a :Teacher .
:Teacher rdfs:subclassOf :Person .
</pre>
</aside>
<aside class="example" title="Example with subjects-of target" id="ex-targetSubjectsOf">
<pre class="ex-input">
:Alice :knows   :Bob .
:Bob   :livesIn :NewYork .
</pre>
<pre class="ex-shape">
:TSOExampleShape a sh:NodeShape ;
  sh:targetSubjectsOf :knows .
</pre>
<pre class="ex-output">
:Alice :knows :Bob .
</pre>
</aside>

## Path triples 
Throughout this document, the term <dfn>path triples</dfn> will be used to refer to triples in the data graph that lay on a path. Path triples are defined by a [=property path=], a start node, and an end node. The path triples between a start and end node for a property path are exactly those triples in the data graph on paths that (i) start in the start node, (ii) match the property path and (iii) end in the end node.

<aside class="example" id="ex-simplePath" title="Simple path example">
The first triple, `:Alice :name "Alice"`, in the input lays on a path that (i) starts in focus node `:Alice`, (ii) ends in value node `"Alice"` and (iii) matches the simple property path `:name`. Therefore the triple `:Alice :name "Alice"` is present in the shape fragment for `:PersonShape`.
The second triple in the input is part of the path triples between focus node `:Bob` and value node `:bobsName`, but is not part of the shape fragment for `:PersonShape` because the value node does not satisfy the shape's constraints: it is not a literal. 
The third triple in the input is not part of the path triples between `:Carol` and `"Carol"`, since its predicate, `:hasName` does not match the shape's property path, `:name`.
<pre class="ex-input">
:Alice :name    "Alice" .
:Bob   :name    :bobsName .
:Carol :hasName "Carol" . 
</pre>
<pre class="ex-shape">
:PersonShape a sh:PropertyShape ;
  sh:targetNode :Alice, :Bob, :Carol ;
  sh:path       :name ;
  sh:nodeKind   sh:Literal .
</pre>
<pre class="ex-output">
:Alice :name "Alice" .
</pre>
</aside>

<aside class="example" id="ex-kleenePath">
Following example shape will extract a fragment containing the friends of `:Alice` and `:Xander`, as well as friends-of-friends, friends-of-friends-of-friends, and so on.
<pre class="ex-input">
:Alice  :friend :Bob .
:Bob    :friend :Carol . 
:Xander :likes  :Yolanda .
</pre>
<pre class="ex-shape">
:PersonShape a sh:PropertyShape ;
  sh:targetNode :Alice, :Xander .
  sh:path       [ sh:zeroOrMorePath :friend ] ;
  sh:minCount   1 .
</pre>
<pre class="ex-output">
:Alice :friend :Bob .
:Bob   :friend :Carol . 
</pre>
</aside>