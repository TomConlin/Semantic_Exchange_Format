# Bring me a Shrubbery!

## Structural Problem

Phenotype data lacks a well defined standard exchange format

Phenotype data is profoundly different than genotype data.
Bioinformatics already has a decent handle on genotype data.
Genotype data is basically ordered linear lists (DNA RNA genes proteins ...)
algorithms for working with lists are well behaved and
only incremental improvements may be expected in the general case.


Capturing Phenotypes requires a branching data structure to describe entities
and related features where the order of the relationships is strictly arbitrary. 

Although I am currently motivated by phenotypes in particular, the
thoughts in this document should be applicable to many other domains
requiring a graph/network representation. 


Graph algorithms are of a higher order of complexity than list algorithms,
some graph algorithm have no known efficient solutions including 
the simple sounding "_Is this graph within that graph?_"
(see: [Subgraph isomorphism problem](https://en.wikipedia.org/wiki/Subgraph_isomorphism_problem)

Short of a world changing breakthrough in Mathematics (see: [P versus NP problem](https://en.wikipedia.org/wiki/P_versus_NP_problem))
we need to limit the structure and content of our graph representations to ensure
that valid, simplifying, assumptions may be made by our algorithms if they are to scale.

The Computer Science background links included in this document are here
to convey the magnitude of the issue and necessity of these proposed constraints.    
However, one need not closely follow the math and jargon to appreciate that
neither seventy years of brilliant minds nor a million dollar prizes
have produced solutions to the fundamental problem,
which lends weight to the importance of  
__an exchange format that guarantees shortcuts are possible__.

  
The genotype landscape is rich in tools which may be combined in a myriad of ways
that consume and produce a few well defined standard formats: FASTA, FASTQ, VCF etc.
 
Before the phenotype tool landscape can become as rich,
well defined standard interchange formats which limit computational complexity are necessary.

--------------------------------------------------------------
## On Graph Structure
     
Any solution to the above problem will address the points below.

Phenotype information exchanges have different use cases
which have conflicting requirements. These include


 - mutability (merge, split, change)
 - accountability (attribution, provenance, history)
 - computability
 - expressiveness
 - aggregation

Mutability and accountability are vaguely opposites,
but from an open exchange format viewpoint, are similar to each other.

- Mutability allows exchanged artifacts to become more complete/correct over time.

- Accountability requires an exchanged artifact be preserved forever unchanged.


Proper [identifier discipline]() will help with both,
but those issues are in the domain of the repository using the open exchange format which only needs to avoid interfering with whatever policy the repository chooses without insisting all repositories adhere to the same policy.

Uniform metadata must be supported, but not uniformly required.


Computability and expressiveness are less nuanced
on one hand you have to be able to communicate something useful
and on the other hand every bit of flexibility in expression allowed reduces
the scale achievable.

So not allowing anything which is not necessary is the clear path.

Deciding what is necessary will be different and growing problem
but to begin with, the simplest thing that could possibly work is
what I am aiming for


From [Wikipedia](https://en.wikipedia.org/wiki/Graph_\(discrete_mathematics\))

```
...
a graph is a structure amounting to a set of objects
 in which some pairs of the objects are in some sense "related".
 The objects correspond to mathematical abstractions called vertices
 (also called nodes or points) and each of the related pairs of vertices
 is called an edge (also called an arc or line)
```

I am using the terms `nodes` and `edges` as the components which make a `graph`.

All graphs are not created equal and the more constraints a graph has
the easier it becomes to work with.

Several constraints to make graph processing easier

- Make edges one way (directed), this imposes a [partial order](https://en.wikipedia.org/wiki/Partially_ordered_set) to the nodes.

- No cycles of edges.
without this constraint when traversing a graph you need
to keep track every node you have visited and stop if you
see one twice because you are in an infinite loop.

(This next one is only quasi true:)
  
- Limit the number of directed edges coming into a node to at most one.
 
It is only quasi true because if we allow the same node
be shared by different graphs then, __when combined__,
there will be one incoming edge from each of the different graphs
which is what we want.

With these three constraints we have simplified graphs in general into
[directed rooted trees](https://en.wikipedia.org/wiki/Tree_\(graph_theory\))
which are far more computable than complete graphs and are still fairly expressive
especially when these trees are allowed to overlap  
(have nodes other than the root node in common),
because they form a
[directed acyclic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph)
which is often the sweet spot in complexity between modeling and computing.

Nodes in a tree are of three types,

- no edges coming in           (root node)
- edges coming in and going out (interior node)
- no edges going out            (leaf node)

Being a rooted tree guarantees exactly one node without incoming edges  
Interior nodes provide structure to determine how leaf nodes are grouped.  
Leaf nodes are used to carry (external) data


We can not know all of the leaf/data node values in advance,
but we can and must externally predefine, label and type every internal node and edge.


Explicitly noting here that
an internal node may only exist exactly once in a tree,
and leaf nodes which may be data, are outside our control may contain duplicate values. 

Directed rooted trees can also be seen as unordered lists of unordered lists.

These unordered lists could be quite arbitrary but if we limit ourselves
to a simple repeatable pattern they become more predictable which is important
when reading, writing and verifying.

The simplest structural pattern I believe could suffice is
one which could model of a deck of flash cards where
cards in the deck are generally related by having a similar theme.
and questions one the front of each card are exactly related to answers
on the back of the same card. This pattern forms a hierarchy four levels deep:
(deck, card, front, back).  
A deck may have many cards and each card many have many questions and answers.


- envelope
  - item_a
     - key_1
          - _value_
     - key_3
          - _value_
     - key_2
          - _value_
  - item_c        
     - key_1
          - _value_
     - key_2
          - _value_
     - key_3
          - _value_
  - item_b
     - key_2
         - _value_
     - key_3
         - _value_
     - key_1
         - _value_

where all the `items` (rdfs:Class) in a `envelope` (rdfs:Container)
share a set of `keys` (rdf:Property) which may be associated with _values_
which have an (rdf:type) or a datatype if they are `Plain Literal`.

depending on your background this may sound like:

 - a set of bijections
 - an array of dictionaries
 - a list of records
 - a trivial pursuit
 
Or a number of other phrases, many names for a useful concept.

This simple pattern is useful as a building block
in a more complex model when we allow a leaf (_value_) of one building block
the become the root (`envelope`) of another building block.

With the deck of flash cards analogy;  
The answer on one of the first deck's card would be to switch to a different deck.

The only strict rule is: a 'deck' may only appear exactly once
on any branch of the tree.
I suspect a more sensible rule is the generalization:
a deck should only exist once in a tree.


note: the generic terms `envelope` and `item` here are just stand in's
for the more loaded terms  
[collection, container, list, array, series ,sequence, set, bag, ... ]  
and
[class, record, map, dictionary, associative array, ... ]  
which through their use in practice may unintentionally imply
either more or less than I intend at the moment.

What is important is an outer construct loosely associating
inner constructs containing tight associations. 

A tree of four levels is relatively short,
computationally not too expensive and they look nice, they are shrubs.

## On Graph Meaning 

As each internal node needs to be unique within the tree,
a node could be given a pseudo random identifier/label based on the path to its root
but there is also the opportunity for

 - all non-leaf nodes
 - some leaf nodes
 - all edges
 
to have well established context.

If these predefined patterns are drawn from well established libraries
of patterns then graph fragments from unrelated sources may
overlap in a meaningful way.

A good source for the most generic of these predefined patterns is the
[W3C Resource Description Framework](https://en.wikipedia.org/wiki/Resource_Description_Framework)
which has been in use for over twenty years now and has most of the kinks worked out.  
As more specific patterns become necessary we must adopt or if necessary create
[ontologies](http://tomgruber.org/writing/ontology-definition-2007.htm) which
carefully describe the patterns we are expressing in a way that allows them
to meaningfully interact in a mechanistic way with other unknown patterns. 

Some core elements  of RDF which are applicable to every internal
node and many leaf nodes regardless of the domain are:  

 - `rdf:type`  which identifies the concept/resource in an external ontology
 - `rdfs:label`  for a human readable string for the concept/resource  
   (only a convenience, the ontology label takes precedence).
 - `rdfs:comment` optional for longer human readable descriptions.


Although the concepts thus far are broadly applicable
for Semantic Exchanges the use case I need to focus on is
biomedical phenotypes and related concepts including genotype, environments, substances etc.
so the ontologies I will refer to are mainly [Open Biomedical Ontologies](http://www.obofoundry.org/)
(OBO) 


## Summary

Semantic (sub)graph exchange is a thorny problem.
Limiting our exchange structure to a particular (fractal) tree pattern
can, when combined with others, result in a complex
but relativity tractable Directed Acyclic Graph.

Insisting our directed rooted trees are structured as a
"list of records" which allows the _value_ of a record's field
to be another "list of records" is a viable way of keeping exchanged
artifacts from being full graphs with their associated costs. 
This comes with the expense of sometimes needing to place
the same datum as a leaf in more than one branch or tree.
And, depending on the implementation,
may require circularity checks on collections which are values.

Leveraging the Semantic Web's definitions for common, higher level constructs,
and metadata standards 

--------------------------------------------------------------
# Phenopackets now

Abstractly a phenopacket is:    

A set of associated statments which conform to a preexisting structure.

## Concretely  a phenopacket is

A versioned fragment of a knowledge graph structured according
to semantic web and resource description framework (RDF) constraints. 

Addional constrains limit the structure of each fragment to
a directed graph with a single root (a tree) 

## Practical considerations
   ...


## Currently phenopacket implementation:
 
A hierarchical record consisting of a container 
with a human readable label (title) and a unique identifier
 
<PFX:ppid> __<rdf:type>__ < PHENOPACKET_URL> .  
<PFX:ppid> __<rdfs:label>__ "Phenopackert Name"::string .  
 
with one or more sections drawn from these catagories:
  
 - "entities"  
        <PFX:entity_id> <RO:part_of> <PFX:ppid> .
        <PFX:entity_id> <rdfs:label> "entity label" .  
        ...

 - "variants"  
        <PFX:variant_id> <RO:part_of> <PFX:ppid> .
        <PFX:variant_id> <rdfs:label> "variant label".
        <PFX:variant_id> <HGVS:description> "hgvs_string".  
        ...

 - "persons"  
        <PFX:person_id> <RO:part_of> <PFX:ppid> .
        <PFX:person_id> <PFX:has_strain> <PFX:strain_id> .
        <PFX:strain_id> <rdfs:label> "strain label".                # secondary label
        <PFX:person_id> <PFX:positive_instance> <PFX:onto_class> . 
            ...
        
        <PFX:person_id> <PFX:negative_instance> <PFX:onto_class> .
            ...
        <PFX:person_id> <rdfs:description> "optional description" .
        <PFX:person_id> <rdfs:label> "person label".
        <PFX:person_id> <inTaxon> <NCBITaxon:xxx>  .  # really? expecting police dogs and corporations maybe?
        <PFX:person_id> <PFX:sex> "code for the biological sex" .
        <PFX:person_id> <>"date_of_birth" .
        ...


 - "organisms"

 - "phenotype_profile"

 - "diagnosis_profile"

 - "environment_profile"

 
-----------------------------------------------------------

(  
 Does order matter?  
 Should categories be lexically ordered?  
 Can there be repeats?  
 What about empty sections?  
) 


 
#### To get the structure from the json-schemsa file into a tab indented tree
    jq . phenopacket-schema.json | cut -f1 -d ':' | grep '[a-z$"_]' | sed 's/  /\t/g;s/^\t//g' > phenopacket_schema_structure.tab



#### To get the named sections of records
    grep '"id" : "urn:jsonschema:org:phenopackets:api' phenopacket-schema.json | cut -f7- -d':' | tr -d ',"'

PhenoPacket
model:entity:Entity
model:entity:Variant
model:entity:                            # (miss used)
model:ontology:OntologyClass             # reused
model:entity:Organism
model:association:PhenotypeAssociation
model:condition:Phenotype
model:condition:Measurement
model:condition:OrganismalSite           # reused
model:condition:TemporalRegion
model:condition:ConditionSeverity        # reused
model:environment:Environment            # reused
model:meta:Evidence                      # reused
model:meta:Publication
model:association:DiseaseOccurrenceAssociation
model:condition:DiseaseOccurrence
model:condition:DiseaseStage
model:association:EnvironmentAssociation


# to get the reused sections
 
grep '"$ref" : "urn:jsonschema:org:phenopackets:api' phenopacket-schema.json | cut -f7- -d':' | tr -d ',"' | sort -u

model:condition:ConditionSeverity
model:condition:OrganismalSite
model:condition:TemporalRegion
model:environment:Environment
model:meta:Evidence
model:ontology:OntologyClass


# to get the relations

egrep '"(id|\$ref)" : "urn:' phenopacket-schema.json | uniq | sed 's/"id"//g;s/"\$ref"//g;s/: "urn:jsonschema:org:phenopackets:api://g;s/  /\t/g;s/^\t//g' | tr -d '",' > pheno_packet_class.tab

# collapse dup sibbling classes 
# add in a blank node where indentation more than one tab greater than predecessor

pheno_packet_class_bnode.tab
################################################



python2 ./outline_to_dot.py --tree=True  pheno_packet_class_bnode.tab > pheno_packet_class_bnode.gv
