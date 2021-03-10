---
title: "Compiling R Markdown to Jekyll"
author: "Ethan Roubenoff"
date: "3/10/2021"
output:
  md_document:
    variant: gfm
    preserve_yaml: yes
knit: (function(inputFile, encoding) { rmarkdown::render(inputFile, encoding = encoding,
  output_file=paste0(Sys.Date(), "-", sub(".Rmd", ".md",inputFile)), output_dir =
  "~/eroubenoff.github.io/_posts") })
layout: post
tags:
- jekyll
- r-markdown
always_allow_html: yes
---

I have to confess that my [last
post](http://www.eroubenoff.net/2021-03-04-spatial_sim/) wasn’t written
exclusively for this blog. It began as an R Markdown document about a
year ago as I was fiddling around with some different autoregressive
models. I thought it would be pretty straightforward to go from R
Markdown to Jekyll– what I’m using to host this blog– and it was pretty
easy, but there’s a few tricks. I learned a ton about how R Markdown
works and wanted to put the notes here, mostly for my future reference,
and maybe for anyone else trying to go from R Markdown to Jekyll.

[This post by Steven Miller was really waht got me going in the right
direction.](http://svmiller.com/blog/2019/08/two-helpful-rmarkdown-jekyll-tips/)

Here are the requirements I had: \* R Markdown post needed to be in any
arbitrary location on my computer \* Rendered .md output file had to be
in `~/eroubenoff.github.io/_posts` \* Any images needed to be in
`~/eroubenoff.github.io/assets/img` \* Of course, the document had to be
renderable by Github/Jekyll \* LaTex needed to be renderable (a whole
other can of worms!)

I began by first trying to knit my R Markdown doc to GFM
(Github-Flavored Markdown). I had a number of figured in my document,
and these would all render to static images and linked with
`[alt text]("relative path")`. However, this relative path became an
issue as Jekyll would move around object locations.

I stole/adapted the following from Steven’s post to put in my YAML
header:

    ---
    title: "Compiling R Markdown to Jekyll"
    author: "Ethan Roubenoff"
    date: "3/10/2021"
    output:
      md_document:
        variant: gfm
        preserve_yaml: TRUE
    knit: (function(inputFile, encoding) {
      rmarkdown::render(inputFile, encoding = encoding, output_file=paste0(Sys.Date(), "-", sub(".Rmd", ".md",inputFile)), output_dir = "~/eroubenoff.github.io/_posts") })
    layout: post
    tags: [jekyll, r-markdown]
    always_allow_html: true
    ---

It’s critical that you include `output: md_document` and the associated
sub-specifications so that your output file is in the right format.
`always_allow_html` lets you put random html tags in your document and
have it work. But the real heavy lifting is done by `knit`:

    function(inputFile, encoding) {
      rmarkdown::render(
        inputFile, 
        encoding = encoding, 
        output_file=paste0(Sys.Date(), "-", sub(".Rmd", ".md",inputFile)), 
        output_dir = "~/eroubenoff.github.io/_posts"
        )}

When you click “knit” at the top of your R Studio editor, it calls this
command. `inputFile` is the file you send to be rendered, and give it
the default encovidng. The output file is the same file with the current
date appeneded (necessary for the Jekyll theme I’m using) and “.Rmd”
subbed for “.md” (possible that this latter part is done by default).
The output directory tells where to put the markdown file.

The next thing (in a code chunk in your document) is to handle images.
Normally knitr will render images (including ggplot, figures, etc),
insert them using pandoc, and then delete the temp images. But since we
are not using pandoc, we need to keep the images and link them to the
.md document. To do this (again, huge credit Steven’s post!):

    base_dir <- "~/eroubenoff.github.io" # i.e. where the jekyll blog is on the hard drive.
    base_url <- "/" # keep as is

    # If the document is currently being knit, do this; skip it in normal execution
    if (!is.null(knitr::current_input())){
      # Output path for figures
      fig_path <- paste0("assets/img/", str_remove(knitr::current_input(), ".Rmd"), "/")
      # Set base directories
      knitr::opts_knit$set(base.dir = base_dir, base.url = base_url)
      # Set figure directories
      knitr::opts_chunk$set(fig.path = fig_path,
                          cache.path = '../cache/',
                          message=FALSE, warning=FALSE,
                          cache = FALSE)
    }

The gist here is that we are setting `text knitr::opts_knit` and
`text knitr::opts_chunk` where to point. Since these are the final
directories in my project folder, we don’t need to worry about
dependencies or relative paths. Yay!

The last thing is Latex in GFM. I have no idea why it doesn’t work, and
truthfully I don’t really care to find a true workaround. But putting my
equation in
<http://www.sciweavers.org/free-online-latex-equation-editor> workes
really well, actually. You can link to the output of iTex2img (what I do
when I am lazy):

    <img src="http://www.sciweavers.org/tex2img.php?eq=%5Cbinom%7Bn%7D%7Bk%7D%20%3D%5Cbinom%7Bn-1%7D%7Bk%7D%20%2B%20%5Cbinom%7Bn-1%7D%7Bk-1%7D&bc=White&fc=Black&im=jpg&fs=12&ff=arev&edit=0" align="center" border="0" alt="\binom{n}{k} =\binom{n-1}{k} + \binom{n-1}{k-1}" width="203" height="46" />

<img src="http://www.sciweavers.org/tex2img.php?eq=%5Cbinom%7Bn%7D%7Bk%7D%20%3D%5Cbinom%7Bn-1%7D%7Bk%7D%20%2B%20%5Cbinom%7Bn-1%7D%7Bk-1%7D&bc=White&fc=Black&im=jpg&fs=12&ff=arev&edit=0" align="center" border="0" alt="\binom{n}{k} =\binom{n-1}{k} + \binom{n-1}{k-1}" width="203" height="46" />

It is probably safer to save a temporary copy…
