# DBCLS BioHackathon 2026 report: TogoMCP

This repository holds the [BioHackrXiv](https://biohackrxiv.org/) preprint of the **TogoMCP** hacking
group at the [DBCLS BioHackathon 2026](https://2026.biohackathon.org/) (BH26JP, 13–19 September 2026,
Matsuyama, Japan).

**Title:** Extending TogoMCP beyond RDF Portal while making its schema guides check their own
answers

**Published preprint:** <https://doi.org/10.37044/osf.io/t25ng_v2>

* Latest PDF: [`paper/paper.pdf`](paper/paper.pdf) (rebuilt automatically on every push to `main`)
* Manuscript source: [`paper/paper.md`](paper/paper.md)
* References: [`paper/paper.bib`](paper/paper.bib)

**Status:** published (version 2 of the BioHackrXiv preprint).

## About TogoMCP

[TogoMCP](https://github.com/dbcls/togomcp) is a Model Context Protocol (MCP) server that lets LLM
agents query life-science knowledge graphs through SPARQL, guided by per-database schema documents
(MIE files). The public server runs at <https://togomcp.rdfportal.org/>. The report covers TogoMCP
releases v2.12.2 to v2.17.0 and the work done during the BioHackathon week.

## Citing this report

Kinjo, A. R. *et al.* Extending TogoMCP beyond RDF Portal while making its schema guides check
their own answers. BioHackrXiv (2026). <https://doi.org/10.37044/osf.io/t25ng_v2>

Author names, ORCIDs, affiliations and CRediT roles are recorded in the YAML front matter of
[`paper/paper.md`](paper/paper.md), and are fixed as of the published version.

## Corrections and new versions

The preprint is published, so `main` no longer tracks a draft. Changes to the text only reach
readers when a new version is submitted to BioHackrXiv, which mints a new `_vN` DOI.

* For a correction, open a pull request rather than pushing to `main`. Each pull request gets a PDF
  preview (see the "Actions" tab, artifact `paper`).
* Substantive changes — including any change to the author list — need the agreement of the
  authors before a new version is submitted.
* Once a new version is out, update the DOI at the top of this file and the status line.
* BioHackrXiv conventions still apply: at most two heading levels, no footnotes, abbreviations
  defined at first use, and about 10 pages including references. The text uses British spelling.
* Add references to [`paper/paper.bib`](paper/paper.bib) and cite them with a CiTO intent, for
  example `[@usesDataFrom:Key]` or `[@citesAsAuthority:Key]`. See the
  [BioHackrXiv guide](https://guide.biohackrxiv.org/) for the list of intents.

## Building the PDF

The GitHub Action in `.github/workflows/gen_pdf.yaml` builds `paper/paper.pdf` with the official
BioHackrXiv generator and commits it back on every push to `main`. You can also paste the repository
URL into the [BioHackrXiv preview service](http://preview.biohackrxiv.org/).

## License

The text of this report is licensed under the
[Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/)
(CC BY 4.0); see [`LICENSE`](LICENSE).
