# WhileyLanguageTests

A suite of acceptance tests for tools (e.g. compilers / verifiers) working with Whiley.  These test clarify expected semantics of the language, as well as expectations regarding verification performance.

## Overview

Tests are written using the test file format setout in
[RFC#110](https://github.com/Whiley/RFCs/blob/master/text/0110-test-file-format.md).
This means test can cover various aspects of the language:

   * **Multiple Files**.  An individual test is not limited to a
     single `whiley` file, but can describe multiple files spread
     across an arbitrary directory structure.
     
   * **Frames**.  An individual test is made up of one or more
     _frames_.  In the first frame, the initial state of files is set
     out.  After that, frames can include incremental updates to
     specific files, deletion of files, etc.
     
   * **Errors**.  After each frame in a test, the expected set of
     errors are listed (if any are expected).  This means, for
     example, a test can start out with an initial file that fails to
     compile, and then update that file (one or more times) to the
     point where it now compiles.  Errors are listed based on the
     error codes set out in the [language
     specification](https://github.com/Whiley/WhileyDocs/tree/master/WhileyLanguageSpecification).
     
   * **Static / Dynamic**.  Testing covers the static checking aspects
     of the language (e.g. type checking), but also runtime execution.
     This means tests can be used to clarify the expected language
     [semantics](https://en.wikipedia.org/wiki/Semantics_(computer_science)).
     
   * **Ignores**.  Tests can be marked as _ignored_ with respect to
     the compiler as a whole, or a specific backend.  For example, if
     a test does not compile correctly on the JavaScript backend, it
     can be marked as ignored for that backend only.

Overall, the test file format has proved remarkably flexible to date.
Some aspects, however, are not covered.  For example, there is no
mechanism for performance regression testing, etc.

## Example

An example test (`001024.test`) is

```
original.name="MethodCall_Invalid_8"
======
>>> main.whiley
function f() -> int:
    &int x = new 1
    return 1
---
E607 main.whiley 2,13:17
```

This is about the simplest possible test, and consists of only one
frame.  Meta-data for the test is given before the first frame begins
(which in this case identifies the original name given to this test).
This test expects the given file to fail compilation with an error
`E607` (as memory allocation is not permitted in a `function`).

Another more detailed test (`001398.test`) is the following:

```
js.compile.ignore=true
=====
>>> main.whiley
type Rec is {int f, int g}

public export method test():
   Rec xs = Rec{f:123, g:456}
   assert xs == {f:223, g:789}
---
E705 main.whiley 5,10:29
E722 main.whiley 5,10:29
=====
>>> main.whiley 5:6
   assert xs{g:=789}{f:=223} == {f:223, g:789}
---
```

In this case, the metadata provided indicates that we cannot yet
compiler this on the JavaScript backend.  The test consists of two
frames.  Initially, the file `main.whiley` is expected to fail at
runtime with (`E705`) and also to fail verification (`E722`).  Then,
in the second frame, line `5` from the original file is replaced with
a different assertion and test now compiles, verifies and executes
without problem.
