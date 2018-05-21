# Bring me a Shrubbery!

  
_note: this is a WIP seeking simplest thing which could work_  
_hence: may still be too simple at a given point_  
 
## Structural Problem

Semantic graph exchange has a low level format in Resource Description Format
([RDF](https://en.wikipedia.org/wiki/Resource_Description_Framework))
which communicates at the level of __edges__ which allow any graph to be
represented.
This includes graphs which are nonsense or intractable.

Semantic graph exchange has a high level format in Web Ontology Language
([OWL](https://en.wikipedia.org/wiki/Web_Ontology_Language)) which is
a formal, committee specified [Description Logic](https://en.wikipedia.org/wiki/Description_logic)
language for representing anything logically describable.
That is OWL communicates the full conceptual graph and
what may be done with it simultaneously.
When implemented correctly it works remarkably well,
allowing mechanical reasoning humans could never cover as well.
But even at its most tractable, an OWL file benefits from special
[tooling](https://protege.stanford.edu/) to author or even view. 


What is lacking is an intermediate scale structure between "RDF's just edges"
and the superposition of all possible graph fragments from your model.
A structure that fits trivially in the users mind with room to spare for
the problem they are actually trying to solve. A simple composable structure
which can be vetted in isolation before being assembled with any number of
similar structures where again the structure of structures may be incrementally
sanity checked.


Before we go on, I am not debating the merits or suitability of the various
transport formats or serializing protocols. I don't care, whatever wins
is fine by me, switching between encodings is a solved problem. 

I am however wanting to impose constraints on the big picture,
eliminating graphs more complex than directed acyclic from consideration,
and insisting graphs are built up from a single repeated tree pattern.  
My reasons follow. 

Phenotype data is profoundly different than genotype data.
Bioinformatics already has a decent handle on genotype data.
Genotype data is basically ordered linear lists (DNA RNA genes proteins ...)
algorithms for working with ordered lists are tractable and well behaved.
Only incremental improvements may be expected in the general case.

Capturing Phenotypes requires a branching data structure to describe entities
and related features where the order between the relationships is strictly
arbitrary.

Although I am motivated by phenotypes in particular, my hope is this document
may be applicable to other domains requiring a graph/network representation.

Graph algorithms are of a higher order of complexity than list algorithms,
some graph algorithm have no known efficient solutions including 
the simple sounding "_Is this graph within that graph?_"
(see: [Subgraph isomorphism problem](https://en.wikipedia.org/wiki/Subgraph_isomorphism_problem))

Short of a world changing breakthrough in Mathematics
(see: [P versus NP problem](https://en.wikipedia.org/wiki/P_versus_NP_problem))
we need to limit the structure and content of our graph representations to ensure
that valid, simplifying, assumptions may be made by our algorithms
if they are to scale along with genomic sequence data.

The Computer Science background links included in this document are here to
convey the magnitude of the issue and necessity of the proposed constraints.
However, one need not closely follow the math and jargon to appreciate that
neither fifty years of brilliant minds nor million dollar prizes have
produced solutions to the fundamental problems,
which lends weight to the importance of  
__an exchange format that guarantees shortcuts are possible__.
  
The genotype landscape is rich in tools which may be combined
in a myriad of ways, each consume and produce a few well defined
standard formats: FASTA, FASTQ, VCF etc.
 
Before the phenotype tool landscape can become as rich, well defined standard
interchange formats which limit computational complexity are necessary.

--------------------------------------------------------------

## On Graph Structure

Computability and expressiveness are less nuanced than the policy issues to come.
On one hand you have to be able to communicate something useful
and on the other hand every bit of flexibility in expression allowed reduces
the scale achievable.

So not allowing anything which is not necessary is the clear path.

Deciding what is necessary will be different and growing problem
but to begin with, the simplest thing that could possibly work is
what I am aiming for.


From [Wikipedia](https://en.wikipedia.org/wiki/Graph_\(discrete_mathematics\))

```
...
a graph is a structure amounting to a set of objects
 in which some pairs of the objects are in some sense "related".
 The objects correspond to mathematical abstractions called vertices
 (also called nodes or points) and each of the related pairs of vertices
 is called an edge (also called an arc or line)
```

I am using the terms `nodes` and `edges` as the components which make a `graph`,
and assuming no degenerate cases such as disconnected nodes or edges.


All graphs are not created equal, and with the right constraints a graph becomes
much easier to work with.

Several constraints to make graph processing easier:

- Make edges one way (directed), this imposes a [partial order](https://en.wikipedia.org/wiki/Partially_ordered_set) to the nodes.

- No cycles of edges.
without this constraint when traversing a graph you need
to keep track every node you have visited and stop if you
see one twice because you are in an infinite loop. 

(This next one is only quasi true:)
  
- Limit the number of directed edges coming into a node to at most one.
 
It is only quasi true because if we allow the same node
be shared by different graphs then, __when combined__,
there may be one incoming edge from each of the different graphs
the node belongs to, which is exactly what we want.

With these three constraints we have simplified general graphs into
[directed rooted trees](https://en.wikipedia.org/wiki/Tree_\(graph_theory\))
which are far more computable than complete graphs and still fairly expressive
especially when these trees are allowed to overlap  
(have nodes other than the root node in common),
because they form a
[directed acyclic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph)
which is often the sweet spot in complexity between modeling and computing.

Note: a `tree` _is_  a `graph` just one that is constrained to have exactly
one fewer edges than nodes. So as we talk about trees, we are just talking about
deliberately simplified graphs. 

Nodes in a tree are of three types,

- no edges coming in            (root node)
- edges coming in and going out (interior node)
- no edges going out            (leaf node)

Being a rooted tree guarantees exactly one node without incoming edges  
Interior nodes provide structure to determine how leaf nodes are grouped.  
Leaf nodes are used to carry external data and types. 


We can not know all of the leaf/data node values in advance,
but we can and must externally predefine,
label and type every internal node and edge.

Directed rooted trees can also be seen as unordered lists of unordered lists.

These unordered lists could be quite arbitrary but if we limit ourselves
to a simple repeatable pattern they become more predictable which is important
for ease and efficiency when reading, writing and verifying.

The simplest structural pattern I believe could suffice is
one which could model of a deck of flash cards where
cards in the deck are generally related by having a similar theme.
and questions one the front of each card are exactly related to answers
on the back of the same card. This pattern forms a hierarchy four-ish levels deep:
deck, card, front_question, back_answer,  (and possibly types, think color coded).
A deck may have many cards and each card many have many questions and answers.


- Deck
  - Card_a
     - Question_1
          - _Answer1_
     - Question_3
          - _Answer3_
     - Question_2
          - _Answer2_
  - Card_c        
     - Question_1
          - _Answer1_
     - Question_2
          - _Answer2_
     - Question_3
          - _Answer2_
  - Card_b
     - Question_2
         - _Answer2_
     - Question_3
         - _Answer3_
     - Question_1
         - _Answer1_

where the `Card`s  in a `Deck` may share `Question`s which are
associated with _Answers_.  
In practice  I expect decks and perhaps cards
may have attendent fields such as  
 - ID  
 - type  
 - label  

Although not explicitly shown here _Answers_ should be typeable.


Depending on your background this may sound like:

 - a set of bijections
 - an array of structures
 - a list of records
 - a trivial pursuit deck
 
Or any number of other phrases, many names for a useful concept.

This simple pattern is useful as a building block
in a more complex model when we allow a leaf (_Answer_) of one building block
the be the root (`Deck`) of another building block.
(Note: we do not allow new decks to appear anywhere but as a root or leaf)

With the deck of flash cards analogy;  
The answer on one of the first Deck's card would be to switch to a different deck.

The only strict rule would be: a 'Deck' may only appear exactly once
on any branch of the tree.
I suspect a more sensible rule is the generalization:
that a deck should only exist once in given a tree.


note: the generic terms `Deck` & `Card` are just stand ins
for more loaded terms such as  
[collection, container, list, array, series ,sequence, set, bag, ... ]  
and
[class, record, map, dictionary, associative array, ... ]  
which through their use in practice may unintentionally imply
either more or less than I intend at the moment.

What is important is an outer construct loosely associating
inner constructs containing tight associations. 


## On Graph Meaning

 
As each internal node needs to be uniquely identified within the tree,
a node is easily be given a pseudo random identifier/label
based on the path from the root,
but there is also the opportunity for

 - all non-leaf nodes
 - some leaf nodes
 - all edges
 
to have well established context.
That is: everything which is not external data comes from and with
globally shared, commonly accepted, referable, predefined, definitions,
and usage patterns.   

If these predefined patterns are drawn from well established libraries
of patterns then graph fragments from unrelated sources may
overlap in a meaningful way despite the data having nothing specific in common.

A good source for the most generic of these predefined patterns is the
[W3C Resource Description Framework](https://en.wikipedia.org/wiki/Resource_Description_Framework)
which has been in use for over twenty years now and has most of the kinks worked out.  
As more specific patterns become necessary we must adopt or if necessary create
[ontologies](http://tomgruber.org/writing/ontology-definition-2007.htm) which
carefully describe the patterns we are expressing in a way that allows them
to meaningfully interact in a mechanistic way with other unknown patterns. 

Some core elements of RDF which are applicable to every internal
node and many leaf nodes regardless of the domain are:  

 - `rdf:type`  which identifies the concept/resource in an external ontology
 - `rdfs:label` (convenience) short human readable string for the concept/resource  
 - `rdfs:comment` optional for longer human readable descriptions.


Although the concepts thus far are broadly applicable
for Semantic Exchanges the use case I need to focus on is
biomedical phenotypes and related concepts including genotype,
environments, substances etc.
so the ontologies I will refer to are mainly from
[Open Biomedical Ontologies](http://www.obofoundry.org/)(OBO) 


## On Paths

A `path` in a graph is an alternating sequence of nodes and edges.
In the directed rooted trees proposed here, there is
exactly one path from the root node to each and every leaf node.
Furthermore, unlike these short trees themselves which are
unordered lists of unordered lists,
the set of all paths in a tree is
a single unordered list of distinctly ordered lists, where the
order comes from the constraint of using directed edges.  

A collection of trees (forest) may have their sets of paths
(ordered lists) all collected together.

An answer to a question may be seen as finding a way through
the forest that begins with the node you have and ends with
the node you want.

Obvious process is scan the collection of paths for nodes
which are in the set of what we have, then  
for each path found, if the node we want is in the path,
we have found a 'way' to an answer

If not, nodes in the path (sans nodes seen) become
the set of nodes we have, and the process repeats.

If we run out of nodes, then we do not have an answer.

This of course is the simplest case, more likely
there are constraints on the direction nodes in a path are found,
and more expensive, may only except a path if a valid DAG
can be assembled including the trees the 'way' is found within.


## Metadata Policy 

Intertwined with structural/mathematical issues are important
considerations which require accommodation in practice.

Metadata issues surrounding history, responsibility, permissions etc.
may vary between non-existent in the case of student experimenting
and critical in a reputable clinical repository. 
 
Information exchanges have different use cases
which have conflicting requirements.
These may (or may not) include:  


 - mutability (merge, split, change)
 - accountability (attribution, provenance, history)
 - computability
 - expressiveness
 - aggregation

Mutability and accountability are vaguely opposites,
but from an open exchange format viewpoint, are similar to each other.

- Mutability allows exchanged artifacts to become more complete/correct over time.

- Accountability requires an exchanged artifact be preserved forever unchanged.


Proper [identifier discipline](http://journals.plos.org/plosbiology/article?id=10.1371/journal.pbio.2001414) will help with both,
but those issues are in the domain of the repository using the
open exchange format which only needs to avoid interfering with
whatever policy the repository chooses without insisting all repositories
adhere to the same policy.

Uniform metadata must be supported, but not uniformly required.


## Theory

A proponent of [Catagory Theory](https://en.wikipedia.org/wiki/Category_theory)
stated that as outlined to them, these structures maintain
the key properties of [OLOG](https://en.wikipedia.org/wiki/Olog)s.
Which would be reasuring conclusion having approached it from a purely pratical 
point of view.

## Summary

Semantic (sub)graph exchange is a thorny problem.
Limiting our exchange structure to a particular (composable) tree pattern
can, when combined with others, result in a complex
but relativity tractable Directed Acyclic Graphs.

Insisting our directed rooted trees are structured as a
"list of records" which allows the typed _value_ of a record's field
to be another "list of records" is a viable way of keeping exchanged
artifacts from being full graphs with their associated costs. 
This comes with the expense of sometimes needing to place
the same datum in more than one branch or tree.
And, depending on the implementation,
may require circularity checks on values which are themselves new collections.

Leveraging existing Semantic Web's definitions to provide context for common,
higher level constructs, and to provide a standard metadata format is prudent
and enables an immediate base level of interoperability.

Trees may be linearized into paths which are ordered sequences of nodes
which may be operated on with some of the same algorithms
bioinformatics developed for various other sequences.

A tree of only four-ish levels is pretty short,
computationally not too expensive, and they look nice,
they are shrubs.
