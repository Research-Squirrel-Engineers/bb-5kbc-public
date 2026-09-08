# bb-5kbc-public

The **BB-5KBC** dataset prepared for the [NFDI4Objects Knowledge Graph][n4o]:
some 540 archaeological sites in Brandenburg, East Germany and West Poland
dating to around 5000 BC.

This repository publishes the dataset; it does not produce it. The RDF is built
in [bb-5kbc-sites][source] and archived here as the exact version that was
loaded into the graph.

[n4o]: https://graph.nfdi4objects.net/
[source]: https://github.com/Research-Squirrel-Engineers/bb-5kbc-sites

| | |
|---|---|
| Publication | [10.5281/zenodo.19830968](https://doi.org/10.5281/zenodo.19830968) |
| Licence | [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) |
| Graph | 28,531 triples · 33 classes · 81 properties |
| Wikidata | [Q141363969](http://www.wikidata.org/entity/Q141363969) |
| Query it | [12 example queries](https://research-squirrel-engineers.github.io/bb-5kbc-public/query/) |
| Map | [all 540 sites](https://research-squirrel-engineers.github.io/bb-5kbc-public/map.html) |

---

## How this repository works

One file is maintained by hand — `metadata.yaml`. Everything else is generated
on every push by [n4o-kg-profile][profile], pinned to `@v1`.

[profile]: https://github.com/Research-Squirrel-Engineers/n4o-kg-profile

```
metadata.yaml                  the only hand-maintained file
rdf/bb5kbc-bundle.ttl          fetched once from bb-5kbc-sites, archived here
  │
  ├── dist/n4o-collection.ttl  the registration record NFDI4Objects reads
  ├── dist/metadata.ttl        DCAT + VoID statistics + CRM alignment + queries
  ├── dist/metadata.jsonld     the same, as JSON-LD
  ├── dist/crm-alignment.ttl   rdfs:subClassOf to CIDOC CRM, loadable alone
  ├── queries.yaml             the example queries
  └── docs/                    GitHub Pages: landing page, query page, .rq files
```

`profile/` and `build/` are copied in by the action on every run and are
overwritten; edit them in `n4o-kg-profile`, never here.

## Rebuilding locally

```bash
pip install pyyaml rdflib pyshacl jinja2
python build/make_metadata.py     # metadata, RDF, queries.yaml
python build/build_sparql.py      # docs/index.html, docs/sparql.html
python -m http.server -d docs     # preview at http://localhost:8000/
```

The bundle is fetched once and then reused. Set `REFRESH_SOURCES = True` in
`build/make_metadata.py` (or `N4O_REFRESH=1`) to pick up a newer version from
`bb-5kbc-sites`.

## Updating after a new release upstream

1. `N4O_REFRESH=1 python build/make_metadata.py` — refetches the bundle.
2. Check the diff of `dist/metadata.ttl`. The outputs are byte-stable, so every
   line that moved says something changed: a new class, a different count, a new
   checksum.
3. Bump `version:` and `modified:` in `metadata.yaml`, and `homepage:` if the
   Zenodo DOI is version-specific.
4. Push. The action rebuilds, revalidates and redeploys the pages.

## Before this goes into the Knowledge Graph

- [ ] **`issued:`** should be the date of the Zenodo release being described.
- [ ] **fuzzy-sl types.** Five classes reach no CIDOC CRM class: `CertaintyType`,
      `LocationType`, `MethodType`, `PointType`, `SourceType`. They belong to
      `fuzzy-sl.squirrel.link`, so the decision is that ontology's, not this
      repository's — either anchor them upstream, or add the namespace to
      `model.external` in `metadata.yaml` to record that reusing them unanchored
      is deliberate.

The collection URI in `id:` is assigned by the VZG by hand; it is not something
this repository can produce.

The CIDOC CRM alignment is **not** on this list. It lives in the bundle, which
anchors 16 of its 21 domain classes; the build measures that rather than
restating it. If an anchor is wrong, fix it in
[bb-5kbc-sites](https://github.com/Research-Squirrel-Engineers/bb-5kbc-sites)
and refresh the bundle — never here.

## The query layer

Twelve queries, each on its own page under `docs/query/`, catalogued at
`docs/query/index.html` and all together on `all.html`. Every one of them is
editable and runs in the browser; every one is also a plain `.rq` file, and
every one is in `dist/metadata.ttl` as `sh:SPARQLSelectExecutable`, where a
harvester can read them.

A result that carries a coordinate is drawn on a map above its table, whatever
the query declared — so narrowing a query narrows its map with it. Three other
views are available per query (`intervals`, `barchart`, `scatter`), declared in
`metadata.yaml` as `view:` with `view_columns:`. The view draws above the
table, never instead of it: an edit that drops the column a view needs makes it
say so rather than go blank.

One trap worth knowing before editing: `bb5kbc:hatDatierung` hangs on the
`KulturelleZuordnung`, not on the `Fundstelle`, because a site with two
cultural attributions has two datings. `?site bb5kbc:hatDatierung ?d` returns
nothing at all.

## What the build checks

- **SHACL.** The four facts the N4O KG needs — title, publication URL, Wikidata
  item, licence — plus the NCMDP mandatory elements.
- **Every example query runs against the bundle.** Zero rows fails the build:
  SPARQL does not fail on a mistyped IRI, it returns nothing, so an empty result
  is the ordinary symptom of a broken graph rather than of a boring question.
- **Byte reproducibility.** A second run must produce identical files, so a diff
  after an unchanged rebuild means something is genuinely wrong.
- **SHA-256 per distribution**, recorded in `dist/metadata.ttl` — the only
  statement that later proves which version was loaded.

## Citation

See `CITATION.cff`, or cite the Zenodo record directly:
[10.5281/zenodo.19830968](https://doi.org/10.5281/zenodo.19830968).

## Licence

Data: CC BY 4.0. Build scripts and profile: MIT, from
[n4o-kg-profile][profile]. See `LICENSE`.
