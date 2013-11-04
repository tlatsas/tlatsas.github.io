---
layout: post
title: preview sphinx documentation
date: 2013-07-23 08:28:59
categories: [python, sphinx, documentation, release]
comments: true
---

I recently released a very small utility called [sphinx-serve][1] which
helps me preview sphinx documentation locally. It spawns a simple http
server on the documentation folder (html or singlehtml).

Installation is very simple, in the same virtual environment with your
sphinx documentation issue `pip install sphinx-serve`. This will make
the __sphinx-serve__ command available.

When you issue the above command the scipt tries to detect the build
folder of the documentation. It searches the current working directory
plus it navigates backwards in case you are in a deeper level (much
like the bundler utility). When it finds the build folder (the folder
name is configurable through the `--build` argument) it spawns a simple
http server. It serves files from the __http__ folder (default) or from
the __singlehtml__ folder if `--single` is supplied.

It is handy to add the sphinx-serve command with any required arguments
to the sphinx Makefile:

    preview:
        sphinx-serve --single --port 4000

now you can easily preview your documentation by issuing `make preview`.

[1]: https://github.com/tlatsas/sphinx-serve
