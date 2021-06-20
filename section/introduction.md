## Introduction {#introduction}

This document interprets SHACL [[SHACL]] as a subgraph extraction language. We call the extracted subgraphs shape fragments, or shape graph fragments if they are defined by multiple shapes. A shape fragment is defined as the union of the neighborhoods of conforming target nodes. A **neighborhood of a node** is the part of a data graph that is needed for the node to conform to a shape. Neighborhoods are defined differently for different kinds of constraint.

A shapes graph fragment has the property that it conforms to the shapes graph, given that the data graph initially conformed to the shapes. In a sense, a shape fragment is the minimal subgraph that conforms to the shapes while still containing all conforming target nodes.
