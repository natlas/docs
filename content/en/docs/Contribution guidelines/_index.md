---
title: "Contribution Guidelines"
linkTitle: "Contribution Guidelines"
weight: 10
description: >
  How to contribute to the docs
---

{{% pageinfo %}}
These are basic guidelines for contributing to the Natlas documentation. For information about contributing to the Natlas project, please see the [Natlas Contributing Guidelines](https://github.com/natlas/natlas/blob/main/CONTRIBUTING.md).
{{% /pageinfo %}}

We use [Hugo] to format and generate our website, the
[Docsy] theme for styling and site structure,
and [Github Pages] to manage the deployment of the site.
Hugo is an open-source static site generator that provides us with templates,
content organisation in a standard directory structure, and a website generation
engine. You write the pages in Markdown (or HTML if you want), and Hugo wraps them up into a website.

All submissions, including submissions by project members, require review. We
use GitHub pull requests for this purpose. Consult
[GitHub Help] for more
information on using pull requests.

## Quick Start

Here's a quick guide to updating the docs. It assumes you're familiar with the
GitHub workflow and you're happy to use the automated preview of your doc
updates:

1. Fork the [docs.natlas.io repo] on GitHub.
1. Make any changes you'd like to propose.
1. Test locally using hugo to build the site. A docker-compose file is provided so that you can simply `docker-compose up hugo` and visit `http://localhost:1313`.
1. If it all looks good, go ahead and send a pull request (PR).
1. If you're not yet ready for a review, add "WIP" to the PR name to indicate
  it's a work in progress and open it as a draft.
1. Continue updating your doc and pushing your changes until you're happy with
  the content.
1. When you're ready for a review, add a comment to the PR, remove any
  "WIP" markers, and change the PR status to "Ready for Review."

## Updating a single page

If you've just spotted something you'd like to change while using the docs, we have a shortcut for you:

1. Click **Edit this page** in the top right hand corner of the page.
1. If you don't already have an up to date fork of the project repo, you are prompted to get one - click **Fork this repository and propose changes** or **Update your Fork** to get an up to date version of the project to edit. The appropriate page in your fork is displayed in edit mode.
1. Follow the rest of the [Quick Start](#quick-start) process above to make, preview, and propose your changes.

## Creating an issue

If you've found a problem in the docs, but you're not sure how to fix it yourself, please create an issue in the [docs.natlas.io repo]. You can also create an issue about a specific page by clicking the **Create Issue** button in the top right hand corner of the page.

## Useful resources

* [Docsy user guide]: All about Docsy, including how it manages navigation, look and feel, and multi-language support.
* [Hugo documentation]: Comprehensive reference for Hugo.
* [Github Hello World]: A basic introduction to GitHub concepts and workflow.

[Hugo]: https://gohugo.io/
[Docsy]: https://github.com/google/docsy
[Github Pages]: https://pages.github.com/
[Github Help]: https://help.github.com/articles/about-pull-requests/
[docs.natlas.io repo]: https://github.com/natlas/docs.natlas.io
[Docsy user guide]: https://www.docsy.dev/docs/
[Hugo documentation]: https://gohugo.io/documentation/
[Github Hellow World]: https://guides.github.com/activities/hello-world/
