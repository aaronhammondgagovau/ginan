 
# Documentation

The html documentation is set up to use client-side processes to similate a more extensive server.
What this means is that the source for pages is separated and spread across multiple files to enable consistent styling and ease of editing.

## URLS

Pages are all accessed through one url, with a parameter passed to the page to indicate which content is desired.

    index.html?p=home
    
This will load the `home` page into the browser. Other pages are accessable by changing the parameter value after the `?p=`

## File Structure

The content is spread across different types of files, which are individually simple and serve a single purpose each.

### The HTML layout - index.html

This file contains the html layout to be used for all pages in the site. 
Things such as the header and footer, dropdown links and all of the scripts and css includes are placed in this file.

### Page Indexes - pageIndexes/

These are simple text files that are used to concatenate different source files, and specify the different pages that can be accessed.
They are simply a list of paths to markdown files.
There should be one file per line - ensure there is no empty lines in the file.

The `home` page described above is created by creating a file called `home` in the `pageIndexes` directory. It's contents are simply:

     markdown/home.md

A more complicated page `peaUsage` merges several markdown files into a single page. It's contents are simply:

    markdown/usingThePea.md
    markdown/peaExamples.md
    markdown/peaConfig.md
    
### Page Content - markdown/

The content for pages is written in markdown, with the ability to add inline mathematical equations using mathjax.

The markdown content is typically stored in the `markdown/` directory.

It should be noted that placing content in this directory does not automatically generate a page, a page index must be created as above for the markdown-html conversion to be applied.

## Editing documentation

While editing markdown is relatively straightforward, it is useful to preview the actual rendered html output before committing changes and pushing back to github. 
The client-side script of this system does not work on modern browsers locally, unless a small http server is simply deployed.

    cd docs
    python3 -m http.server
    
These commands should start a server and provide a link that can be used to access the preview of the rendered page.

Other IDEs will also be able to give a simplified rendering of the output, but without any css or mathjax support.

## Publishing to gh-pages

The addition of a github workflow in the repository - `.github/workflows/publish.yaml` - means that any time a commit is pushed to `main`, the content of the repo's `docs` directory will be moved to the `gh-pages` branch, replacing any existing content.
This ensures that the documentation available online is kept synchronised with the correct version of the code.
The content available on the gh-pages website will always be up to date with the latest updates, but the synchronised history of all documentation for each version of the code will be available in the source files. (Like the file you're reading right now)

### Code documentation

The documentation that is written inline within the software's source files will also be copied to the `gh-pages` branch.
The publish workflow will run `doxygen` which will scrape the code for comments and create indexes and call-graphs for all relevant functions.
This documentation will be created within the `codeDocs/` folder automatically, and will be available on the `gh-pages` site.

