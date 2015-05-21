
# Okta Developer Site

Okta developer site (developer.okta.com) is a custom [Jekyll](http://jekyllrb.com/) site deployed on [GitHub
pages](https://pages.github.com/)

All API documentation submissions are welcome. To submit a change, fork this repo, commit your changes, and send us a
[pull request](http://help.github.com/send-pull-requests/)


## Setup Environment

Most of the content of the developer site is written in a format called
[kramdown](http://kramdown.gettalong.org/syntax.html) which is a flavor of Markdown with some enhanced features like the
ability to do footnote notations. We use Kramdown because the markup is both human readable and convertable in to fully
fledged HTML. If there's some functionality you can't implement in Kramdown you can always drop down into HTML as a
last resort.

Though kramdown gives us benefits of readbilty it comes at the cost of requiring a build step to preview the changes to
the site. Jekyll, handles this nicely by detecting changes to the file system and refreshing automatically. To setup
Jekyll to be able to build and view the site, follow the steps below: 

1. Install RVM if you don't already have it. On OSX `brew install rvm` or on linux `yum install rvm`. To ensure you have
   rvm installed properly run `rvm list` and ensure that there is a version marked as 'current'.
2. Clone repository `git clone git@github.com:okta/okta.github.io.git`
3. Go into project directory `cd okta.github.io`
4. You maybe told that the version of ruby needed by this project isn't installed, follow the given instruction to
   install the correct verion of ruby for RVM. If you did have to install a new version of ruby re-enter the directory
   you're in. `cd ..;cd okta.github.io` then verify that the current gemset you're using is called "okta.github.io" by
   running `rvm gemset list`. This project sepcifies a gemset so that the build of this project isn't affected by global
   gems on your system.
5. Now install gems needed by this project into the projects gemset with `gem install bundler` and then `bundle install`.
   This will essentially read the Gemfile in the root directory and install all packages required to run the site.
6. Start the site `bundle exec jekyll serve --watch`
8. Visit `http://localhost:4000` in your browser
9. To stop serving the site hit `ctrl+c` in the terminal

## Publishing Changes (the build process)

Though github pages do support an automatic build process, we use a custom toolbar gem which requires that we build the
site ourselves. This means that this git repo contains both the source files of the site and the compiled site files. As
we edit the site we should only make changes to the source files and then rely on the following build process to compile
and generate the final site content.

### Important Directories

* **_source** this is where the source files live. If you're editing content, or adding assets like images or css, they
  belong in here
* **_site** this directory is ignored in github. It contains the local version of each of the built files in the site
* **almost everything else** most of the other directories in the root are the checked in versions of the built site.
  These files should not be edited directly. Find the corresponding version of the file in the _source directory, modify
  that and then re-run the build.

### Build Steps

1. Create a topic branch or your work `git checkout -b <branch_name>`
2. Make changes / additions in _source directory
3. Compile changes into the _site directory `bundle exec jekyll serve --watch`
4. Navigate the site and validate your changes
5. Stop Jekyll with `ctrl+c`
6. Sync the built _site files with the checked-in code with: `rsync -av _site/ ./`
7. Git commit and push changes to github. When ready for review create a pull request and mention the users you want to
   review your changes.

## Authoring Guide

### Pages

Pages are single purpose html or markdown files

1. Create a folder with `index.html` file. (e.g. *pricing/index.html*)
2. Author front matter as follows:

 ```
---
layout: page
title: YOUR_TITLE
css: CSS_FILENAME.css (optional page specific css file)
---
```
3. Put html or css content under front matter
4. Expose link to page in header or elsewhere. URL will be the folder
   name, `index.html` is not necessary (e.g. */pricing*)

### Docs

1.  Create `PAGE_NAME.md` in `docs/FOLDER_NAME` (e.g. *docs/api/rest/users.md*)
2.  Filenames are underscore seperated and all lowercase. (e.g. *my_cool_doc_page.md*)
3.  Author front matter

 ```
---
layout: docs_page
title: Title Case Name of Page
---
```
4.  The content under the front matter should not have any `h1`s -
    this will be set by the `title` property
5.  All `h2`s in the content will be rendered as a link in the table
    of contents
6.  Create placement entry in manifest at `_data/docs.yml` in
    appropriate `pages` section.
7.  The `pages` name must match the filename without extension. (e.g. `users.md` => `users`)
8.  To create a new section in the manifest

 ```
section: NAME_OF_FOLDER
title: TITLE_CASE_NAME_OF_SECTION
```
9.  The sections structure should follow the folder names
10. The ordering of sections and pages in the manifest determines the order in
    which they appear in the leftnav

### Customers

1.  Create an `index.md` folder in `customers/CUSTOMER_NAME` (e.g. *customers/box/index.md*)
2.  Create the images:

```
`index_image`:     220px x 165px @144dpi  // image used on index page
`header_image`:   1080px x 220px @144dpi  // title bar image on story page
`diagram_image`:   694px x <n>px @144dpi  // information graphic at bottom of story page
```
3.  Author front matter

 ```
`index_blurb` will be clipped over three lines of text
`sidebar_copy` will be markdownified. Newlines sohuld be doubled.
```

### Authors

1.  Create an entry in _source/_data/authors.yml
2.  Put avatar image in _source/assets/img. Make sure aspect ratio of image is square.

### Blog Posts

1. Create a file in `_source/_posts/` with the form `YYYY-mm-dd-the-title.md`
2. The header should look something like:

```
---
layout: blog_post
title: Productionalizing ActiveMQ
author: okta_generic_blogger
tags: [activemq, jvm]
---
```

3. Supporting images should be placed in `_source/assets/img` and should follow the convetion:
   `<name_of_post>-image-name.extension` where name of the post doesn't include the ".md" extension. This will allow us
   to know which images are referenced by which post.
3. Look at other posts for examples.
