---
strand: R
excerpt: "Rnotebooks are scientific notebooks for R, somewhat like jupyter for anyone coming from python"
---

# Rnotebooks - What and Why?

For anyone unfamiliar with [Rnotebooks][1] here is a quick overview of why you might want to use them more experienced users can [skip ahead](#body). Rnotebooks are scientific notebooks for `R`, somewhat like [jupyter](https://jupyter.org/) for anyone coming from `python` but baked right into the Rstudio IDE which offers some benefits over the browser based interface of jupyter. It permits you to organise your code, notes, reasoning and references in one place. Combining Rnotebooks with a version management system such as git gives a robustness similar paper lab book records when it comes to seeing what you did and when coupled with dynamism, portability, share-ability and ease of backup of electronic working. Rnotebooks use a simple flavour of markdown with options to render output to HTML and PDF (via LaTeX) formats. Rnotebooks also have big pluses for reproducibility, creating an Rnotebook that does, explains and references your analysis makes it very easy to give to another at least somewhat competent `R` user and have them re-run your analysis - potentially with their own variants. Reproducibility and verifiability are substantial issues in scientific computing, including my own field of biology. [A recent article in PeerJ](https://doi.org/10.7717/peerj-cs.158) provides a nice discussion of these issues and a look at what the future of scientific computing notebooks might resemble.

## Basic Structure

Raw Rmarkdown looks like this:

	---
	title: "Example Rnotebook" # a yaml header with document properties and options
	---
	
	# Introduction

	Mardown formatted text, with __Bold__ and *exciting!* Scientific claims
	about $in-line math^2$ at least according to @Smith2007.

	```{r}
	print("some actual R code in chunks")
	```

	saved with a .Rmd file extension

Rstudio of course adds nice syntax highlighting, and various bells and whistles.

# The acctual YAML header stuff {#body}

## Inline R {#inliner}

You can use inline `R` in the YAML header of an Rnotebook to produce dynamic content. This takes the general form: 

```yaml
option: "`r <some R> `"
```

For example you can include the current date with: 

```yaml
date: "`r Sys.Date()`"
``` 

I'm partial to the YYYY-MM-DD format due to it's unambiguousness and nice sorting behaviour but you can of course employ `format()` to render the date in other ways.

## Params

The params option allows you to add arguments to your Rnotebook. The params you add to your header are accessible from within the notebook from the immutable `params` list. Rstudio makes the contents of this list available in interactive sessions so you can use them whilst working on your code not just when you build the notebook. Note that you can reference `params` in other options ([see](#bibyaml)).

```yaml
---
params:
  includeThing: TRUE # set the default to TRUE
---
```


	```{r optionalChunk, eval = (params$includeThing == TRUE), echo = (params$includeThing == TRUE)}
	print(
		paste0(
			"Some R that is only evaluated if and included in the notebook if",
			" params$includeThing is true"
		)
	)
	```


## Document Format Options

For an HTML output these are a few of my favourite options. There are numerous additional options described in the [outputs section](https://bookdown.org/yihui/rmarkdown/documents.html) of the manual, setting the depth of the table of contents for example.

```yaml
---
output:
  html_document:
    df_print: paged       # print paged tables - like the default 'html_notebook' format
    fig_caption: yes
    number_sections: yes  # prepend x.y style numbering to you sections
    toc: yes              # Add a table of contents
    toc_float: yes        # have to TOC float at the side of your HTML page so you do have to keep scrolling to the top
---
```

For a PDF output `pdf_document` can be used instead of `html_document` though my preferred table format for PDF is `df_print: kable`. More advanced LaTeX customisations can also be used in conjunction with PDF outputs.

## Bibliograghy and Citation YAML options {#bibyaml}

Placing a `bibliography` option in your Rnotebook's header and pointing it to a bibtex file containing your citation information permits you to create citations in Rnotebooks using the following syntax: `@Smith2016` for an in-line citation e.g. 'work by Smith *et al.* 2016 showed that cheese...' or `[@Smith2016]` for a reference like this: 'assertion (Smith *et al.* 2016)', or even lists of citations to be contracted where possible given the citation style e.g. `[@Smith2016; @Jones2018]`, (note the semi-colon list separator) yielding something like this: 'assertion [1-2]'

I frequently use a header that contains code like this:

```yaml
---
bibliography: "`r normalizePath(params$bib)`"
params:
  bib: "~/Documents/bibtex/library.bib"
---
```

The reason I do this is my bibliography has the same path relative to my home directory on my laptop, desktop and computing clusters but the absolute paths differ and __these headers seem to prefer absolute paths__. Thus, if I compose a notebook on one system it won't execute on another unless I change the path or use a set-up like this to do so dynamically when building the notebook.  

I also frequently set the path to my working directory as a parameter to my Rnotebooks and use relative paths to any files I want to load/write in the body of the Rnotebook so as to achieve similar portability between the different system's I work on as I get with my bibliography files.

The following chunk sets the working directory for when you 'knit' your Rnotebook into the desired format in the first line and for interactive sessions in the second.

	```{r}
	knitr::opts_knit$set(root.dir = normalizePath(params$pwd))
	setwd(params$pwd)
	```


__A note on generating your bibtex file(s).__ I currently use [Mendeley](https://www.mendeley.com/) as my refernce manager and it has a nice bibtext output option which is automatically updated whenever you sync (On balance I would probably recomend [Zotero](https://www.zotero.org/) to someone starting out afresh with reference management but its bibtex output is not quite as convenient as Mendeley's)

If you have multiple bibliography files this can be done:

```yaml
bibliography: [multiple.bib, dotbib.bib, files.bib]
```

Including a `csl` option allows you to specify a citation style using the `.csl` format. The specific citations styles of numerous journals in `.csl` format can be found [here](https://github.com/citation-style-language/styles). Including the `link-citations: yes` option will create hyperlinks from the in-text references to the full citations at the end of the document.

By default the bibliography is placed at the very end of your document, so simply placing a `# References` header at the end of your document helps to separate your bibliography from the body of your text and puts an entry for it in the table of contents. If however you have some appendices to add after your references placing this HTML snippet in your Rnotebook should set the position at which the references will be rendered: `<div id="refs"></div>`. Helpfully this will set the postion in both HTML and PDF outputs. (This may not work with older versions of `pandoc`).

# Executing an Rnotebook with params

Whilst you can render your Rnotebook with a one line `R` command from your terminal if you have a lot of params it can get unwieldy, you may also want to be able to reproduce your render at a later time or even submit it as a job to a batch computing manager. To do this you can create simple bash scripts like the one below to render your Rnotebook.

```bash
#!/bin/bash
R --no-save --no-restore <<EOF
rmarkdown::render(
	'notebook.Rmd',
	output_file = 'notebook.html',
	params = list(
		bib = "path/to/some/bib.bib"
	)
)
EOF

```

The `--no-save` option prevents `R` from saving your notebook's R session, and the `--no-restore` option prevents your Rnotebook from loading whatever random previous `R` session files you have lying around in your working directory into it's session.

# Full Length Example YAML header

```yaml
---
title: "A thing I'm Working on - Ideally with a more descriptive title"
author: "Richard J. Acton"
date: "`r Sys.Date()`"
output: # Specifying multiple outputs appears to favour the first
  pdf_document:
    toc: yes
    fig_caption: yes
    df_print: kable
  html_document:
    fig_caption: yes
    number_sections: yes
    toc: yes
    toc_float: yes
    df_print: paged
  html_notebook: # This determines the RStuido preview format
    fig_caption: yes
    number_sections: yes
    toc: yes
    toc_float: yes
bibliography: "`r normalizePath('~/Documents/bibtex/library.bib')`"
csl: "`r normalizePath('~/Documents/bibtex/genomebiology.csl')`"
link-citations: yes # make citations hyperlinks
linkcolor: blue
---
```

# Resources

- [R Markdown The Definitive Guide][1]
- [Citation Style Launguage Styles Repository](https://github.com/citation-style-language/styles)

[1]: https://bookdown.org/yihui/rmarkdown/

*Feedback is always welcome, especially if you spot any mistakes.*
