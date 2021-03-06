# Remove

The `remove` command allows you to remove selected axioms from an ontology. The [`filter`](/filter) command is the opposite of `remove`, allowing you to keep only selected axioms. `remove` works in three steps:

1. `--term`: specify a set of terms (default: all terms, use `--term-file` for a file with terms line by line)
2. `--select`: select a new set of terms using one or more relations (zero or more)
3. `--axioms`: specify the axiom types to remove from those terms (default: all axioms)

This operation maintains structural integrity; lineage is maintained, and gaps will be filled where classes have been removed. If you wish to *not* preserve the hierarchy, include `--preserve-structure false`.

For example, to remove all descendants of 'assay' from OBI:

```
robot remove --input obi.owl --term OBI:0000070 --select descendants
```

`remove` also includes a `--trim` option, set to `true` by default. For an axiom to be removed, *one or more* of the entities in that axiom must be in the removal set. If `--trim false` is specified, *all* entities in the axiom must be in the selected set of terms. Dangling entities (entities without any axioms *about* them) will also be removed when `--trim` is `true`.

## Select

`--select` supports multiple options. These can be provided in quotes in one select statement (`--select "x y z"`), or in multiple statements (`--select x --select y --select z`). When selects are provided in multiple statements, the output of the first will be passed to the second and so on. If multiple options are provided in one statement, all options will be processed at the same time.

The following selection options can be combined in any way.

#### General

There are three general select options that give control over the types of axioms that are removed. By default, both `named` and `anonymous` entities are removed. But, for example, if only `--select anonymous` is provided, the named classes will not be removed.

1. `complement`: remove the complement set of the terms (equivalent to [filter](/filter), except that `remove` creates a new ontology whereas `filter` edits the input ontology).
2. `named`: remove named entities.
3. `anonymous`: remove anonymous entities (e.g. anonymous ancestors).
4. `ontology`: remove ontology annotations (for [filter](/filter), this returns just the ontology annotations)
5. `imports`: remove import statements and "dangling" references to imported terms (for [filter](/filter), this will copy the import declarations to the output ontology) **Warning:** If you wish to remove import statements but keep dangling references to imported terms, you must use `--trim false`.

#### Relation Types

Relation type selections provide the ability to select which terms to remove based on their relationship to the term (or terms) specified by `--term`/`--term-file`. If no relation type `--select` is provided, the default is `self`. This means that the term set itself will be removed.

1. `self` (default)
2. `parents`
3. `ancestors`
4. `children`
5. `descendants`
6. `equivalents`
7. `types`
8. `instances`
9. `domains`
10. `ranges`

#### Entity Types

If an entity type is provided, only the terms of that type will be included in the removal. By default, all types are included.

1. `classes`
2. `properties`
3. `object-properties`
4. `data-properties`
5. `annotation-properties`
6. `individuals`

### Patterns

Terms can also be selected from the set based on their annotations. This can be helpful if you want to remove only terms with a specific annotation. When selecting with axioms, the `--select` option must always be quoted.

1. `CURIE=CURIE`
    - e.g. `rdfs:seeAlso=obo:EX_123`
2. `CURIE=<IRI>`
    - e.g. `rdfs:seeAlso=<http://purl.obolibrary.org/obo/EX_123>`
3. `CURIE='literal'`
    - e.g. `rdfs:label='example label'`
4. `CURIE='literal'^^datatype`
    - e.g. `rdfs:label='example label'^^xsd:string`
5. `CURIE=~'regex pattern'`
    - e.g. `rdfs:label=~'example.*'`

It is also possible to select terms based on parts of their IRI. You may include an IRI pattern using one or more wildcard characters (`*` and/or `?`) enclosed in angle brackets. This MUST be enclosed in quotes to work on the command line.

- `<IRI-pattern>`
    - e.g. `<http://purl.obolibrary.org/obo/EX_*>`
    
If you wish to match a more complicated pattern, you may also use a regex pattern here by preceding the pattern with a tilde (`~`):
- `<~IRI-regex>`
    - e.g. `<~^.+EX_[0-9]{7}$>`

## Axioms

The `--axioms` option allows you to specify the type of OWLAxiom to remove. More than one type can be provided and the order is not significant. For each axiom in the ontology (not including its imports closure), if the axiom implements one of the specified axiom types AND *any* of the selected terms are in the axiom's signature, then the axiom is removed from the ontology.

1. `all` (default)
2. `logical`
3. `annotation`
4. `subclass`
5. `subproperty`
6. `equivalent` (classes and properties)
7. `disjoint` (classes and properties)
8. `type` (class assertions)
9. `tbox` (classes and class axioms)
10. `abox` (instances and instance-level axioms)
11. `rbox` (object properties, aka relations)

## Examples

Remove a class ('organ'), all its descendants, and any axioms using these classes:

    robot remove --input uberon_module.owl \
      --term UBERON:0000062 \
      --select "self descendants" \
      --output results/remove_class.owl

Remove all axioms containing an entity in the BFO namespace from the UBERON module:

    robot remove --input uberon_module.owl \
      --select "<http://purl.obolibrary.org/obo/BFO_*>" \
      --output results/remove_bfo.owl

Remove all anonymous entities from the UBERON module:

    robot remove --input uberon_module.owl \
      --select anonymous \
      --output results/remove_anonymous.owl

Remove all individuals from OBI:

```
robot remove --input obi.owl --select individuals 
```

Remove all deprecated classes from OBI:

```
robot remove --input obi.owl \
  --select "owl:deprecated='true'^^xsd:boolean" 
```

*Filter* for only desired annotation properties (in this case, label and ID). This works by actually *removing* the opposite set of annotation properties (complement annotation-properties) from the ontology:

    robot remove --input uberon_module.owl \
      --term rdfs:label --term oboInOwl:id  \
      --select complement --select annotation-properties \
      --output results/filter_annotations.owl
