README Files and Markdown
=========================

You can write program documentation in the popular Markdown format;
Pants eases publishing your docs to places where your users can read it.
E.g., that `README.md` file in your source code is handy for editing;
but you might want to generate a web page from that so folks can decide
whether they want to look at your source code.

Markdown to HTML
----------------

Pants uses the Python `Markdown` module; thus, in addition to the usual
Gruber `Markdown` syntax, there are [other
features](http://pythonhosted.org/Markdown/) Pants uses Python
Markdown's `codehilite`, `extra`, `tables`, and `toc` extensions.

To tell Pants about your Markdown file, use a
<a xref="bdict_page">`page`</a>
target in a `BUILD` file as in this excerpt from
[examples/src/java/com/pants/examples/hello/main/BUILD](https://github.com/pantsbuild/pants/blob/master/examples/src/java/com/pants/examples/hello/main/BUILD):

!inc[start-after=README page](hello/main/BUILD)

To render the page as HTML, use the
<a xref="gref_goal_markdown">markdown goal</a>. For example, to view
`examples/src/java/com/pants/examples/hello/main/README.md` as HTML in
your browser,

    :::bash
    $ ./pants goal markdown --markdown-open examples/src/java/com/pants/examples/hello/main:readme

Pants generates the HTML files in the `dist/markdown/` directory tree.

Link to Another `page`
----------------------

One `page` can link to another. Regular Markdown-link syntax works for
regular links; but if you use `page`s to generate both `.html` files and
wiki pages, it's not clear what to link to: the `.html` file or the wiki
address. You can use a Pants-specific syntax to make links that work
with generated HTML or wiki pages.

To set up a page `source.md` that contains a link to `dest.md`, you make
a `dependencies` relation and use the special `pants(...)` syntax in the
markdown.

In the `BUILD` file:

    :::python
    page(name='source',
      source='source.md',
      dependencies=[':dest'], # enables linking
      provides=[...publishing info...],
    )

    page(name='dest',
      source='dest.md',
      provides=[...publishing info...],
    )

To set up the links in `source.md` that point to `dest.md` or an anchor
therein:

    For more information about this fascinating topic,
    please see [[Destinations|pants('path/to:dest')]],
    especially the
    [[Addendum section|pants('path/to:dest)'#addendum]],

Pants replaces the `pants('path/to:dest')` with the appropriate link.

Include a File Snippet
----------------------

Sometimes the best way to explain `HelloWorld.java` is to show an
excerpt from `HelloWorld.java`. You can use the `!inc` markdown to do
this. Specify a file to include and (optionally) regexps at which to
start copying or stop copying. For example, to include an excerpt from
the file `HelloMain.java`, starting with the first line matching the
pattern `void main` and stopping before a subsequent line matching
`private HelloMain`:

    !inc[start-at=void main&end-before=private HelloMain](HelloMain.java)

To include *all* of `HelloMain.java`:

    !inc(HelloMain.java)

To include most of `HelloMain.java`, starting after license boilerplate:

    !inc[start-after=Licensed under the Apache](HelloMain.java)

It accepts the following optional parameters, separated by ampersands
(&):

start-at=*substring*<br>
When excerpting the file to include, start at the first line containing
*substring*.

start-after=*substring*<br>
When excerpting the file to include, start after the first line
containing *substring*.

end-before=*substring*<br>
When excerpting the file to include, stop before a line containing
*substring*.

end-at=*substring*<br>
When excerpting the file to include, stop at a line containing
*substring*.

Publishing
----------

You can tell Pants to publish a page. So far, there's only one way to
publish: as a page in an Atlassian Confluence wiki. (You can add other
doc-publish backends to Pants;
[[send us a patch|pants('src/python/pants/docs:howto_contribute')]]!)

To specify the "address" to which to publish a page, give it a
`provides` parameter.

Once you've done this, you can publish the page by invoking the goal
<a xref="page_setup_confluence">set up in your workspace</a>. For example, if
the goal was set up with the name "confluence", you invoke:

    :::bash
    $ ./pants goal confluence examples/src/java/com/pants/examples/hello/main:readme

To specify that a page should be published to a Confluence wiki page,
set its `provides` to something like:

    :::python
    page(...
      provides=[
        wiki_artifact(wiki='//:confluence',
          space='ENG',
          title='Pants Hello World Example',
          parent='Examples',
        )
      ],)

...assuming your workspace is set up with a wiki target named
`confluence` in a top-level `BUILD` file (as described below).

<a xmark="page_setup_confluence"></a>

**Setting up your workspace for Confluence publish**

That `wiki` specifies some information about your wiki
server. So far, the only kind of thing you can publish to is a
Confluence wiki. You want to specify its target. Do this in one of the
`BUILD` files that's always processed (probably a top-directory `BUILD`
file). If your Confluence server lived at wiki.archie.org, the target
would probably look something like:

    :::python
    import urllib

    def confluence_url_builder(page, config):
      return config['title'], 'https://wiki.archie.org/display/%s/%s' % (
        config['space'],
        urllib.quote_plus(config['title']))

    # Use this wiki target's address for the wiki= param in your wiki_artifacts
    confluence = wiki(name="confluence",
                      url_builder=confluence_url_builder)

You need to install a goal to enable publishing a doc to confluence. To
do this, create a Pants plugin that installs a goal that subclasses
ConfluencePublish.

In your `pants.ini` file, add a section with the url of your wiki
server. E.g., if your server is at wiki.archie.org, it would look like:

    [confluence-publish]
    url: https://wiki.archie.org