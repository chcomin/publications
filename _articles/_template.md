---
# Filename convention: YYYY-short-title.md
layout: article
title: "Full Title of the Article"
authors:
  - name: "First Author"
    orcid: "0000-0001-2345-6789"   
  - name: "Second Author"
    orcid: "0000-0009-8765-4321"
date: YYYY-MM-DD                   # ISO 8601 publication date
year: YYYY                         # duplicated for convenient use in templates
journal: "Journal Name"
volume: "1"
issue: "1"
pages: "1-10"                      # use a plain hyphen, e.g. "100-115"
doi: "10.XXXX/XXXXXXXX"
abstract: |
  Write the full abstract here.  The vertical bar (|) preserves line breaks
  in YAML; the text is treated as a single block.  Markdown and inline LaTeX
  (e.g. $x^2$) are supported when mathjax is true.
keywords:
  - keyword one
  - keyword two
  - keyword three
mathjax: true         
# license: "CC BY 4.0"  
bibtex: |
  @article{CITEKEY,
    title   = {Full Title of the Article},
    author  = {First Author and Second Author},
    journal = {Journal Name},
    year    = {YYYY},
    volume  = {1},
    number  = {1},
    pages   = {1--10},
    doi     = {10.XXXX/XXXXXXXX}
  }
---

## Introduction

Article body starts here.  Use standard Markdown headings, paragraphs,
lists, and code fences.

With `mathjax: true`, inline equations use `$...$` or `\(...\)` and
display equations use `$$...$$` or `\[...\]`.

## Methods

...

## Results

...

## Conclusion

...

## References

...
