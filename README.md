# ccpp-doc
This repository contains the technical documentation for the Common Community Physics Package (CCPP),
maintained and developed at the Developmental Testbed Center (DTC). The latest documentation, as well
as documentation for previous versions, can be viewed at
[the DTC website](https://dtcenter.org/community-code/common-community-physics-package-ccpp/documentation).

## Notes to Developers
The documentation is generated with Sphinx, using the reStructuredText (*.rst*) files in the 
*CCPPtechnical/source* directory.  Output can be generated in HTML or PDF formats.

## Prerequisites

In order to create the technical documentation as described below, Sphinx and its 
extension sphinxcontrib-bibtex need to be installed on the system. If PDF output
is required, TeX/LaTeX must also be installed.

Sphinx can be installed in various ways (see
[https://www.sphinx-doc.org/en/master/usage/installation.html]([https://www.sphinx-doc.org/en/master/usage/installation.html])),
for example using Anaconda:
```
conda install sphinx
conda install -c conda-forge sphinxcontrib-bibtex
```

Comprehensive TeX/LaTeX distributions are provided for Windows, macOS and Linux.
For more information see [https://www.latex-project.org/get/](https://www.latex-project.org/get/).

## Creating the technical documentation

To generate the technical documentation:

1. Clone the repository.
```
git clone https://github.com/NCAR/ccpp-doc.git
```
2. Build the HTML document.
```
cd ccpp-doc/CCPPtechnical
make html
```
This will generate HTML files in *./build/html*.
3.  Build the PDF document.
```
cd ccpp-doc/CCPPtechnical
make latexpdf
```
This will generate a PDF file *./build/latex/CCPPtechnical.pdf*.

