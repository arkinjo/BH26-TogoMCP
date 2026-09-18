---
title: 'DBCLS BioHackathon 2026 report: Extending TogoMCP beyond RDF Portal while making its schema guides check their own answers'
title_short: 'BH26JP: TogoMCP'
tags:
  - TogoMCP
  - Model Context Protocol
  - SPARQL
  - knowledge graphs
  - large language models
  - schema documentation
authors:
  # Collaborators: please add, correct or remove your own entry (name, ORCID, affiliation, role).
  - name: Akira R. Kinjo
    orcid: 0000-0002-4006-8208
    affiliation: 1
    role: Conceptualization, Software, Data curation, Writing – original draft
  - name: Kozo Nishida
    orcid: 0000-0001-8501-7319
    affiliation: 2
    role: Data curation, Software, Validation
  - name: Shuichi Kawashima
    orcid: 0000-0001-7883-3756
    affiliation: 3
    role: Data curation, Resources
  - name: Yuki Moriya
    orcid: 0000-0001-8195-5893
    affiliation: 3
    role: Software, Validation
  - name: Takatomo Fujisawa
    orcid: 0000-0001-8978-3344
    affiliation: 4
  - name: Yasuhiro Tanizawa
    orcid: 0000-0002-6294-3309
    affiliation: 4
    role: Data curation, Resources
  - name: Priscilla Joanne
    affiliation: 5
    role: Validation, Investigation
  - name: Yoko Okabeppu
    affiliation: 6
    role: Data curation, Resources
  - name: Shuya Ikeda
    orcid: 0000-0002-1357-5159
    affiliation: 3
    role: Software, Resources
  - name: Toyofumi Fujiwara
    orcid: 0000-0002-0170-9172
    affiliation: 3
    role: Software, Resources
  - name: Yasunori Yamamoto
    orcid: 0000-0002-6943-6887
    affiliation: 3
    role: Conceptualization, Resources, Writing – review & editing
affiliations:
  - name: Anima Machina G.K., Osaka, Japan
    index: 1
  - name: RIKEN Center for Biosystems Dynamics Research, Kobe, Japan
    index: 2
  - name: Database Center for Life Science (DBCLS), Research Organization of Information and Systems, Japan
    index: 3
  - name: Bioinformation and DDBJ Center, National Institute of Genetics, Research Organization of Information and Systems, Mishima, Japan
    index: 4
  - name: Department of Computational Biology and Medical Sciences, Graduate School of Frontier Sciences, The University of Tokyo, Japan
    index: 5
  - name: OKBP, Inc., Yokohama, Japan
    index: 6
date: 19 September 2026
cito-bibliography: paper.bib
event: BH26JP
biohackathon_name: "DBCLS BioHackathon 2026"
biohackathon_url: "https://2026.biohackathon.org/"
biohackathon_location: "Matsuyama, Japan, 2026"
group: TogoMCP
# URL to project git repo --- should contain the actual paper.md:
git_url: https://github.com/arkinjo/BH26-TogoMCP
# This is the short authors description that is used at the
# bottom of the generated paper (typically the first two authors):
authors_short: Akira R. Kinjo \emph{et al.}
---

# Abstract

TogoMCP is a Model Context Protocol (MCP) server that lets large language model (LLM) agents query
life-science knowledge graphs in SPARQL, guided by per-database schema documents called MIE
(Metadata Interoperability Exchange) files. During the DBCLS BioHackathon 2026 (BH26), the TogoMCP
group set out to extend the server, refine the MIE files, add databases, learn what makes a good
SPARQL example, and turn use cases into reusable agent skills. In the first six days of the event we
published ten releases (v2.12.2 to v2.20.0) and grew the catalogue from 37 to 45 databases:
Fanta.bio, WikiPathways, IDSM, PubCaseFinder, LIPID MAPS, SwissLipids and MarpolBase, plus BH26
Microbes, an experimental dataset of KEGG Orthology assignments for 57.6 million prokaryotic
proteins built for the BioHackathon. Five of them are served from endpoints outside RDF Portal and
one from a QLever engine rather than Virtuoso, and they broke assumptions the rest of the corpus had
taught, such as named-graph pinning, federation with `SERVICE`, literal typing, and even that a
result set is complete. For the first time, MIE files were written by people other than the server's
maintainer, one of them by the maintainer of the database it describes, and a second group member
built TogoCX, a companion MCP server that returns database edges with instructions on how their
claims may be stated. We made every worked example in the MIE corpus (now 420) assert its recorded
result against the live endpoint in continuous integration. The first full comparison found drift
that execution-only checks had missed, most seriously a NANDO release that made an example join
silently miss about 88% of mapped diseases. Onboarding the new databases uncovered a series of
quantified "silent wrong answers", queries that return plausible results rather than errors,
including a category count inflated 3.1-fold by an upstream data defect and a gene enumeration that
returns 10,000 of 18,080 rows without an error or a warning. We also added two PubCaseFinder tools
for phenotype-driven rare-disease diagnosis support, began serving analysis workflows (agent skills)
from the server, fixed failure modes found in production call logs, made TogoID errors suggest
working conversion routes, and prepared a LOTUS natural-products graph for hosting on RDF Portal. We
argue that for LLM-facing schema documentation, "the example still runs" is not evidence that it is
still correct.

