---
name: brick-ontology-lookup
description: Fetch, pin and reason over the Brick ontology itself — the classes, relationships and SHACL shapes that give a building model meaning.
api: Brick Ontology
generated: '2026-09-04'
method: generated
source: vocabulary/brick-vocabulary.yml ; packages/brick-packages.yml ; changelog/brick-changelog.yml
operations: []
---

# Look up and reason over the Brick ontology

Brick is a downloadable graph, not a hosted service. There is no API to call, no key to obtain and no
rate limit. This skill is how an agent gets the vocabulary right.

## Pin a version

The namespace is deliberately unversioned — `https://brickschema.org/schema/Brick#` — and has been
since v1.2. Version is carried by what you **import**, not by the term IRIs. Pin explicitly:

- Current stable: `https://brickschema.org/schema/1.4.4/Brick.ttl` (released 2025-05-02, 1.7 MB
  Turtle) or `https://brickschema.org/schema/1.4.4/Brick.jsonld`.
- Minor line alias: `https://brickschema.org/schema/1.4/Brick.ttl`.
- Nightly: `https://github.com/BrickSchema/Brick/releases/download/nightly/Brick.ttl` — moves under
  you; never pin an agent to it.
- A 1.5.0 release candidate exists (2026-06-09) and is not stable.

There is **no content negotiation**: requesting `https://brickschema.org/schema/Brick` returns the
same RDF/XML body whatever `Accept` header you send. Ask for the file extension you want.

Which distribution: `Brick+imports.ttl` is "recommended for end-applications", `Brick.ttl` "for
platforms". `Brick-only.ttl` excludes the vendored external ontologies.

## Know the shape before you query

Brick 1.4.4 defines 1,428 OWL classes and 79 object properties. Eight root classes sit under
`brick:Entity`:

- `brick:Point` (935 subclasses) — telemetry sources and control targets. Explicitly disjoint from
  Equipment, Location, Substance, Quantity, Collection and EntityProperty.
- `brick:Equipment` (330) — "devices that serve all or part of the building".
- `brick:Location` (108) — site, building, floor, room, zone.
- `brick:Collection` (44), `brick:Measurable`, `brick:Tag`, `brick:EntityPropertyValue`, `brick:Class`.

Relationships come in inverse pairs — traverse either direction:
`hasPoint`/`isPointOf`, `hasPart`/`isPartOf`, `feeds`/`isFedBy`, `hasLocation`/`isLocationOf`,
`meters`/`isMeteredBy`. Also `hasTag`, `hasUnit` (a QUDT unit), `hasQUDTReference`,
`hasInputSubstance`/`hasOutputSubstance`, and `isReplacedBy` — Brick's in-graph succession pointer for
retired or renamed terms. **Follow `isReplacedBy` before reporting a term as missing.**

## Work with it locally

```
pip install brickschema
```

```python
from brickschema import Graph
g = Graph(load_brick=True)
g.load_file("myBuilding.ttl")
g.expand(profile="owlrl")     # or "rdfs", "shacl", "vbis"
```

- Validate: `brick_validate myBuilding.ttl` — checks against the Brick SHACL shapes.
- Convert an existing source: `pip install brickschema[brickify]` then
  `brickify SOURCE --input-type haystack-v4` (also `rac`, `table`, `rdf`; xls/csv/tsv/RDF formats).
- Explore interactively: `g.serve("localhost:8080")` starts a local Yasgui SPARQL interface
  (needs the `[web]` extra). Localhost only.

## Cross-vocabulary

Brick declares these in its own graph, so a term you need may already be aligned: ASHRAE 223P
(`s223:`), RealEstateCore (`rec:`, vendored into the distribution), QUDT (`qudt:`, units), SOSA,
SKOS, ASHRAE 135 BACnet (`bacnet:`, external references), plus published alignments for W3C LBD BOT
and VBIS.

## Reference

- Ontology browser, class by class: https://ontology.brickschema.org/
- Documentation: https://docs.brickschema.org/intro.html
- Every release ships a generated list of added and removed classes:
  https://github.com/BrickSchema/Brick/releases
