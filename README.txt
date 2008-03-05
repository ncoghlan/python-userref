#
# Python User's Reference
#
#   Copyright 2008 Nick Coghlan
#   Licensed to the Python Software Foundation under a Python Contributor's Agreement
#

The text in the ODF directory is based on a pre-release manuscript for a book project which is no longer going ahead. After discussion with Guido and the publisher, I am cleaning the files up for donation to the PSF under my existing contributor agreement.

Any thoughts on how to efficiently convert these files to ReST markup would be appreciated – it seems to me that it should be possible to use the style information in the ODT XML to generate markup that is appropriate for input to the Python doc building system.

If I can figure out how to do the ReST conversion, can get the files up to date for Python 2.6/3.0 in a timely fashion and get enough positive feedback from doc-sig and python-dev, they may be worth considering as the backbone of a new section in the standard Python docs. Alternatively, they may prove to be a useful resource for folks updating other parts of the documentation or working on their own documentation projects, even if they don't end up being used directly

The sandbox directory also includes a hacked version of Python 2.4's doctest.py to support the script that runs the examples directly from the ODT files (for an audience of one, modifying a copy of doctest was easier at the time than figuring out how to do it properly).

=========
TODO List
=========

- any items flagged with XXX in the ODT files
- check that sys.exc_info() is covered in the section on exception (Chapter 4)
- cover contextlib.nested and contexlib.closing (Chapter 4)
- cover both sys.stderr and sys.stdout in section on printing to output streams (Chapter 2)
