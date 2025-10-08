# manget -- fetch and store a remote readme as local man page

## Synopsis

```shell
manget [options]
manget some/github-repo [exename]
```

## Description

Fetch a man page from project `README.md` file, convert it to roff (man page
format), and store it locally (default `~/.local/share/man/...`) for instant
viewing.

<!-- This could just be a wrapper around `eget` that calls `eget` and then uses the -->
<!-- target given to it to determine the README.md file, and curl it, then do the -->
<!-- ronn etc steps. -->

## Background

> Why not just read the docs in the browser??

Because you're gonna google or LLM it, context switch, trigger Gemini (using a
day's worth of household electricity), get served ads, wind up _maybe_ on the
page you wanted, forget to close that new tab that's eating 250 MB in your
already crawling browser, and forget what you were even doing. OR, you could
just hit `Ctrl-h` in your terminal and be on your merry way.

> But READMEs are different than man pages!

Actually, I have found that the majority of popular projects' READMEs are
loaded with great "usage" and other content. Granted, there are some unwanted
"installation" etc sections, but they are omitted with `manget`.

## Built-In Help Systems

I consider there to be three levels of built-in local help: here in order of
access.

### 1. Tab completion

In Zsh (and other shells) the completion system is nearly comprehensive. In
fact, this is probably the number one reason I remain using Zsh (among
others). Many modern CLI-parsers libs can even generate completion files for
Zsh. Whenever you can't find one, you can ask Zsh to generate a pretty good
one simply with (for `ronn` command in this example):

```shell
% compdef _gnu_generic ronn # collect these in your .zshrc equivalent

% ronn -«TAB»
--- option
--date              -- published date in YYYY-MM-DD format (bottom-center)
-E                  -- specify the encoding files are in (default is UTF-8)
--fragment      -f  -- generate HTML fragment
--help              -- show this help message
...
```

This will display a short one-line description for each option. That's often
all you need as a refresher, once you know a tool well enough.

### 2. Help option

Pretty much every CLI now features a pair of `-h`/`--help` options. (Some
tools like `git` take it a step further and distinguish those two, opening a
man page for the latter.) So `-h` is the go-to second level help that tells
you more about its args and basic usage.

### 3. Man page

Man pages are very helpful documents to tell you all about the command. Try
`man ls` to see a fine example of `ls`'s section-1 page (there can be many
sections, each with designated purpose). There are 30,000 man pages on my
semi-minimal system without even trying! The problem is that many newer tools
don't have man pages! And that's what `manget` is all about. Because they do
all have at least READMEs that tend to serve about the same purpose.

