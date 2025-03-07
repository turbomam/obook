# Getting started with DOSDP templates

Dead Simple OWL Design patterns (DOSDP) is a templating system for documenting and generating new OWL classes. The templates themselves are designed to be human readable and easy to author. Separate tables (TSV files) are used to specify individual classes.

The complete DOSDP documentation can be found here http://incatools.github.io/dead_simple_owl_design_patterns/.

For another DOSDP tutorial see [here](dosdp-template.md).

## Anatomy of a DOSDP file:

A DOSDP tempaltes are written in [YAML](<[url](https://en.wikipedia.org/wiki/YAML)>) file, an easily editable format for encoding nested data structures. At the top level of nesting is a set of 'keys', which must match those specified in the DOSDP standard. The various types of key and their function are outlined below. Each key is followed by a colon and then a value, which may be a text string, a list or another set of keys. Lists items are indicated using a '-'. Nesting is achieved via indenting using some standard number of spaces (typically 3 or 4). Here's a little illustration:

```yaml
key1: some text
key2:
  - first list item (text; note the indent)
  - second list item
key3:
  key_under_key3: some text
  another_key_under_key3:
    - first list item (text; note the indent)
    - second list item
  yet_another_key_under_key3:
    key_under_yet_another_key_under_key3: some more text
```

In the following text, keys and values together are sometimes referred to as 'fields'.

### Pattern level keys

[Reference doc](https://github.com/INCATools/dead_simple_owl_design_patterns/blob/master/docs/dosdp_schema.md#properties)

A set of fields that specify general information about a pattern: name, description, IRI, contributors, examples etc

e.g.

```yaml
pattern_name: abnormalAnatomicalEntity
pattern_iri: http://purl.obolibrary.org/obo/upheno/patterns/abnormalAnatomicalEntity.yaml
description: "Any unspecified abnormality of an anatomical entity."

contributors:
  - https://orcid.org/0000-0002-9900-7880
```

### Dictionaries

[Reference doc](https://github.com/INCATools/dead_simple_owl_design_patterns/blob/master/docs/dosdp_schema.md#owl-entity-dictionaries)

A major aim of the DOSDP system is to produce self-contained, human-readable templates. Templates need IDs in order to be reliably used programatically, but templates that only use IDs are not human readable. DOSDPs therefore include a set of dictionaries that map labels to IDs. Strictly any readable name can be used, but by convention we use class labels. IDs must be OBO curie style e.g. CL:0000001).

Separate dictionaries are required for classes, relations (object properties) & annotationProperties

e.g.

```yaml
classes:
  quality: PATO:0000001
  abnormal: PATO:0000460
  anatomical entity: UBERON:0001062

relations:
  inheres_in_part_of: RO:0002314
  has_modifier: RO:0002573
  has_part: BFO:0000051
```

### Variables

[Reference doc](https://github.com/INCATools/dead_simple_owl_design_patterns/blob/master/docs/dosdp_schema.md#var-types)

These fields specify the names of pattern variables (TSV column names) and map these to a range. e.g. This specifies a variable called 'anatomy' with the range 'anatomical entity':

```yaml
vars:
  anatomy: "'anatomical entity'"
```

The var name (anatomy) corresponds to a column name in the table (TSV file) used in combination with this template, to generate new terms based on the template. The range specifies what type of term is allowed in this column - in this case 'anatomical entity' (UBERON:0001062; as specified in the dictionary) or one of its subclasses, e.g.-

| anatomy        |
| -------------- |
| UBERON:0001154 |

There are various [types of variables](https://github.com/INCATools/dead_simple_owl_design_patterns/blob/master/docs/dosdp_schema.md#var-types):

`vars` are used to specify OWL classes (see example above). data_vars and data_list_vars are used to specify single pieces or data lists respectively. The range of data_vars is specified using XSD types. e.g.

```yaml
data_vars:
  number: xsd:int

data_list_vars:
  xrefs: xsd:string
```

A table used to specify classes following this pattern could have the following content. Note that in lists, multiple elements are separated by a '|'.

| number | xrefs                                         |
| ------ | --------------------------------------------- |
| 1      | pubmed:123456\|DOI:10.1016/j.cell.2016.07.054 |

### Template fields

Template fields are where the content of classes produced by the template is specified. These mostly follow [printf format](https://en.wikipedia.org/wiki/Printf_format_string): A `text` field has variable slots specified using %s (for strings), %d for integers and %f for floats (decimals). Variables slots are filled, in order of appearance in the text, with values coming from a list of variables in an associated `vars` field e.g.

```yaml
name:
  text: "%s of %s"
  vars:
    - neuron
    - brain_region
```

If the value associated with the neuron var is (the class) 'glutamatergic neuron' and the value associated with the = 'brain region' var is 'primary motor cortext', this will generate a classes with the name (label) "glutamatergic neuron of primary motor cortex".

#### OBO fields

[Reference doc](https://github.com/INCATools/dead_simple_owl_design_patterns/blob/master/docs/dosdp_schema.md#obo-fields)

DOSDPs include a set of convenience fields for annotation of classes that follow OBO conventions for field names and their mappings to OWL annotation properties. These include `name`, `def`, `comment`, `namespace`. When the value of a var is an OWL class, the name (label) of the var is used in the substitution. (see example above).

The annotation axioms generated by these template fields can be [annotated](). One OBO field exists for this purpose: `xrefs` allows annotation with a list of references using the obo standard xref annotation property (curies)

e.g.

```yaml
data_list_vars:
  xrefs: xsd:string

def:
  text: "Any %s that has a soma located in the %s"
  vars:
    - neuron
    - brain_region
  xrefs: xrefs
```

#### Logical axioms convenience fields

[Reference doc](https://github.com/INCATools/dead_simple_owl_design_patterns/blob/master/docs/dosdp_schema.md#logical-convenience-fields)

Where a single equivalent Class, subclassOf or GCI axiom is specified, you may use the keys 'EquivalentTo', 'subClassOf' or 'GCI' respectively. If multiple axioms of any type are needed, use the core field `logical_axioms`.

### Core fields

```yaml
annotations:
  - annotationProperty:
    text:
    vars:
    annotations: ...
  - annotationProperty:
    text:
    vars:

logical_axioms:
  - axiom_type: subClassOf
    text:
    vars:
      -
      -
  - axiom_type: subClassOf
    text:
    vars:
      -
      -
    annotations:
      - ...
```

### Advanced usage:

#### Optionals and multiples (0-many)

TBA

### Using DOSDP templates in ODK Workflows

The Ontology Development Kit (ODK) comes with a few pre-configured workflows involving DOSDP templates. For a detailed tutorial see [here](dosdp-odk.md).
