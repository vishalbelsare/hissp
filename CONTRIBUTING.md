
## Contributing
There are many ways to contribute to an open-source project,
even without writing code.

Questions are allowed on the issue tracker,
they help illustrate deficiencies in our documentation,
but do check there first!

Bug reports are welcome.

PRs to help improve documentation,
structure, or compatibility will also be considered.

### Patches
PRs must be aligned with the philosophy and goals of the project to be
considered for inclusion.

PRs do not have to be *perfect* to be submitted,
but must be perfect enough to pass peer review before they are merged in.
Small, focused changes are more likely to be reviewed.

Changes to the source code must be properly formatted and have full test
coverage before the PR can be accepted.
Manual tests may suffice for configuration files.
Our Python source uses Black formatting.
Disable this using `# fmt: off` tags for "readerless mode" Hissp snippets
which should be formatted Lisp-style (play with Parinfer until you get it),
or anywhere the extra effort of manual formatting is worth it.
In readerless mode, Hissp tuples shall always include the trailing `,`.
Follow PEP 8 even when Black has no opinion.
Our .lissp source uses Emacs lisp-mode for indentation.
It must also pass Parlinter.

Documentation is expected to have correct (American English) spelling
and grammar. All Doctests must pass.

You can use pytest to run unittests and doctests at the same time.
Make sure you install the dev requirements first.
Hissp has no dependencies, but its test suite does.
```
$ pip install -r requirements-dev.txt
```
```
$ pytest --doctest-modules --cov=hissp
```

We merge to master without squashing.
Commits must be small enough to be reviewable.
We don't accept PRs on faith.

Note section 5 of the LICENSE.
You must have the legal rights to the patch to submit them under those terms:
either you own the copyright
(e.g. because you are the author of the patch and it was not a work for hire)
or have appropriate license to do it.

The git repository itself will normally suffice as a record of
authorship for copyright purposes.
Don't update the original boilerplate notices on each file.
But commits authored by or owned by someone else must be clearly labeled as such.
No plagiarism will be permitted,
even if you're copying something from the public domain.
We may maintain a NOTICE file per section 4.(d) of the LICENSE if needed.
