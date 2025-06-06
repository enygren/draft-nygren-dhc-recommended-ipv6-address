# Early Version

This is a first version of this draft for consideration as to suitability.
As such it is a work-in-progress that isn't yet even ready for publication
as a draft.


The repo uses Martin Thomson's [I-D template](https://github.com/martinthomson/i-d-template).

## Building the Draft

Formatted text and HTML versions of the draft can be built using `make`.

```sh
$ make
```

This requires that you have the necessary software installed.  See
[the instructions](https://github.com/martinthomson/i-d-template/blob/master/doc/SETUP.md). For this I-D, Markdown is the canonical source, so `kramdown-rfc2629` is what I use. This must be installed in addition to the right version of `make` and `xml2rfc` (yay, software) - please check the [SETUP doc](https://github.com/martinthomson/i-d-template/blob/master/doc/SETUP.md) for details. 

Also note that `main` is the default and working branch.

## Quick commands

Once you have everything set up correctly, the usual flow is:

```sh
$ make
```

This generates the `.txt` and `.html` outputs. 

To automatically fix lints:
```sh
$ make fix-lint
```

Then do a `git commit`. Note that a git commit will typically fail if the lint check fails.

### Submitting 

Current datatracker version is `6`. Next version will be `7`.

When you're ready to cut a new version, do:
```sh
make 
make next # this generates the versioned doc
# Once you're happy with it, upload the XML file generated in versioned/
# https://datatracker.ietf.org/submit/
# Then, upload the new tag to GitHub
git tag draft-name-07
git push origin draft-name-07
```

This is the manual process outlined in the [I-D template docs](https://github.com/martinthomson/i-d-template/blob/main/doc/SUBMITTING.md#manual-process). 