The beauty of modern language tooling (golang, rust, etc) is that they can
build standalone executables, instead of a tedious and breakage-prone things
like `pip install`. There are even tools like
[eget](https://github.com/zyedidia/eget) that can install such tools trivially
in one stroke. But then there are no docs installed. This tends to be the
difference between `brew/dnf/apt/pacman install ...` -- those usually install
man pages.

### 4. tldr

OK, there is a fourth level, if missing from man pages: concise plain
examples. This is a little outside my scope since it's an entirely different
set of tools and doc sources, but it's worth mentioning, and you should set it
up. I have found [tealdeer](https://github.com/tealdeer-rs/tealdeer) to be a
nice, small, fast implementation with great completions. Maybe worth aliasing.

```shell
% alias eg=tldr
```

**IDEA:** It might be a nice effort to turn all of the tldr listings (which
are just markdown) into a special section of man pages, so that you could
instead say: `man 1e git-log`. Then they could all (~5k pages)  i be packaged and
be part of the single man system.

## Implementation

The simplest way to get man pages (or something close in spirit to them) onto
your system for anything installed is something like this recipe:

```shell
dest=~/.local/share/man sect=1
proj='walles/moor'
host='https://github.com/'
cmdname='Moor Pager'
exe='moor'
# eget $proj
curl $host/$proj/README.md |
  ronn --section $sect -r --name $cmdname --manual 'FIXME Manual Name' --pipe |
  gzip >$exe.$sect.gz
```

I'm planning to write a small go-based utility to do these steps and avoid
having to install `ronn` etc. But for now there's a little POC Zsh script
(`manget`) to run these.

## Installation

(Note that this section is omitted from the `manget` man page!)

There are presently some deps to install manually:

- [ronn]

`manget` is just a tiny Zsh script. Put it on your `PATH`.

```shell
% eget   micahelliott/manget
OR
% wget https://raw.githubusercontent.com/MicahElliott/manget/main/bin/manget
```

## Usage

```shell
% manget walles/moor 
```

### Zsh `run-help` setup

You can set up Zsh to auto-run `man` (and retain your whole command line):

```shell
% cat -A foo.txt«Alt-h» # invokes: man cat
```

[Here's a short setup guide.](https://stackoverflow.com/a/46415388)

### During development, to preview

```shell
% ronn ... --pipe >myapp.1
% man ./myapp.1
```

## Primer on Man

Man pages live in places like `/usr/share/man/...` and are automatically found
by `man`. More are enabled via `MANPATH` (works like `PATH`).

The [ronn]() page describes well why man pages are so valuable.

```shell
% man man

% man 1 ls # the section-1 page listing

% man -f crontab # See what sections are available
crontab (1) - maintains crontab files for individual users
crontab (5) - files used to schedule the execution of programs
```

## Organizing Your Man Pages

You can organize your docs into multiple "pages", or just have a single page
that contains "sections", typically `(1)` and maybe `(5)`.

### Multiple pages

Some commands break their docs into multiple man pages. Here's Zsh:

```shell
% man zsh«TAB»
--- manual page
zsh          zshbuiltins  zshcompctl     zshcompwid   zshexpn      zshmodules
zshparam     zshtcpsys    zshzle         zshcalsys    zshcompsys   zshcontrib
zshmisc      zshoptions   zshroadmap     zshzftpsys   zshall
```

### Man Sections

Most projects organize docs into "sections", though the default section `(1)` is
usually the one you're looking for. (C, Perl, and maybe others use man pages
to document libs in sections like `(3)`). Specs, formats, conventions go in
`(5)`. Use `man 5 crontab` to see a spec.

### Page Sections

Not to be confused with the "sections" above! These are just the `H2`s in your
doc.

From `man man`:
> Conventional section names include NAME, SYNOPSIS, CONFIGURATION,
> DESCRIPTION, OPTIONS, EXIT STATUS, RETURN VALUE, ERRORS, ENVIRONMENT, FILES,
> VERSIONS, CONFORMING TO, NOTES, BUGS, EXAMPLE, AUTHORS, and SEE ALSO.

These are also a great rubric for READMEs! (though may need to add
INSTALLATION)

## Exemplary Man Pages

There are some great man pages that can be modeled after. Here are a few:

- `jq`
- `fzf`

## Pagers

`man` runs its output through a "pager". You can change this by exporting
`PAGER` or `MANPAGER`. Here are some common ones:

- less
- bat
- moor (eget)
- [most](https://www.jedsoft.org/most/)
- info
- woman (part of emacs)

## MANPATH

```shell
mkdir -p ~/.local/share/man/man{1..8}
export MANPATH+=~/.local/share/man
```

## Setting Up Keybindings

If you like Emacs key bindings, here's a way to get those: `man lesskey`,
see [example config](https://github.com/dLuna/config/blob/master/.lesskey).

Or you could use `info` which falls back to man pages when no info page exists.

## Setting Up Colors

https://unix.stackexchange.com/questions/119/colors-in-man-pages

## Conversion from Markdown To Roff

- [ronn](https://github.com/rtomayko/ronn)
  Install: `sudo dnf install rubygem-ronn-ng`

- `pandoc README.md -s -t man >myproj.1`

## Tips/assumptions on READMEs

- It is assumed that the README you wish to process has the main sections
  grouped into second level headings (`H2`).
- Start with _Synopsis_, then _Descrption_ sections.
- Put an _Examples_ section at the end.
- Use `manget` to package a `yourtool.1` file that is available on your
  _Realease_ pages.
- Use [markdownlint](https://github.com/markdownlint/markdownlint)
- Use [flymake-mdl](https://github.com/MicahElliott/flymake-mdl) if Emacs

## Beyond the README

A README's purpose is typically to get you up and running, with some sales
patch and explanation. Sometimes that's all that's needed. But in practice,
READMEs are a gold mine of background, usage, FAQs, etc.

There are other types and sources of docs for a given project: blog post,
dedicated project page, GH wiki, etc.

The [Diátaxis](https://diataxis.fr/start-here) approach is widely adopted and
suggests a taxonomy. Man pages tend to be of _Reference_ type, somewhat like
API docs; whereas READMEs are often a mix of at least a couple types, leaning
toward _Explanation_.

## Packaging a Set of Man Pages into Your Releases

As part of releasing your binaries, you can also push a `docs.tgz` file.

## Related Tooling

There is already a way to view a github project readme. It looks pretty and
does the job. However, it's slow, requires authentication, is tedious to
type, is not discoverable, and has to fetch every time you reference it.

```shell
% gh repo view rtomayko/ronn
```

- [eget]()
- [tldr]()
- [gh]()
- [mdbook-man](https://github.com/vv9k/mdbook-man) -- convert mdbook to man page

## License

This project is licensed under the terms of the MIT license.
