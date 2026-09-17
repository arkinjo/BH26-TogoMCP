# DBCLS BioHackathon 2026 report: TogoMCP

This repository holds the [BioHackrXiv](https://biohackrxiv.org/) preprint of the **TogoMCP** hacking
group at the [DBCLS BioHackathon 2026](https://2026.biohackathon.org/) (BH26JP, 13–19 September 2026,
Matsuyama, Japan).

**Working title:** Extending TogoMCP beyond RDF Portal while making its schema guides check their own
answers

* Latest PDF: [`paper/paper.pdf`](paper/paper.pdf) (rebuilt automatically on every push to `main`)
* Manuscript source: [`paper/paper.md`](paper/paper.md)
* References: [`paper/paper.bib`](paper/paper.bib)

**Status:** draft. Author list and several sections (marked `[TODO]`) are still open.
[TODO: submission deadline]

## About TogoMCP

[TogoMCP](https://github.com/dbcls/togomcp) is a Model Context Protocol (MCP) server that lets LLM
agents query life-science knowledge graphs through SPARQL, guided by per-database schema documents
(MIE files). The public server runs at <https://togomcp.rdfportal.org/>. The report covers TogoMCP
releases v2.12.2 to v2.17.0 and the work done during the BioHackathon week.

## For collaborators

### Adding, correcting or removing your name

Authors are listed in the YAML front matter at the top of `paper/paper.md`. Please edit your own
entry:

```yaml
authors:
  - name: Your Name
    orcid: 0000-0000-0000-0000        # optional but encouraged
    affiliation: 2                    # index into the affiliations list below
    role: Software, Validation        # CRediT terms, see https://credit.niso.org/
affiliations:
  - name: Your Institute, City, Country
    ror: 00xxxxx00                    # optional, see https://ror.org/
    index: 2
```

* If your affiliation is shown as `[TO BE CONFIRMED]`, please replace it.
* If you would rather not be an author, delete your entry (we will thank you in the
  Acknowledgements instead).
* If you stay on the list, please add a sentence or two in the text about what you did during the
  week, for example under "Community, use cases and skills". Authorship should reflect a contribution.

### Editing the text

* Open a pull request, or push to `main` if you have write access. Each pull request gets a PDF
  preview (see the "Actions" tab, artifact `paper`).
* Search for `[TODO` to find open items.
* BioHackrXiv conventions: at most two heading levels, no footnotes, abbreviations defined at first
  use, and about 10 pages including references. The draft uses British spelling.
* Add references to `paper/paper.bib` and cite them with a CiTO intent, for example
  `[@usesDataFrom:Key]` or `[@citesAsAuthority:Key]`. See the
  [BioHackrXiv guide](https://guide.biohackrxiv.org/) for the list of intents.

## Building the PDF

The GitHub Action in `.github/workflows/gen_pdf.yaml` builds `paper/paper.pdf` with the official
BioHackrXiv generator and commits it back on every push to `main`. You can also paste the repository
URL into the [BioHackrXiv preview service](http://preview.biohackrxiv.org/).

## License

The preprint will be submitted to BioHackrXiv under CC BY 4.0. [TODO: replace the template's CC0
`LICENSE` file before submission.]
