---
layout: post
category: interest
title: Setup-X GitHub actions using typescript
---

GitHub actions are useful snippets to use inside CI workflows on the GitHub
platform. Typically these provide common functionality that can be reused across
different projects and workflows, such as the [checkout] and [cache]
actions. Many actions are also available to set up the tools required inside
workflows, such as the GitHub provided [setup-python]. These actions
ensure that the required version of the tool is provided and available on the
`PATH` for subsequent steps in the workflow.

As many open source projects now provide pre-built binary releases on GitHub, it
is easy to provide our own setup action that provides these binaries. Such an
action will query the releases available on GitHub, download the requested
binary and provide it to the workflow through the [tool cache].

Examples of 

<!--end-excerpt-->

### Parsing GitHub releases

### Using the tool cache



[checkout]: https://github.com/marketplace/actions/checkout
[cache]: https://github.com/marketplace/actions/cache
[tool cache]: https://github.com/actions/toolkit/tree/master/packages/tool-cache
