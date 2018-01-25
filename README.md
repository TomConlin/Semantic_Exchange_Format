# Tom's long diatribe.  Read

## Structural Problem

Phenotype data lacks a well defined standard exchange format

Phenotype data is profoundly different than geneotype data.
Bioinformatics already has a decent handle on genotype data.
Genotype data is basicly ordered linear lists (DNA RNA genes protiens ...)
algorithms for working with lists are well behaved and
only incremental improvments may be expected in the general case.


Capturing Phenotypes requires a branching data structure to describe entities
and related features where the order of the relationships is strictly arbitrary. 

Although I am currently motivated by phenotypes in particular, the
thoughts in this document should be applicable to many other domains
requireing a graph/network representation. 


Graph algorithms are of a higher order of complexity than list algorithms,
some graph algorithm have no known efficent solutions including 
the simple sounding "_Is this graph within that graph?_"
(see: [Subgraph isomorphism problem](https://en.wikipedia.org/wiki/Subgraph_isomorphism_problem)

Short of a world changing breakthrough in Mathematics (see: [P versus NP problem](https://en.wikipedia.org/wiki/P_versus_NP_problem))
we need to limit the structure and content of our graph representations to ensure
that valid, simplifying, assumptions may be made by our algorithms if they are to scale.

The Computer Science background links included in this document are here
to convey the magnitude of the issue and how unfrivolous proposed constraints are.   
However one need not follow the math and jargon to appreicate that
neither seventy years of trying nor a million dollar prize
have produced solutions to the fundamental problem,
which lends weight to the importance of  
__an exchange format that guarentees shortcuts are possible__.

  
The genotype landscape is rich in tools which may be combined in a myriad of ways
that consume and produce a few well defined standard formats: FASTA, FASTQ, VCF etc.
 
Before the phenotype tool landscape can become as rich,
well defined standard interchange formats which limit computational complexity are necessary.

--------------------------------------------------------------
## Graph Structure Discussion
     
Any solution to the above problem will address the points below.

Phenotype information exchanges have different use cases
which have conflicting requirements. These include


 - mutability (merge, split, change)
 - accountability (attribution, provanance, history)
 - computability
 - expressiveness
 - aggregation

Mutability and accountability are vaguely opposites,
but from an open exchange format viewpoint, are similar to each other.

- Mutability allows exchanges to become more complete/correct over time.

- Accountability requires an exchange is preserved forever unchanged.


Good identifiers and versioning will help with both,
but those issues are in the domain of the repository using the open exchange format
which only needs to avoid interfering with whatever policy the repository
chooses without insisting all repositories adhere to the same policy

Uniform metadata must be supported, but not uniformly required.


Computability and expressiveness are the less nuanced
on one hand you have to be able to communicate something useful
and on the other hand every bit of flexibility allowed reduces
the scale achievable (guarenteed).

So not allowing anything which is not necessary is the clear path.

Deciding what is necessary will be different and growing problem
but to begin with, anything is better than nothing.


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
see one twice because you are in an infinate loop.

(This next one is only quasi true:)
  
- Limit the number of directed edges coming into a node to at most one.
 
It is only quasi true because if we allow the same node
be shared by different graphs then, __when combined__,
there will be one incomming edge from each of the different graphs
which is what we want.

With these three constraints we have simplified graphs in general into
[directed rooted trees](https://en.wikipedia.org/wiki/Tree_\(graph_theory\))
which are far more computible than graphs and are still fairly expressive
especially when these trees are allowed to overlap  
(have nodes other than the root node in common),
because they form a
[directed acyclic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph)
which is often the sweet spot between modeling and computing.

Nodes in a tree are of three types,

- no edges comming in           (root node)
- edges coming in and going out (interior node)
- no edges going out            (leaf node)

Being a rooted tree guarentees exactly one node without incomming edges  
Interior nodes provide structure to determine how leaf nodes are grouped.  
Leaf nodes are used to carry (external) data


We can not know the leaf/data node values in advancce,
but we must be able to predefine/label/type every internal node allowable.

Explicitly noting here that
an internal node may only exist exactly once in a tree,
and leaf nodes being data, are outside our control may contain duplicate values. 

Directed rooted trees can also be seen as unordered lists of unordered lists.

Practically speaking, at least some of these unordered lists will have common
stuctural patterns.

- collection
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

where all the `items` (rdfs:Class) in a `collection` (rdfs:Bag)
share a set of `key`s (rdf:Property) which may be given different _values_
which have an (rdf:type).

depending on your background this may sound like:

 - a set of bijections
 - an array of dictionaries
 - a list of records

Or a number of other phrases, many names for a useful concept.


## Discussion of Graph meaning 

With each internal node needing to be unique, a node could just be
given a pseudo random identifier/label based on its path to the root
but there is also the oppertunity for every internal node
to corespond with a predefined pattern.

If these predefined patterns are drawn from well estabilished libraries
of pattern then graph fragments from unrelated sources have some chance of
matching up in a meaningful way.
A good source for the more generic of these predefined patterns is the
[W3C Resource Description Framework](https://en.wikipedia.org/wiki/Resource_Description_Framework)
which has been in use for over twenty years now and has most of the kinks worked out.  
As more specific patterns become absoultley necessary we must create or adopt
[ontologies](http://tomgruber.org/writing/ontology-definition-2007.htm) which
carefully describe the patterns we are expressing in a way that allows 




Althought I do hope the concepts here are more broadly applicacable
for Semantic Exchanges the use case I neeed to focus in on is

biomedical phenotypes and related concepts.
Some core RDF concepts which can be applicacable to every internal node regardless of the domain
are:

 - `rdf:type` which identifies the concept/resourse in the specified external ontology
 - `rdfs:label` for a human readable string for the concept/resource  
   (a conveniance, ontology label takes precedence).
 - `rdfs:comment` optional for longer human readable descriptions
 

--------------------------------------------------------------
## Abstractly a phenopacket is  

A set of associated statments which conform to a prexisting structure.

## Concretely  a phenopacket is

A versioned fragment of a knowlege graph structured according
to semantic web and resource description framework (RDF) constraints. 

Addional constrains limit the structure of each fragment to
a directed graph with a single root (typicallt a tree 

## Practical considerations
   ...


## Currently a phenopacket is:
 
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


( 
 Does order matter? 
 Should catagories be lexicaly ordered? 
 Can there be repeats? 
 What about empty sections?
 Can there be repetes? 
 Is it a set?
) 


 
#### To get the structure from the json-schemsa file into a tab indented tree
    jq . phenopacket-schema.json | cut -f1 -d ':' | grep '[a-z$"_]' | sed 's/  /\t/g;s/^\t//g' > phenopacket_schema_structure.tab



#### To get the named sections of records
    grep '"id" : "urn:jsonschema:org:phenopackets:api' phenopacket-schema.json | cut -f7- -d':' | tr -d ',"'

PhenoPacket
model:entity:Entity
model:entity:Variant
model:entity:Person
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
