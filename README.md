<!--
Copyright 2019, 2020, 2021 Matthew Egan Odendahl
SPDX-License-Identifier: Apache-2.0
-->
[![Gitter](https://badges.gitter.im/hissp-lang/community.svg)](https://gitter.im/hissp-lang/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)
[![Documentation Status](https://readthedocs.org/projects/hissp/badge/?version=latest)](https://hissp.readthedocs.io/en/latest/?badge=latest)
[![codecov](https://codecov.io/gh/gilch/hissp/branch/master/graph/badge.svg)](https://codecov.io/gh/gilch/hissp)
<!-- Hidden doctest requires basic macros for REPL-consistent behavior.
#> (operator..setitem (globals) '_macro_ (types..SimpleNamespace : :** (vars hissp.basic.._macro_)))
>>> __import__('operator').setitem(
...   globals(),
...   '_macro_',
...   __import__('types').SimpleNamespace(
...     **vars(
...         __import__('hissp.basic',fromlist='?')._macro_)))

-->
![Hissp](https://raw.githubusercontent.com/gilch/hissp/master/docs/hissp.svg)

It's Python with a *Lissp*.

Hissp is a modular Lisp implementation that compiles to a functional subset of
Python—Syntactic macro metaprogramming with full access to the Python ecosystem!

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [Installation](#installation)
- [Show Me Code!](#show-me-code)
    - [Readerless Mode](#readerless-mode)
    - [Lissp](#lissp)
    - [Hebigo](#hebigo)
- [Features and Design](#features-and-design)
    - [Radical Extensibility](#radical-extensibility)
    - [Minimal implementation](#minimal-implementation)
    - [Interoperability](#interoperability)
    - [Useful error messages](#useful-error-messages)
    - [Syntax compatible with Emacs' `lisp-mode` and Parlinter](#syntax-compatible-with-emacs-lisp-mode-and-parlinter)
    - [Standalone output](#standalone-output)
    - [REPL](#repl)
    - [Same-module macro helpers](#same-module-macro-helpers)
    - [Modularity](#modularity)

<!-- markdown-toc end -->

# Installation

Install the latest PyPI release with
```
$ pip install hissp
```
Or install the bleeding-edge version directly from GitHub with
```
$ pip install git+https://github.com/gilch/hissp
```

# Show Me Code!

## Readerless Mode
Hissp is a *metaprogramming* language composed of Python data structured into trees of tuples,
```python
>>> hissp_code = (
... ('lambda',('name',),
...  ('print',('quote','Hello'),'name',),)
... )

```
which are compiled to Python code,
```python
>>> from hissp.compiler import readerless
>>> python_code = readerless(hissp_code)
>>> print(python_code)
(lambda name:
  print(
    'Hello',
    name))

```
and evaluated by Python.
```python
>>> eval(python_code)('World')
Hello World

```

## Lissp

The Hissp data-structure language can be written directly in Python using the "readerless mode" demonstrated above,
or it can be read in from a lightweight textual language called *Lissp* that represents the Hissp.
```python
>>> lissp_code = """
... (lambda (name)
...   (print 'Hello name))
... """

```
As you can see, this results in exactly the same Hissp code as the previous example.
```python
>>> from hissp.reader import Lissp
>>> next(Lissp().reads(lissp_code))
('lambda', ('name',), ('print', ('quote', 'Hello'), 'name'))
>>> _ == hissp_code
True

```

Hissp comes with a basic REPL (read-eval-print loop, or interactive command-line interface)
which compiles Hissp (read from Lissp) to Python and passes that to the Python REPL for execution.

Lissp can also be read from ``.lissp`` files,
which compile to Python modules.

Here's one definition from the basic macros:
```Lisp
(defmacro attach (target : :* args)
  "Attaches the named variables as attributes of the target.

  Positional arguments use the same name as the variable.
  Names after the ``:`` are identifier-value pairs.
  "
  (let (iargs (iter args)
        $target `$#target)
    (let (args (itertools..takewhile (lambda (a)
                                       (operator..ne a ':))
                                     iargs))
      `(let (,$target ,target)
         ,@(map (lambda (arg)
                  `(setattr ,$target ',arg ,arg))
                args)
         ,@(map (lambda (kw)
                  `(setattr ,$target ',kw ,(next iargs)))
                iargs)
         ,$target))))
```
If you've never used a Lisp before, don't let this scare you.
You should be able to read this much after completing the
[tutorials](https://hissp.readthedocs.io/).

## Hebigo

Hissp is modular, and the reader included for Lissp is not the only one.
Here's a native unit test class from the separate
[Hebigo](https://github.com/gilch/hebigo) prototype,
a Hissp reader and macro suite implementing a language designed to resemble Python:
```python
class: TestOr: TestCase
  def: .test_null: self
    self.assertEqual: () or:
  def: .test_one: self x
    :@ given: st.from_type: type
    self.assertIs: x or: x
  def: .test_two: self x y
    :@ given:
      st.from_type: type
      st.from_type: type
    self.assertIs: (x or y) or: x y
  def: .test_shortcut: self
    or: 1 (0/0)
    or: 0 1 (0/0)
    or: 1 (0/0) (0/0)
  def: .test_three: self x y z
    :@ given:
      st.from_type: type
      st.from_type: type
      st.from_type: type
    self.assertIs: (x or y or z) or: x y z
```

The same Hissp macros work in readerless mode, Lissp, and Hebigo, and can be written in any of these.
Given Hebigo's macros, the class above could be written in the equivalent way in Lissp:

```Lisp
(class_ (TestOr TestCase)
  (def_ (.test_null self)
    (self.assertEqual () (or_)))
  (def_ (.test_one self x)
    :@ (given (st.from_type type))
    (self.assertIs x (or_ x)))
  (def_ (.test_two self x y)
    :@ (given (st.from_type type)
              (st.from_type type))
    (self.assertIs .#"x or y" (or_ x y)))
  (def_ (.test_shortcut self)
    (or_ 1 .#"0/0")
    (or_ 0 1 .#"0/0")
    (or_ 1 .#"0/0" .#"0/0"))
  (def_ (.test_three self x y z)
    :@ (given (st.from_type type)
              (st.from_type type)
              (st.from_type type)
    (self.assertIs .#"x or y or z" (or_ x y z)))))
```

Hebigo looks very different from Lissp, but they are both Hissp!
If you quote this Hebigo code and print it out,
you get Hissp code, just like you would with Lissp.

In Hebigo's REPL, that looks like
```
In [1]: pprint..pp:quote:class: TestOr: TestCase
   ...:   def: .test_null: self
   ...:     self.assertEqual: () or:
   ...:   def: .test_one: self x
   ...:     :@ given: st.from_type: type
   ...:     self.assertIs: x or: x
   ...:   def: .test_two: self x y
   ...:     :@ given:
   ...:       st.from_type: type
   ...:       st.from_type: type
   ...:     self.assertIs: (x or y) or: x y
   ...:   def: .test_shortcut: self
   ...:     or: 1 (0/0)
   ...:     or: 0 1 (0/0)
   ...:     or: 1 (0/0) (0/0)
   ...:   def: .test_three: self x y z
   ...:     :@ given:
   ...:       st.from_type: type
   ...:       st.from_type: type
   ...:       st.from_type: type
   ...:     self.assertIs: (x or y or z) or: x y z
   ...: 
('hebi.basic.._macro_.class_',
 ('TestOr', 'TestCase'),
 ('hebi.basic.._macro_.def_',
  ('.test_null', 'self'),
  ('self.assertEqual', '()', ('hebi.basic.._macro_.or_',))),
 ('hebi.basic.._macro_.def_',
  ('.test_one', 'self', 'x'),
  ':@',
  ('given', ('st.from_type', 'type')),
  ('self.assertIs', 'x', ('hebi.basic.._macro_.or_', 'x'))),
 ('hebi.basic.._macro_.def_',
  ('.test_two', 'self', 'x', 'y'),
  ':@',
  ('given', ('st.from_type', 'type'), ('st.from_type', 'type')),
  ('self.assertIs', '((x or y))', ('hebi.basic.._macro_.or_', 'x', 'y'))),
 ('hebi.basic.._macro_.def_',
  ('.test_shortcut', 'self'),
  ('hebi.basic.._macro_.or_', 1, '((0/0))'),
  ('hebi.basic.._macro_.or_', 0, 1, '((0/0))'),
  ('hebi.basic.._macro_.or_', 1, '((0/0))', '((0/0))')),
 ('hebi.basic.._macro_.def_',
  ('.test_three', 'self', 'x', 'y', 'z'),
  ':@',
  ('given',
   ('st.from_type', 'type'),
   ('st.from_type', 'type'),
   ('st.from_type', 'type')),
  ('self.assertIs',
   '((x or y or z))',
   ('hebi.basic.._macro_.or_', 'x', 'y', 'z'))))
```

Want more examples? See the [Hissp documentation](https://hissp.readthedocs.io/) for the quickstart and tutorials.

# Features and Design

## Radical Extensibility

Python is already a really nice language, so why do we need Hissp?

> *Any sufficiently complicated C or Fortran program contains an ad hoc,
informally-specified, bug-ridden, slow implementation of half of Common Lisp.*  
— Greenspun's Tenth Rule

If the only programming languages you've tried are those designed to feel familiar to C programmers,
you might think they're all the same.

I assure you, they are not.

While any Turing-complete language has equivalent theoretical power,
they are not equally *expressive*.
They can be higher or lower level.
You already know this.
It's why you don't write assembly language when you can avoid it.
It's not that assembly isn't powerful enough to do everything Python can.
Ultimately, the machine only understands machine code.
The best programming languages have some kind of expressive superpower.
Features that lesser languages lack.

Lisp's superpower is *metaprogramming*,
and it's the power to copy the others.
It's not that Python can't do metaprogramming at all.
(Python is Turing complete, after all.)
You can already do all of this in Python,
and more easily than in lower languages.
But it's too difficult (compared to Lisp),
so it's done rarely and by specialists.
The use of `exec()` is frowned upon.
It's easy enough to understand, but hard to get right.
Python Abstract Syntax Tree (AST)
manipulation is a somewhat more reliable technique,
but not for the faint of heart.
Python AST is not simple, because Python isn't.

Python really is a great language to work with.
"Executable pseudocode" is not far off.
But it is too complex to be good at metaprogramming.
By stripping Python down to a minimal subset,
and encoding that subset as data structures rather than text,
Hissp makes metaprogramming as easy as
the kind of data manipulation you already do every day.
On its own, meta-power doesn't seem that impressive.
But the powers you can make with it can be.
Those who've mastered metaprogramming wonder how they ever got along without it.

Actively developed languages keep accumulating features,
Python included.
Often they're helpful, but sometimes it's a misstep.
The more complex a language gets,
the more difficult it becomes to master.

Hissp takes the opposite approach: extensibility through simplicity.
Major features that would require a new language version in lower languages
can be a library in a Lisp.
It's how Clojure got Goroutines like Go and logic programming like Prolog,
without changing the core language at all.
The Lissp reader and Hissp compiler are both extensible with macros.

It's not just about getting other superpowers from other languages,
but all the minor powers you can make yourself along the way.
You're not going to campaign for a new Python language feature
and wait six months for another release
just for something that might be nice to have for you special problem at the moment.
But in Hissp you can totally have that.
You can program the language itself to fit your problem domain.

Once your Python project is "sufficiently complicated",
you'll start hacking in new language features just to cope.
And it will be hard,
because you'll be using a language too low-level for your needs,
even if it's a relatively high-level language like Python.

Lisp is as high level as it gets.
You're going to need it.
Why settle for anything less?

## Minimal implementation
Hissp serves as a modular component for other projects.
The language and its implementation are meant to be small and comprehensible
by a single individual.

The Hissp compiler should include what it needs to achieve its goals,
but no more. Bloat is not allowed.
A goal of Hissp is to be as small as reasonably possible, but no smaller.
We're not code golfing here; readability still counts.
But this project has *limited scope*.
Hissp's powerful macro system means that additions to the compiler are
rarely needed.
Feature creep belongs in external libraries,
not in the compiler proper.

Hissp compiles to an unpythonic *functional subset* of Python.
This subset has a direct and easy-to-understand correspondence to the Hissp code,
which makes it straightforward to debug, once you understand Hissp.
But it is definitely not meant to be idiomatic Python.
That would require a much more complex compiler,
because idiomatic Python is not simple.

Hissp's basic macros are meant to be just enough to bootstrap native unit tests
and demonstrate the macro system.
They may suffice for small embedded Hissp projects,
but you will probably want a more comprehensive macro suite for general use.

Currently, that means using [Hebigo](https://github.com/gilch/hebigo),
which has macro equivalents of most Python statements.

The Hebigo project includes an alternative indentation-based Hissp reader,
but the macros are written in readerless mode and are also compatible with the
S-expression "Lissp" reader bundled with Hissp.

## Interoperability
Why base a Lisp on Python when there are already lots of other Lisps?

Python has a rich selection of libraries for a variety of domains
and Hissp can mostly use them as easily as the standard library.
This gives Hissp a massive advantage over other Lisps with less selection.
If you don't care to work with the Python ecosystem,
perhaps Hissp is not the Lisp for you.

Note that the Hissp compiler is written in Python 3.8.
(Supporting older versions is not a goal,
because that would complicate the compiler.
This may limit the available libraries.)
But because the compiler's target functional Python subset is so small,
the compiled output can usually run on Python 3.5 without too much difficulty.
Watch out for positional-only arguments (new to 3.8)
and changes to the standard library.
Running on versions even older than 3.5 is not recommended,
but may likewise be possible if you carefully avoid using newer Python features.

Python code can also import and use packages written in Hissp,
because they compile to Python.

## Useful error messages
One of Python's best features.
Any errors that prevent compilation should be easy to find.

## Syntax compatible with Emacs' `lisp-mode` and Parlinter
A language is not very usable without tools.
Hissp's basic reader syntax (Lissp) should work with Emacs.

## Standalone output
This is part of Hissp's commitment to modularity.

One can, of course, write Hissp code that depends on any Python library.
But the compiler does not depend on emitting calls out to any special
Hissp helper functions to work.
You do not need Hissp installed to run the final compiled Python output,
only Python itself.

Hissp includes some very basic Lisp macros to get you started.
Their expansions have no external requirements either.

Libraries built on Hissp need not have this restriction.

## REPL
A Lisp tradition, and Hissp is no exception.
Even though it's a compiled language,
Hissp has an interactive command-line interface like Python does.
The REPL displays the compiled Python and evaluates it.
Printed values use the normal Python reprs.
(Translating those to back to Lissp is not a goal.
Lissp is not the only Hissp reader.)

## Same-module macro helpers
Functions are generally preferable to macros when functions can do the job.
They're more reusable and composable.
Therefore, it makes sense for macros to delegate to functions where possible.
But such a macro should work in the same module.
This requires incremental compilation and evaluation of forms in Lissp modules,
like the REPL.

## Modularity
The Hissp language is made of tuples (and atoms), not text.
The S-expression reader included with the project (Lissp) is just a convenient
way to write them.
It's possible to write Hissp in "readerless mode"
by writing these tuples in Python.

Batteries are not included because Python already has them.
Hissp's standard library is Python's.
There are only two special forms: ``quote`` and ``lambda``.
Hissp does include a few basic macros and reader macros,
just enough to write native unit tests,
but you are not obligated to use them when writing Hissp.

It's possible for an external project to provide an alternative
reader with different syntax, as long as the output is Hissp code.
One example of this is [Hebigo](https://github.com/gilch/hebigo),
which has a more Python-like indentation-based syntax.

Because Hissp produces standalone output, it's not locked into any one Lisp paradigm.
It could work with a Clojure-like, Scheme-like, or Common-Lisp-like, etc.,
reader, function, and macro libraries.

It is a goal of the project to allow a more Clojure-like reader and
a complete function/macro library.
But while this informs the design of the compiler,
it is beyond the scope of Hissp proper,
and does not belong in the Hissp repository.
