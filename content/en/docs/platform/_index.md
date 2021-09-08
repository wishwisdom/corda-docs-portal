---
section_menu: homepage
---

This link works:

* [Main documentation](platform/corda/4.8/os.html)

This link does not:

* [Main documentation](corda/4.8/_index.md)

The first link points to `platform/corda/4.8/os.html`. This is wrong because this file is already in the `platform` directory, and because it points to `os.html`, which doesn't exist until the website is generated.

The second link points to `corda/4.8/os/_index.md`. This is the right link, because it points to a markdown file that exists.

Normally, when hugo is working with page bundles, it rewrites all the links correctly when it renders the page bundle. For some reason that is not happening here.
