About the documentation
=======================

Pants documentation is generated from markdown sources and some
reflection code by Pants itself.

Generating the site
-------------------

A script encapsulates all that needs to be done. To generate and preview
changes, just:

    :::bash
    # This publishes the docs **locally** and opens (-o) them in your browser for review
    ./build-support/bin/publish_docs.sh -o

Publishing the site
-------------------

We publish the site via [Github Pages](https://pages.github.com/). You need `pantsbuild` commit
privilege to publish the site.

Use the same script as for generating the site, but request it also be published. Don't
worry—you'll get a chance to abort the publish just before it's committed remotely:

    :::bash
    # This publishes the docs locally and opens (-o) them in your browser for review
    # and then prompts you to confirm you want to publish these docs remotely before
    # proceeding to publish to http://pantsbuild.github.io
    ./build-support/bin/publish_docs.sh -op

If you'd like to publish remotely for others to preview your changes easily, the `-d` option creates
a copy of the site in a subdir of <http://pantsbuild.github.io/>:

    :::bash
    # This publishes the docs locally and opens (-o) them in your browser for review
    # and then prompts you to confirm you want to publish these docs remotely before
    # proceeding to publish to http://pantsbuild.github.io/sirois-test-site
    ./build-support/bin/publish_docs.sh -opd sirois-test-site

Cross References
----------------

If your doc has a
link like `<a pantsref="bdict_java_library">java_library</a>`, it links to
the BUILD Dictionary entry for `java_library`. To set up a short-hand
link like this...

Define the destination of the link with an `pantsmark` anchor, e.g.,
`<a pantsmark="bdict_java_library"> </a>`. The `pantsmark` attribute (here,
`bdict_java_library`) must be unique within the doc set.

Link to the destination with an `pantsref`, e.g.,
`<a pantsref="bdict_java_library">java_library</a>`.

Doc Site Config
---------------

The site generator
takes "raw" `.html` files, "wraps" them in a template with some
navigation UI, and writes out the resulting `.html` files.

You configure this with `src/python/pants/docs/docsite.json`:

`sources`:<br>
Map of pages to the `.html` files they're generated from. E.g.,
`"build_dictionary": "dist/builddict/build_dictionary.html",` means to
generate the site's /build\_dictionary.html page, the site generator
should get the "raw" file `dist/builddict/build_dictionary.html` and
apply the template to it.

`tree`:<br>
Outline structure of the site. Each node of the tree is a dict. Each
node-dict can have a `page`, a page defined in `sources` above. Each
node-dict can have a `children`, a list of more nodes.

`template`:<br>
Path to mustache template to apply to each page.

`extras`:<br>
Map of "extra" files to copy over. Handy for graphics, stylesheets, and
such.

`outdir`:<br>
Path to which to write the generated site.

To add a page and have it show up in the side navigation UI, add the
page to the `sources` dict and to the `tree` hierarchy.
