# Python User Reference

Many many years ago (OK, it was 2008) a publisher recognised a potential gap
in Python's documentation: the tutorial and library reference explain how
to use the language, and the language reference explains how to implement
the language, but there isn't a particular clear explanation anywhere that
is aimed at helping Python users understand what their interpreter is
actually *doing* to execute the instructions that they give to it.

The Python User Reference attempts to fill that gap: it assumes the reader
already knows the basics of programming in Python, and aims to use that
foundation to build clearer mental pictures of *why* various high level
constructs behave the way that they do (often through the time-honoured
technique of putting `print` calls in various magic methods and showing
what happens).

## Project Status

The User Reference languished in the original manuscript's ODF files for
many years after the original book project was cancelled. It was revived
in 2019 when Cheryl Sabella started converting the original text to a
Sphinx project with a view to then starting to update it for the state
of things in 2019+, rather than 2008.

## License

The Python User Reference is licensed under a Creative Commons
Attribution-Share-Alike license (CC-BY-SA):
http://creativecommons.org/licenses/by-sa/3.0 .

## History

Initial copyright 2008 Nick Coghlan. Subsequently licensed to the Python Software
Foundation under a Python Contributor's Agreement.

The text in the ODF directory is based on a pre-release manuscript for a book
project which never went ahead. After discussion with Guido and the publisher,
I cleaned the files up for donation to the PSF under my existing
contributor agreement.

When the PSF's old subversion repository went away, I moved the files into a new
project under my GitHub account.

The project repository also includes a hacked version of Python 2.4's doctest.py
to support the script that ran the examples directly from the ODT files (for an
audience of one, modifying a copy of doctest was easier at the time than
figuring out how to do it properly).


## Conversion to reStructured Text

After I successfully failed to do anything about it for more than a decade, Cheryl
Sabella has recently started the conversion of the old ODF manuscript to
reStructured Text. Huzzah!


## TODO List

- make a new Sphinx project for modernised versions of the chapters
- cover the mere decade of changes since the original manuscript was written :)
- convert the publisher's XREF notation to Sphinx cross-references
- any items still flagged with XXX in the converted files
- check that sys.exc_info() is covered in the section on exception (Chapter 4)
- cover contextlib.ExitStack and contexlib.closing (Chapter 4)
- cover both sys.stderr and sys.stdout in section on printing to output streams (Chapter 2)