**Keywords:** TogoMCP; Model Context Protocol; SPARQL; RDF; knowledge graphs; LLM agents;
schema documentation

# Introduction

Life-science data are increasingly published as Resource Description Framework (RDF) graphs behind
SPARQL endpoints, but writing correct SPARQL against an unfamiliar schema remains a barrier for most
researchers. TogoMCP addresses this by exposing SPARQL endpoints, REST search services and ID
conversion to LLM agents through the Model Context Protocol
([MCP](https://modelcontextprotocol.io/)), and by giving the agent a compact, database-specific
schema guide, the MIE file, before it writes a query [@citesAsAuthority:Kinjo2026database]. At the
DBCLS BioHackathon 2025 we redesigned the MIE format around executable examples after an ablation
study showed that the query-construction material carries almost all of the benefit
[@extends:Kinjo2026bhxiv]. The public server runs at
[togomcp.rdfportal.org](https://togomcp.rdfportal.org/).

At the start of BH26 (v2.12.1, 3 September 2026), TogoMCP covered 37 databases. Thirty-five of them
were served from RDF Portal endpoints operated by DBCLS; the other two were TogoVar and GlyCosmos.
The TogoMCP hacking group registered five objectives for the week:

1. extending and enhancing TogoMCP;
2. examining and refining MIE files;
3. finding interesting use cases and workflows, and turning them into skills;
4. learning how to make good SPARQL examples;
5. adding new databases to TogoMCP.

This report describes what we did toward each objective between 13 and 19 September 2026, and what
we learned about keeping LLM-facing schema documentation correct as the set of databases, endpoints
and authors grows.

# New databases

Table 1 summarises the databases added during the week. Each was onboarded the same way: a row in
the endpoint registry, an MIE file with live-verified examples and documented traps ("gotchas"), and a
review pull request. Falsifiable claims carry a `check:` block that can be re-run against the endpoint.

Table: Databases added during BH26. CRE, cis-regulatory element; HPO, Human Phenotype Ontology;
KO, KEGG Orthology. Counts are as measured when each MIE was verified.

| Database (release) | Content | Endpoint | Notable for agents |
|---|---|---|---|
| Fanta.bio (2.13.0) | 821,722 human and mouse CREs with linked genes and ChIP-Atlas peak overlaps | RDF Portal `primary` | chromosome IRIs spelled differently from HCO |
| WikiPathways (2.14.0) | 2,087 pathways across 39 organisms | WikiPathways' own Virtuoso | revision-pinned pathway IRIs; `SERVICE` works |
| IDSM (2.14.0) | structure search over nine small-molecule datasets | ELIXIR CZ, PostgreSQL-based | union default graph (×30 rows unpinned) |
| PubCaseFinder (2.15.0) | 18,375 OMIM/Orphanet diseases, 379,263 HPO annotations | RDF Portal `primary` | only Japanese HPO and disease labels |
| LIPID MAPS (2.15.0) | 52,175 classified lipids, shorthand such as `PC 34:1` | lipidmaps.org, behind a WAF | no named graphs; inverted `subClassOf` |
| SwissLipids (2.16.0) | 777,965 lipids with sn-position composition | SIB | upstream ID-mapping examples return 0 rows |
| BH26 Microbes (2.19.0; experimental) | KofamScan KO assignments for 57.6M RefSeq proteins in 23,434 prokaryotic genomes | RDF Portal `microbes` (QLever) | 24% of hits below threshold; no organism names |
| MarpolBase (2.20.0) | 18,080 *Marchantia polymorpha* genes (MpTak_v7.1) with function, co-expression, orthogroups and curated literature | marchantia.info, its own Virtuoso | every result truncated at 10,000 rows |

## What each database adds

**Fanta.bio** holds CAGE-defined promoters and enhancers from FANTOM, annotated with transcription
factor binding peaks from ChIP-Atlas [@citesAsDataSource:Zou2024chipatlas], ENCODE SCREEN cCREs and
refTSS. Before it, TogoMCP could say where a gene is and which variants lie in it, but not which
regulatory elements act on it. **WikiPathways** [@citesAsDataSource:Agrawal2024wikipathways] adds
community-curated pathways with signed, directed interactions. **IDSM**
[@citesAsDataSource:Galgonek2021idsm] is the first database in the catalogue that searches by chemical
structure: its Sachem extension [@usesMethodIn:Kratochvil2018sachem; @usesMethodIn:Kratochvil2019idsm]
turns one SMILES string into the molecule's identifiers in PubChem, ChEMBL, ChEBI, Wikidata,
DrugBank and four other datasets. Additionally, in 2024, IDSM was extended to integrate mass spectrometry
databases, allowing for SPARQL queries based on mass spectral similarity [@citesAsDataSource:Galgonek2024idsm].
**PubCaseFinder** [@citesAsDataSource:Fujiwara2018pubcasefinder]
contributes the knowledge base behind DBCLS's rare-disease diagnosis support, including
text-mined disease–phenotype annotations based on the HPO [@citesAsDataSource:Gargano2024hpo].
**LIPID MAPS** [@citesAsDataSource:Conroy2024lipidmaps] is the only resource here that resolves the
lipidomics shorthand used in result tables (34,567 structures map onto 6,988 shorthand strings), and
**SwissLipids** [@citesAsDataSource:Aimo2015swisslipids] is the only one that records which fatty acid
occupies which sn-position, so "lipids with palmitate at sn-1" becomes a structured query.
**MarpolBase** [@citesAsDataSource:Tanizawa2026marpolbase] is the first plant genome in the catalogue
and the first bryophyte: the MpTak_v7.1 reference for the liverwort *Marchantia polymorpha*, with
structural and functional annotation for 18,080 genes, 18,232 orthogroups, a 740,830-edge co-expression
network over 164 RNA-seq conditions, and 2,609 hand-curated gene–literature assertions across 510 papers.
Those assertions hold what no other source in the catalogue does: each records, in a curator's own words,
what a given paper says about a given gene, typed by the gene's role in that study (subject, background,
comparator or tool) and by the kind of evidence behind it.

## An experimental BioHackathon dataset

**BH26 Microbes** (`bh26microbes`) was built for this BioHackathon by Shuichi Kawashima and Yoko
Okabeppu, and has not yet been publicly announced. It holds KofamScan
[@usesMethodIn:Aramaki2020kofamkoala] KEGG Orthology (KO) assignments, each with its HMM score,
E-value, profile threshold and significance, for 57,612,257 RefSeq
[@citesAsDataSource:OLeary2016refseq] proteins across 23,434 prokaryotic genome assemblies. It is the
only source in the catalogue that answers "which genomes encode function X" from gene content: 2,246
genomes carry a significant hit to the nitrogenase iron protein NifH (K02588), and 1,839 carry both NifH
and the MoFe protein alpha chain NifD (K02586). Because the dataset may still change shape or be
withdrawn, the MIE and the Usage Guide catalogue mark it as experimental, and it is deliberately left
off the public introduction page. Genomes and proteins carry no names or taxonomy on this endpoint.
The MIE therefore resolves organisms by rewriting each RefSeq assembly accession (GCF) to its paired
GenBank accession (GCA) and matching UniProt proteomes over `SERVICE`, and resolves proteins to
UniProtKB by rewriting the RefSeq protein IRI into UniProt's form. All nine examples pass their result
assertions, and all eight gotcha checks pass against the live endpoint.

## Endpoints outside RDF Portal

Five of the eight databases are served from their maintainers' own endpoints, and BH26 Microbes is
served by RDF Portal from a QLever [@citesAsRelated:Bast2017qlever] instance rather than Virtuoso.
These endpoints exposed habits that the Virtuoso-hosted corpus had taught agents and authors:

* **Graph pinning.** TogoMCP's guidance is to pin every query to a named graph, because co-hosted
  graphs re-declare shared ontology classes. LIPID MAPS has no named graphs, so a pinned query returns
  nothing. IDSM has a union default graph, so an unpinned query returns about 30 times as many rows.
  MarpolBase shows that the rule was not specific enough. Its endpoint hosts one database, yet that
  database's own function and annotation graphs re-declare identifiers and labels on the same gene IRIs,
  so the canonical gene lookup returns 36,087 rows unpinned against 18,080 pinned (×2.00). Pinning "the
  database's graphs" does not help when both offenders belong to the database: exactly one graph has to
  be named, and which one depends on the question.
* **Federation.** WikiPathways and IDSM can call RDF Portal and SIB with `SERVICE`; every outbound
  `SERVICE` from LIPID MAPS returned HTTP 502. Cross-database LIPID MAPS queries therefore have to be
  run from RDF Portal's `ebi` endpoint, with LIPID MAPS inside the `SERVICE` clause. MarpolBase refuses
  `SERVICE` by configuration rather than by failure, in 0.1 s, with a Virtuoso privilege error, so no
  retry and no simpler remote pattern will succeed, and every cross-database hop has to be a second query.
* **Engine semantics.** A Virtuoso behaviour documented across the corpus (plain and `xsd:string`
  literals do not match each other) does not apply to IDSM's PostgreSQL-based engine. Re-testing the
  two-argument `REGEX()` bug showed it on WikiPathways but not on IDSM, so it holds on 11 of 12 endpoints
  rather than "every endpoint". QLever, behind BH26 Microbes, rejects a query that uses any undeclared
  prefix (even `rdf:`), returns 0 rows without error for Virtuoso's `bif:contains` text search, and
  matches string literals only in plain form although `DATATYPE()` reports `xsd:string`. It also caps
  memory per query, so a `VALUES` join over several KOs can abort where a `UNION` of constant-bound
  branches runs about ten times faster.
* **Deployment, not only engine.** MarpolBase runs Virtuoso, the same engine as RDF Portal, and still
  needed three new rules. Its results are truncated at 10,000 rows and a larger `LIMIT` does not lift the
  ceiling: enumerating the 18,080 genes returns exactly 10,000 rows with `LIMIT 20000` and with no
  `LIMIT` at all, while `COUNT` over the same pattern correctly reports 18,080, and ordered `OFFSET`
  paging fails in two further ways. `ORDER BY` sorts `xsd:double` and `xsd:integer` literals as strings,
  so `ORDER BY DESC(?fold)` tops out at 999.8 when the true maximum is 48,226.3. And adding `ORDER BY` to
  an aggregate that has no `GROUP BY` makes Virtuoso group by the sort variable, turning one row of
  18,080 into 18,080 rows of 1. The last of these is not a MarpolBase property at all: it reproduces on
  RDF Portal's `primary` endpoint, so a trap found while onboarding a new database turned out to apply to
  databases the corpus had already documented.
* **Web application firewall (WAF).** The LIPID MAPS endpoint sits behind Cloudflare, which rejects
  legal SPARQL with HTTP 403 whenever the raw request body matches `\b(substr|concat|char)\(`, even
  inside a string literal. `GROUP_CONCAT(` passes because the underscore defeats the word boundary, and
  `SUBSTR (` with a space is valid SPARQL that clears the rule. Separately, a client that sends Python's
  default User-Agent is rejected outright.

These differences also exposed a bug in our own tooling. `run_sparql` distinguishes "the entity is
absent" from "the query is broken" by probing the anchor entity inside the query's graph; on an
endpoint without named graphs that probe is always false, so broken queries were reported as true
negatives. The probe now follows the query's own scoping.

# Schema guides that check their own answers

Each MIE example records the result it returned when it was verified. Until this week, nothing
compared that record with what the endpoint returns today. A weekly job flagged examples that errored
or returned zero rows, but a query that kept running while its answer changed looked healthy.
The `verified:` block was also free-form: across 334 examples it used 198 different key sets, mostly
prose.

## Assertable results

We reserved four keys that `scripts/check_mie_examples.py` asserts against the live endpoint
(Listing 1):

* `n`, a single-cell result such as a `COUNT`, within a tolerance (default 2%);
* `row_count`, the number of rows, which must be below any `LIMIT`;
* `min_rows`, the honest assertion for a `LIMIT`-capped query;
* `has_values`, values that must each appear in the result (IRI local names match).

Every example must carry at least one of them. A malformed assertion is reported as MALFORMED, and a
well-formed one that the live result no longer satisfies is reported as DRIFT; both fail pull-request
and weekly jobs. We migrated the corpus by re-running every example and comparing the result with the
old free-form record, rather than stamping today's figures over it. That comparison is what found
the problems below.

```yaml
# Listing 1. One example from the LIPID MAPS MIE (abridged).
- id: lipid_core_record
  question: What are the formula, monoisotopic mass, InChIKey and names
    of cholesterol (LMST01010001)?
  sparql: |
    SELECT ?label ?formula ?mass ?inchikey WHERE {
      <https://www.lipidmaps.org/rdf/LMST01010001>
        rdfs:label ?label ; chebi:formula ?formula ;
        chebi:monoisotopicmass ?mass ; chebi:inchikey ?inchikey . }
  verified:
    row_count: 2
    has_values: [Cholesterol, C27H46O, HVYWMOMLDIMFJA-DPAQBDIFSA-N]
    date: '2026-09-15'
    note: 2 rows, not 1; the two rdfs:label values duplicate every other column.
```

## What the comparison found

* **NANDO had been replaced upstream.** The 2026-05-22 NANDO release moved most MONDO mappings from
  `skos:closeMatch` to `skos:exactMatch`, so the MIE's MONDO join silently missed about 88% of mapped
  diseases. The same release added a pediatric chronic-disease programme, so notification numbers now
  repeat across two programmes. The file was revised against the live graph.
* **Figures had moved.** The Gene Ontology graph-pin warning quoted a 3.27-fold row inflation that is
  now 2.84-fold, because the set of co-hosted graphs re-declaring GO classes changed. Other figures were
  re-measured with dated notes, including taxonomy species (+4.9%) and HGNC EC-code cross-references (+3.8%).
* **Examples returned more than they claimed.** A ChEBI hierarchy example returned 18 rows for 5 amino
  acids (one row per path through `rdfs:subClassOf+`), and an anatomy example named parts that its query
  does not return.
* **Documentation one day old was already wrong.** MarpolBase changed shape twice during the week: an
  ontology namespace was retired on 17 September and a literature graph on 18 September, each leaving
  queries written against the previous day's description returning 0 rows with no error. Both are
  recorded in the MIE as traps rather than quietly corrected, because an agent may still be working from
  the older description.

The same release fixed a user-reported UniProt error of the same kind
([dbcls/togomcp#219](https://github.com/dbcls/togomcp/issues/219)). An example fetched a protein's
canonical sequence by naming the `-1` isoform, but across all 575,503 reviewed entries, 782 have a
different canonical isoform, and in 613 of them a non-canonical `-1` exists. The lookup therefore
returned the wrong sequence rather than no rows.

The checker itself had a bug that only an external endpoint could reveal. It ran every example
against its own database's endpoint and ignored an example's `endpoint_name`. For a database hosted on
RDF Portal the two are almost always the same (61 of 62 examples carrying the key), so the bug stayed
invisible until the LIPID MAPS cross-database example, which must run from `ebi`, failed on every run.
Resolution now follows `run_sparql` (explicit URL, then `endpoint_name`, then the database default),
and two guards assert that the override is still exercised.

## Keeping examples away from the benchmark

The MIE specification forbids using a benchmark question's subject as an example's vehicle, since that
would leak answers into the context we evaluate. The rule had been enforced by eye.
`scripts/check_mie_leakage.py` now matches every question's keywords, answer heads and answer IDs
against the examples of the databases that question uses, and runs in CI on any change to an MIE or a
question. Its first run over 100 questions and 37 MIEs found 19 matches, all generic vocabulary, which
were recorded as reasoned waivers that fail once they go stale.

# Silent wrong answers

The most expensive failures we met did not raise errors. They returned plausible numbers. Table 2
lists those found while onboarding and auditing databases this week. Each is now a gotcha in the
relevant MIE with its measured magnitude and a re-runnable check.

Table: Queries that return plausible but wrong results, found during BH26.

| Database | Trap | Measured effect | Remedy documented |
|---|---|---|---|
| LIPID MAPS | spurious upstream edge `category/6 -> category/101` pulls Fatty Acids [FA01] into Prenol Lipids | 8,558 instead of 2,739 (×3.1) | drop the edge in category queries |
| LIPID MAPS | lipid class nouns (e.g. "cardiolipin") are absent from lipid labels | label search finds 2 of 1,306 (recall 0.2–4%) | classify via headgroup shorthand or category tree |
| LIPID MAPS | three count keys had been merged into one figure | 607 reported; 708 nodes, 672 labelled, 607 with children | three keys with set arithmetic |
| ChEBI and LIPID MAPS | `chebi:` vs `chemrof:` property names | joins return 0 rows | join on InChIKey (1,498 of 1,500 agree) |
| WikiPathways | one compound filed under several ChEBI IDs | 59 instead of 109 pathways for glucose (×1.85) | collect the full ID set first |
| WikiPathways | `dcterms:isPartOf` also targets interactions and complexes | 304 instead of 107 pathways for MTOR (×2.84) | require `a wp:Pathway` on the parent |
| SwissLipids | fixed-depth hierarchy walk | a 4-hop walk misses 87% of Sphingolipids | use 1–5 hops for a category |
| PubCaseFinder | HP labels re-declared by three ontology graphs | ×3.15 rows unpinned | pin the graph |
| PubCaseFinder | article-to-MeSH predicate minted as `fabiohasSubjectTerm` | correctly spelled predicate finds nothing | use the IRI as minted |
| Fanta.bio | chromosome IRI form differs from HCO | IRI join returns 0 rows | join on strings |
| NANDO | release moved MONDO mappings to `skos:exactMatch` | ~88% of mapped diseases missed | query both predicates |
| BH26 Microbes | `rdfs:seeAlso` protein-to-KO links include the 24.1% of hits KofamScan marks not significant | 2,292 instead of 2,246 genomes for NifH; ×1.07 KO pairs for *E. coli* K-12 | require `significant = true` on the reified hit |
| BH26 Microbes | one identical hit statement per genome that shares a protein | ×1.033 hits overall | count distinct proteins or genomes |
| BH26 Microbes | Virtuoso `bif:contains` sent to QLever | 0 rows, no error | `FILTER(CONTAINS(LCASE(...)))` on KO definitions |
| TogoID and BH26 Microbes | RefSeq-to-UniProt conversion | RecA (NP_417179) maps only to A0A485JBB4, not Swiss-Prot P0A7G6 | rewrite the RefSeq IRI and join UniProt over `SERVICE` |
| MarpolBase | every result truncated at 10,000 rows, and a larger `LIMIT` does not lift it | 10,000 of 18,080 genes returned, while `COUNT` says 18,080 | partition on the gene-ID prefix |
| MarpolBase | `ORDER BY` on an aggregate with no `GROUP BY` is implicitly regrouped | one row of 18,080 becomes 18,080 rows of 1 | drop the `ORDER BY`, or write `GROUP BY` explicitly |
| MarpolBase | the database's own two graphs re-declare identifiers and labels on the same gene IRIs | 36,087 instead of 18,080 (×2.00) | pin exactly one graph, not "the database's graphs" |
| MarpolBase | FALDO puts `begin` after `end` on the minus strand | a `begin`/`end` window finds 2 genes instead of 3; 50.1% of genes at risk | range on `mpo:minPosition` and `mpo:maxPosition` |
| MarpolBase | `ORDER BY` sorts numeric literals lexically | `DESC(?fold)` tops at 999.8 against a true maximum of 48,226.3 | cast the sort key |
| MarpolBase | co-expression edges stored one-directionally | `mpo:gene1` alone loses 97% of one gene's 200 partners; 690 genes return 0 | `UNION` over both directions |

The LIPID MAPS findings began with an audit of the new MIE that K.N. carried out with Claude, which
was followed up in review ([dbcls/togomcp#231](https://github.com/dbcls/togomcp/pull/231)). The label
trap is structural rather than a data defect: every lipid carries a systematic name and a shorthand
abbreviation, and the class noun a biologist would type appears in neither. PGE2 has two labels, and
neither contains "prostaglandin". Two routes that should agree (headgroup shorthand and the category
tree) give the same 1,306 cardiolipins, so each checks the other.

The BH26 Microbes traps show that the pattern holds even for a dataset built during the week itself. The
most natural query, following the direct protein-to-KO link, over-annotates because the link is asserted
for every KofamScan hit, including those below the KO's adaptive score threshold; only the reified hit
records which hits KofamScan accepted.

MarpolBase contributes a kind of trap the corpus had not recorded before. Two of its six are not about
the data at all but about how much of it the endpoint will hand over, and neither is visible from inside
the result: a truncated enumeration and a regrouped aggregate both return well-formed rows of the right
shape, in well under a second, with no warning, and nothing in the response distinguishes 10,000 rows
that are the answer from 10,000 rows that are the first half of one. The dataset's producers had also
shipped query caveats of their own in a metadata graph, and two of them did not reproduce (a warning that
a `UNION` neighbour query "did not finish in 43 minutes", measured here at 0.04 s, and a claim about
casts inside `FILTER`), so the MIE records that they were tested and dropped, to stop a later author
restoring them from the upstream source.

# Rare-disease diagnosis support with PubCaseFinder

Most of the PubCaseFinder REST API [@citesAsAuthority:Fujiwara2022pubcasefinder] is a front end over the
RDF that TogoMCP already serves. We wrapped only the two functions that the RDF cannot reproduce.

* `pubcasefinder_rank_by_phenotypes` ranks OMIM diseases, Orphanet diseases or genes against a set of
  HPO terms, with PubCaseFinder's information-content-weighted matching through the HPO hierarchy. For
  three Marfan-like phenotypes, a SPARQL "has all three" query finds 6 unordered diseases, whereas the
  ranking places 9 diseases at score 1.0 and grades every partial match below them.
* `pubcasefinder_get_case_reports` lists published case reports for a MONDO disease, in English
  (PubMed) or Japanese (J-STAGE). The index is built by text mining rather than MeSH indexing: 507 of
  Marfan syndrome's 1,615 reports carry no Marfan MeSH heading, and the MeSH route over the PubMed RDF
  did not finish within 200 s.

Results carry disease names, MONDO IDs and genes, so the output of one tool feeds the other. Two
operational details shaped the implementation. The documented ranking endpoint returns HTTP 404, so the
tool uses the path the PubCaseFinder web application calls. DBCLS caps API use at 10 requests per
minute, 100 per hour and 1,000 per day for the whole server, so the tools cache results and refuse
with an explanatory error rather than exceed the quota.

# Fixes driven by real usage

## Production call logs

The server's call log from 27 July to 15 September 2026 showed two classes of calls that always failed.

* **Agents used the upstream API's parameter names.** 134 calls to the NCBI tools were rejected
  because agents sent E-utilities' own names (`retmax`, `retstart`, `id`) instead of ours. The tools
  now accept both.
* **Clients cached old tool lists.** ChatGPT connectors use a frozen snapshot of the tool list, so some
  clients still called tools renamed months earlier; 107 such calls failed, including calls from one
  user on five separate days. Renamed tools are now served under their old names with a notice asking the user to refresh the
  connector, and retired tools return an error that points to their replacement.

We also rewrote the ChatGPT setup instructions after finding that the advice to "re-run Scan Tools"
cannot work on several plans, where updating a tool list requires an administrator or re-creating the
app. The handbook (English and Japanese) and tutorial used by participants were regenerated.

## TogoID conversion errors that suggest routes

TogoID [@usesMethodIn:Ikeda2022togoid; @citesAsRelated:Ikeda2025togoid] converts identifiers between
databases along pre-computed tables. When an agent asked for a pair with no direct table, it
previously received a bare "no route" error. The error now lists up to five routes of at most two hops
from TogoID's `/route` endpoint, shortest first. `togoid_identifyId` gained an opt-in `verify=True`
that asks `/lookup/id` which tables actually contain an identifier, since pattern matching alone says
only that `672` is well formed for 17 datasets. Both endpoints were suggested by TogoID's maintainers
in response to our configuration request
([togoid/togoid-config#396](https://github.com/togoid/togoid-config/issues/396)). We did not adopt
`/search/id` for candidate generation: over 127 probes it returned exactly our local pattern matches,
unranked, at one request per identifier.

# Preparing LOTUS for RDF Portal

LOTUS [@citesAsDataSource:Rutz2022lotus] curates referenced structure–organism pairs for natural
products, but it lives in Wikidata and publishes no RDF of its own. It therefore cannot be added as a
database pointing at a live endpoint: the subset is a query pattern rather than a named graph,
Wikidata's endpoint stops every query at 60 s (six of LOTUS's twelve published examples exceed that),
and the live data carry no version string. We instead wrote a converter, in `scripts/lotus/`, from
LOTUS's frozen CSV release (v11, 2026-04-13) to a dated graph that RDF Portal could host. Entities keep
their Wikidata IRIs, so the graph joins with IDSM's Wikidata mirror without a mapping table.

The v11 conversion yields 9,138,012 triples covering 672,413 occurrences, 227,256 structures, 37,486
organisms and 91,426 references. Two of the week's lessons came from this work. First, the release's
core table is authoritative for which occurrences exist; reading only the richer metadata table
silently dropped 48 occurrences. Second, the vocabulary must ship in the same named graph as the data:
because TogoMCP queries pin their graph, a vocabulary in a separate graph is invisible, and schema
discovery would return nothing. The hand-written vocabulary (4 classes, 44 properties) documents traps in
its comments, for example that `lotus:ncbiTaxonId` reaches only 78.0% of organisms, and a test fails if
the converter emits a property the vocabulary does not define.

Shuichi Kawashima is curating the converted graph for RDF Portal, which plans to host it after
the BioHackathon.

# Community, use cases and skills

**New MIE authors.** Until this week every MIE file had been written by A.R.K. Two other people
wrote them during BH26. K.N. contributed four of the eight new databases (WikiPathways, IDSM, LIPID
MAPS and SwissLipids), working from the `mie-generator` skill, the MIE specification and the CI
checkers. Y.T. wrote the MarpolBase file, which needed only light revision afterwards, and which is
the first MIE written by the maintainer of the database it describes rather than by a consumer of
it. Review still caught what tooling could not, such as an example that named the wrong endpoint and
would have returned 0 rows if followed literally.

The [mie-generator skill](https://github.com/dbcls/togomcp/blob/main/.claude/skills/mie-generator/SKILL.md)
skill was executed for each of the following endpoints to generate the corresponding MIE files:
-	https://sparql.wikipathways.org/sparql
-	https://idsm.elixir-czech.cz/sparql/endpoint/idsm
-	https://lipidmaps.org/sparql
-	https://beta.sparql.swisslipids.org/sparql

**A companion server.** Y.M. built TogoCX during the week: an MCP server meant to run alongside
TogoMCP rather than replace it. Where TogoMCP hands an agent a schema guide and general SPARQL
access, TogoCX returns database edges with reading instructions attached: how the direction of each
edge may be stated (as `data`, `measured`, `assumed` or `undetermined`), what the relation may and
must not be used to claim, whether the record states the answer itself, and which queries ran, with
their row counts and the reasons the others did not. It targets a failure the MIE files do not
address. The problem is rarely that data is missing; it is that the data found does not say what the
agent then writes down, as when free text is recorded as a structured field or an answer is used to
explain itself. Over four shared cases, adding TogoCX raised gold records recovered from 82% to 95%
and cut the share of rows resting on an answer-bearing record from 79% to 64%, while a third
measure, competing explanations considered, fell from 74% to 50%, which its author reports as a
regression rather than leaving out. Agents called both servers within the same run, a mean of 7.8
TogoCX calls against 9.2 TogoMCP calls. TogoCX is a research prototype.

**Ideas from participants.** P.J. built and circulated a form asking for databases with SPARQL endpoints
to add, general improvements, and use cases where an MCP-based approach makes sense compared with
alternatives. It drew no responses. The databases added this week therefore came from the group's own
proposals and from conversations at the venue, which is worth recording: at an event where everyone is
already deep in their own project, a form competes badly with a hallway conversation.

**Skills.** At the mid-term report we showed the `research-article-analysis` skill, which validates a
paper's claims about compounds, reactions, pathways and protein functions against ChEBI, Rhea, UniProt,
Reactome and GO instead of trusting the paper's text. Such skills used to be installed on each client,
so a fix reached nobody who already had a copy, and hosts whose models only call tools could not use
them at all. Release 2.18.0 serves the public skills (`prism`, `research-article-analysis` and
`disease-analysis`) from the server through two routes that read one directory: a `get_workflow` tool,
which lists the workflows, returns a workflow's `SKILL.md` with its file list and content digest, or
returns one of its reference files; and `skill://` resources for hosts that read skills over MCP, in
anticipation of the skills extension proposed for MCP (SEP-2640), which the FastMCP versions we use do
not yet implement. Because the tool route has no description-based skill triggering, the Usage Guide
gained a Workflows section generated from the same registry; this also reaches clients whose cached
tool list does not yet show `get_workflow`. Serving skills centrally means that fast-changing facts
inside them, of the same kind as MIE gotchas, are corrected for every user at once. Developer-facing
skills (`mie-generator`, `qa-generator`) are reachable by neither route. The handbook and tutorial now
tell readers that the skills come with the connection, so local installation is optional.

# Discussion

Table 3 relates the group's objectives to the work reported above.

Table: Objectives of the TogoMCP group and corresponding outcomes.

| Objective | Outcome |
|---|---|
| Extend and enhance TogoMCP | 10 releases; PubCaseFinder tools; TogoID route suggestions; server-side workflows; log-driven fixes; TogoCX companion server |
| Examine and refine MIE files | two new MIE authors; result assertions over 420 examples; NANDO, GO, ChEBI, UniProt and LIPID MAPS corrections |
| Use cases and workflows into skills | `research-article-analysis` demonstration; three public workflows served by `get_workflow` (2.18.0) |
| How to make good SPARQL examples | see below |
| Add new databases | 37 to 45 databases (one an experimental BH26 dataset), 5 on external endpoints and 1 on QLever; LOTUS conversion prepared |

**Onboarding cost lies in traps, not YAML.** Writing an MIE file is quick; finding out where a database
returns a plausible wrong answer is not. Every trap in Table 2 would pass a test that only checks that
a query runs and returns rows. Quantifying each trap and attaching a re-runnable check is what lets the
documentation survive upstream releases, as the NANDO case shows.

**A database's own maintainer can write its MIE.** The MarpolBase file was written by the person who
maintains the database, and needed only light revision in review. That matters more than one file's
arithmetic. The expensive part of onboarding is knowing where a database returns a plausible wrong
answer, and the people who hold that knowledge are the ones who built the resource; if the format is
legible enough for them to write in, the cost of the corpus stops scaling with the time of the team
that maintains the server.

**Rules learned on one platform do not transfer.** Guidance that was true across RDF Portal (pin the
graph, federate with `SERVICE`, beware of literal typing) became endpoint-specific once databases
arrived from other operators and other query engines. Agent guidance should state the scope of each rule, and tools that
reason about queries, such as the empty-result probe, need the same care. The transfer failure runs in
both directions: MarpolBase runs the same engine as RDF Portal and still needed three new rules, one of
which (the implicitly regrouped aggregate) turned out to hold on RDF Portal as well and had simply never
been provoked. Onboarding a new endpoint is therefore also a test of the documentation already written.

**What makes a good SPARQL example.** From this week's practice, an example written for an LLM agent
should (i) record a machine-assertable result, (ii) name the endpoint it must run on, (iii) avoid the
subjects of evaluation questions, (iv) date and check every quantitative claim, and (v) never be copied
from an upstream collection without being re-run: SwissLipids' published ID-mapping queries use
`rdfs:seeAlso`, which carries only Wikidata links, and return 0 rows, while two of the caveats
MarpolBase's own producers ship with the data did not reproduce when we tested them.

**Agent-facing APIs should accept what agents already know.** Agents reached for E-utilities parameter
names and for tool names from cached lists. Accepting aliases and explaining retirements removed
failures that no amount of documentation had prevented.

**Wrap services only for what the graph cannot compute.** The PubCaseFinder tools were justified by
ranking and text-mined indexing that SPARQL over the same RDF cannot reproduce; everything else stays a
SPARQL query guided by the MIE.

**Limitations.** We did not re-run the TogoMCP benchmark on the enlarged catalogue during the week, so
we cannot yet quantify the effect of the new databases or corrections on answer quality. The checks
depend on live external endpoints, which are occasionally unavailable, and all figures are dated
measurements that will drift. BH26 Microbes is experimental, so its figures may change with the dataset
itself rather than with upstream releases.

# Future work

* Add benchmark questions that exercise the eight new databases and re-run the evaluation.
* Host the LOTUS graph on RDF Portal, and report the upstream defects found in LIPID MAPS and other
  sources to their maintainers.
* Measure use of `get_workflow` from the call logs.
* Promote BH26 Microbes out of experimental status once the dataset's shape is settled.
* Align MIE examples with community SPARQL example collections.
* Onboard more MIE authors, especially among database maintainers, and follow up databases proposed by
  neighbouring BH26 groups that are building RDF or MCP interfaces.

# Software and data availability

* TogoMCP source code: <https://github.com/dbcls/togomcp> (releases v2.12.2 to v2.20.0; MIT License).
* Public TogoMCP server: <https://togomcp.rdfportal.org/>.
* LOTUS converter and vocabulary: `scripts/lotus/` in the TogoMCP repository.
* TogoCX: <https://github.com/moriya-dbcls/togocx-mcp>.
* This report: <https://github.com/arkinjo/BH26-TogoMCP>.

## Acknowledgements

We thank the organisers of the DBCLS BioHackathon 2026 and the Database Center for Life Science for
hosting the event in Matsuyama. We thank the maintainers of RDF Portal, TogoID, PubCaseFinder and the
external SPARQL endpoints used here, the participants who registered interest in the TogoMCP group
(Daniel Puthawala, Mayumi Kamada, Susumu Goto, Núria QR, Naoya Yoshikuwa, Claude Nanjo
and Danil Ezhov). We thank Egon Willighagen for his advice on LOTUS. TogoMCP is
developed under contract with DBCLS.

## Funding

This work was supported by the MEXT National Life Science Database Project (NLDP) (grant number
JPNLDP202401) and the Life Science Database Integration Project, NBDC of Japan Science and Technology
Agency.

# References
