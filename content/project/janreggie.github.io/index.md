---
title: "janreggie.github.io: internals of janreggie's sandbox"
type: project
date: 2020-09-21T16:11:13+08:00
publishdate: 2020-09-21
lastmod: 2021-07-20
draft: false
description: "Writing this very website using Hugo, a Bootstrap template, modifying some templates for personal use, and hosting on GitHub Pages"
tags:
  - "hugo"
  - "html"
  - "bootstrap"
  - "web development"
categories:
  - "blog"
---

janreggie's sandbox is built using [Hugo](https://gohugo.io/),
[bootstrap-bp-hugo-theme](https://themes.gohugo.io/bootstrap-bp-hugo-theme/),
external fonts, and tears and sweat from [the author](https://github.com/janreggie/).
In this post I describe the steps I made in building this website.

<!--more-->

## Choosing and customizing a theme

There are [a lot of Hugo themes](https://themes.gohugo.io/) out there.
I was looking for one that is relatively simple but can be extended quite well.

I settled with bootstrap-bp-hugo-theme for several reasons:

- It has built-in i18n support, which I can utilize in a future time.
- It's based on [Bootstrap](https://getbootstrap.com/), which contains a plethora of classes
- It has cards out of the box for one's homepage, which looks pleasant to the user
- It has best practices supported out of the box
- It has a dark theme which I can use for a later project

With all of that,
I installed the theme using the following command:

```sh
git submodule add https://github.com/spech66/bootstrap-bp-hugo-theme.git themes/bootstrap-bp-hugo-theme
```

I also based my `config.toml` using the `exampleSite/config.toml` provided.
I only did minimal configuration here, such as updating socials
and setting up a Markup theme.

I copied the contents of the theme's
`assets`, `archetypes`, `i18n`, `layouts`, and `static` folder
into my project folder.
It's time to do some customizing.

### New type: `project`

The `project` type is intended to be used for projects.
Establishing it is simple enough: creating a `project.md` on my archetypes folder
such that when I run `hugo new project/PROJECT_NAME/index.md`, it will use said archetype.

### Structural changes

#### `layouts/index.html`: Adding a header image and a list of projects

The first thing I wanted in my Index is a header image containing a welcoming message and a lede.
Immediately below that I wanted the list of projects
taken by listing all sites with `Type == project`.

```html
{{ define "main" }}

<div class="container mt-3 mb-3">
  <div class="jumbotron text-center" id="index-header">
  <h1 class="display-4">{{ .Site.Params.header }}</h1>
  <p class="lead">{{ .Site.Params.lede }}</p>
  </div>
  <div id="projects-list">
  <h1>{{ i18n "projects" }}</h1>
  <div class="container mt-3 mb-3">
    {{ $paginator := .Paginate ( where .Site.RegularPages.ByPublishDate.Reverse "Type" "==" "project") }}
    <div class="card-columns">
    {{ range $paginator.Pages }}
      {{- partial "content_index.html" . -}}
    {{ end }}
    </div>
    {{ template "_internal/pagination.html" . }}
  </div>
  </div>
</div>

{{ end }}
```

Customizing the jumbotron is done through an external CSS file

```css
#index-header {
  height: 70vh;
  background-image: url("/header_image.jpg");
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center center;
}
```

#### `partials/content_card_body.html`: Adding support for project link

My new project type may contain a `link` parameter which links to the project itself,
just like the link type for the original theme.
I wanted my "cards" to enable accepting a `link` param but only if it exists.

```html
{{ if eq .Type "video" }}
{{ else if eq .Type "audio" }}
{{ else if eq .Type "link" }}
  <p class="text-center"><a href="{{ .Params.link }}" target="_blank" rel="noopener noreferrer" class="btn btn-primary btn-lg" tabindex="-1" role="button"><i class="fas fa-link"></i>&nbsp;{{ .Title }}</a></p>
{{ else if and (eq .Type "project") (.Params.link)  }} <!-- projects that have external links (most of the time they should have one!) -->
  <p class="text-center"><a href="{{ .Params.link }}" target="_blank" rel="noopener noreferrer" class="btn btn-primary btn-lg" tabindex="-1" role="button"><i class="fas fa-globe"></i>&nbsp;{{ index (split .Title ":") 0 }}</a></p>
{{ else if eq .Type "quote" }}
  <!-- some code here -->
{{ end }}
```

#### Having a summary by placing `<!--more-->`

How Hugo generates summaries is by using the text above the
`<!--more-->` comment in one's page.
If a page has no summary, according to the theme,
it will try to create a summary by choosing the first few sentences.

### Thematic changes

#### Adding Overlock and Kumbh Sans

[Overlock](https://fonts.google.com/specimen/Overlock) and [Kumbh Sans](https://fonts.google.com/specimen/Kumbh+Sans)
are excellent fonts that I have found on Google Fonts.
Including them in `custom.css` is relatively simple:

```css
@import url('https://fonts.googleapis.com/css?family=Overlock|Kumbh Sans&display=swap');
```

Putting them into my website is also simple:

```css
body {
  font-family: 'Kumbh Sans', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
}

h1, h2, h3, h4, h5, h6, .h1, .h2, .h3, .h4, .h5, .h6 {
  font-family: 'Overlock', 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif
}
```

#### Adding Fantasque Sans

Getting [Fantasque Sans](https://github.com/belluzj/fantasque-sans) to work is a bit more difficult
since importing it from the instructions at [Font Library](https://fontlibrary.org/en/font/fantasque-sans-mono)
doesn't work since it pulls TTF files and [Firefox does not like that](https://stackoverflow.com/a/26695077/14020202).

Instead, what I did is I downloaded the fonts in `/static/webfonts`,
placed the `Fantasque...-decl.css` files in `/assets/css`,
modified them a bit such that they point to `/webfonts/Fantasque...`,
and included the following in my `/layouts/head_custom.html`:

```html
<!-- Install FantasqueSans because somehow I can't just import it in custom.css -->
{{ $fantasqueBold := resources.Get "/css/FantasqueSansMono-Bold-decl.css" }}
{{ $fantasqueBoldItalic := resources.Get "/css/FantasqueSansMono-BoldItalic-decl.css" }}
{{ $fantasqueItalic := resources.Get "/css/FantasqueSansMono-Italic-decl.css" }}
{{ $fantasqueRegular := resources.Get "/css/FantasqueSansMono-Regular-decl.css" }}
{{ $fantasque := slice $fantasqueBold $fantasqueBoldItalic $fantasqueItalic $fantasqueRegular | resources.Concat "/css/fantasque.css" | minify | fingerprint "sha512" }}
<link rel="stylesheet" href="{{ $fantasque.RelPermalink }}" integrity="{{ $fantasque.Data.Integrity }}">
```

Then I placed this in my `custom.css`:

```css
pre, code {
  font-family: 'Fantasque Sans Mono', 'Courier New', Courier, monospace;
}
```

I later decided to "deprecate" Fantasque Sans Mono in my post [Using System Fonts]({{< relref "/post/using-system-fonts">}}).

## git pre-commit hook to check for errors

I created a `.git/hooks/pre-commit` file to check if there are any errors
during compilation of the website:

```fish
#!/usr/bin/env fish

# just compile to hugo and check if it goes well
echo "Compiling... Please do wait."
hugo --debug --verboseLog
```

If the script returns a non-zero value,
which happens if there is an error during compilation,
it will not be able to commit properly.

Of course, this is not foolproof.
Suppose that I have a `{{ relref }}` somewhere
that points to a file that hasn't been committed yet.
The hook will be executed successfully since the file is present on the filesystem,
but in reality it is not present in the git folder's perspective.

## Publishing on GitHub Pages

I created a new workflow at `.github/workflows/deploy.yml`
such that every push on `master` will cause `hugo` to run
which will host the website on the `gh-pages` branch.

```yml
# Code from https://github.com/peaceiris/actions-hugo
# Licensed under MIT
name: deploy

on:
  push:
    branches:
      - master  # Set a branch to deploy

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.75.1'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          publish_branch: gh-pages
```

I have to confess that I spent an hour or so debugging why this isn't working.
I did not realize that I set `branches` here to `main` instead of `master`,
just like in the example in its page.

## Further improvements

Other improvements on janreggie's sandbox are listed in
[the issues page](https://github.com/janreggie/janreggie.github.io/issues).
For more complex changes, I may write separate pages for those in this very site,
mostly under the [hugo tag](/tags/#hugo).
