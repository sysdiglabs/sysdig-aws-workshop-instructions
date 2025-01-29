# Kraken Hunter Instructor Guide

This repository is a set of instructions for Sysdig Kraken Hunter instructors to use when teaching the Kraken Hunter workshop. There are more moving parts to running a workshop, but part of it is these instructions which the users use to go through the actual workshop, performing actions and learning about Sysdig. 

This git repository is that set of instructions, and is what you would share with the users.

* URL: https://sysdiglabs.github.io/kraken-hunter-instructions/

Specifically you would share the "learning path" you want to use for your workshop.

## Developing Locally

This site uses Github pages, which are based on Jekyll, and a slightly modified version of the [just-the-docs](https://just-the-docs.github.io/just-the-docs/) theme.

Here are docs for developing locally with Jekyll and Github pages:

* https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll

It should be almost as simple as running the below command.

```bash
bundle exec jekyll serve
```

That would launch a local server at `http://localhost:4000` where you can see the changes you make.

## Developing a Module

The "modules" are just markdown! That's it, just a collection of material related to something that we want to demonstrate. There's some conventions used in the markdown files to make it easier for users to know what to do, but ultimately it's just markdown with some extra contextual markup.

### Best practices and conventions

- Add a `{: .goals }` callout to indicate the goals of the module at the beginning of the module.
- Add a `{: .value }` callout to indicate the value of the module at the end of the module.
- Use the various callouts to make it easier for users to know what to do, e.g. using `{: .sysdig-gui }` for Sysdig GUI instructions and `{: .command }` for CLI commands. We try to make it easy for users to identify the different types of content and know what they need to do to complete the module.
- Add borders around images:

```bash
convert image.png -bordercolor grey -border 2 image.png
```

- When setting up images, make sure to use the `{{ site.baseurl }}` variable to make sure the images are displayed correctly.

e.g.

```
![image]({{ site.baseurl }}/assets/images/image.png)
```

### Organization

The `docs/modules` directory is where the modules are stored.

The files are setup in order usually, with top levels of:

* Getting Started
  * Getting Your Attendee Information
  * Getting Access to AWS
  * etc, etc.
* Learning Paths
    * Full Day Kraken Hunter
    * Kubernetes
    * etc, etc.
* Modules
    * Real World Attacks
    * Kubernetes 
    * etc, etc.
* Certification

Most of the files are in the `docs/modules` directory.

At the top of each file you'll see some metadata, e.g.

```
---
title: Risks and Attack Path
parent: Modules
nav_order: 4
---
```

Modules have a `parent` which is the section they belong to, and a `nav_order` which is the order they should appear in the sidebar.

## Learning Paths

We are curating learning paths for Sysdig workshops using the `docs/learning-paths` directory. 

A learning path is just a set of links to modules and other content that you want to include in your specific workshop, or you can just use the existing learning paths.

Basically you create a markdown file with links to the modules you want to include in the learning path. It's very straightforward, and you can easily curate your own learning paths, your own workshops, etc.

As well, you could create a learning path using any documentation you'd like, whether it's a Google doc or something else--just setup some links to the Github pages documentation you'd like to use. Or, if you want it in this repository, add a markdown file to the `docs/learning-paths` directory and link to it from the `index.md` file in the `docs/learning-paths` directory.

## Github Forks

It is best to create a pull request to update this documentation.

* Create a fork of this repository
* Make your changes locally 
    * Test with `bundle exec jekyll serve` to be able to see the changes locally
* Create a pull request to this repository
* Once it's approved, you can merge

