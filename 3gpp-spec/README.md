The purpose of this project is to put the [3GPP](http://www.3gpp.org/) specifications in a version control system.
This is accomplished using the `Rakefile` script.

requirement
===========

To perform all operations, the `Rakefile` script requires the following components:

- `rake`: to run the `Rakefile` (the ruby alternative of `Makefile`)
- `wget`: to download the 3GPP specifications and `tika`
- `rubyzip`: to extract the .doc specification documents from the downloaded .zip archives
- `java`: to convert the .doc specification documents to machine readable .xhtml documents using `tika`.
- `nokogiri`: to parse the .xhtml specification documents and output .txt documents
- `git`: to put the resulting .txt specification documents in a revision control

To install all requirements on a Debian based distribution:

  sudo apt install rake wget ruby-zip default-jre ruby-nokogiri git

steps
=====

download
--------

The 3GPP specification are available [online](http://www.3gpp.org/ftp/Specs/archive/).
There also is an [FTP mirror](ftp://ftp.3gpp.org/Specs/archive/).
While it is faster to download then over HTTP the first time, updating them afterwards is significantly faster over FTP (~15 minutes over FTP vs >90 minutes over HTTP when no new file is present).

`rake download` will download all publicly available specification, or get the new ones when not already downloaded.
The first time `rake` is used this will be done automatically, but this task should then be done by hand periodically to get new specifications.

The specifications come in .zip archives containing the .doc specification documents.

extract
-------

The .doc specification documents are extracted from the .zip archives.
The file name of the .zip archive is use the get the specification number and version.
These are then used to extract the corresponding .doc specification document.

`rake extract` will extract all .doc specification documents from all .zip archives.
Else the corresponding .doc specification documents are extracted from .zip archives when need to generate the .XHTML documents for the particular specification.

convert
-------

The .doc specification documents are converted to machine readable .xhtml documents using *tika*.

`rake convert` will convert all .doc specification documents to .xhtml documents.
Else the corresponding .doc specification documents is converted to the .xhtml document when need to generate the final text document for the particular specification.

NOTE: This is the longest step.

text
----

The .xhtml specification documents are converted to cleaner .txt documents.
All embedded graphics, diagrams, and page information are removed since we are only interested in the text changes.

`rake text` will convert all .xhtml specification documents to .txt documents.
Else the corresponding .xhtml specification documents is converted to the .text document when need to generate the final text document for the particular specification.

versioning
----------

The generated .txt specification documents correspond to single versions of a specification.
These become the final specification xx.yyy.txt text file.
The individual versions are committed into the git repository.

`rake ftp.3gpp.org/Specs/archive/xx_series/xx.yyy/xx.yyy.txt` will generate and commit a single specification.
`rake` will generate and commit all specifications.

exceptions
==========

The 3GPP specification and created, maintained, and only meant for humans.
Trying got find a scheme able to handle all documents is pretty hard.
The script allows to add exceptions for specifications that don't match the general schemes.

The script will first collects a list of all specifications.
Wrongly formatted specification archive names are displayed.
Add these to the **BAD_ARCHIVE** list to ignore these.
 
extract
-------

Sometimes the .doc specification document file name does not correspond to the specification number and version extracted from the .zip archive file name.
In this case a warning message will be displayed and an empty .doc file is created, so to avoid trying to extract the document every time the final text specification needs to be generated.

You can add the actual .doc specification document file name corresponding to the expected file name in the **DOC_FILENAME** list.
Also delete the existing empty .doc file.
The specified .doc will automatically be extracted and used to generate the final text specification next time the specification needs to be generated, or `rake` is run to generate all text specifications.

Sometimes the .zip archive does not contain a .doc(x) specification document, but a .txt text, or again a .zip archive containing the final .doc documents.
This is not handled by the script.
Extract the .doc file by hand to the expected .doc file, and this will then be used to generate the text specification.

convert
-------

Sometimes the .doc specification document uses the old Microsoft Word format not supported by *tika*, or is malformed.
In this case a warning message will be displayed and an empty .xhtml file is created, so to avoid trying to extract the document every time the final text specification needs to be generated.
To prevent this warning to be displayed add the file path in the **DOC_NOCONVERT** list.

You can also try to manually fix the malformed .doc file into a regular .doc file (readable by *tika*).
The new .doc file will automatically be converted to an .html document and used to generate the final text specification next time the specification needs to be generated, or `rake` is run to generate all text specifications.

versioning
----------

Extracting the specification version from the document is a hard task due to the various formatting of the documents.
It also happens that the actual version in the document is wrong.
Use the **FILENAME_VERSION** list to enforce a specification number.
If **USE_FILENAME_VERSION** is set the script will use the version information extracted from the file name in case it does not find the version information in the document of this is erroneous.

cleaning
--------

`rake clean_bogus` will remove empty .txt, .xhtml, and .doc specification version files.
These files are generated during the above documents steps when the documents are malformed to prevent re-performing the steps every time.

`rake clobber` will remove the gir repository and .txt specifications.

`rake clean` will remove all  .txt, .xhtml, and .doc specification version files.

tips & tricks
=============

Because of the large number of .doc, .xhtml, and .txt intermediate files, it is recommended to use a file system offering transparent compression.
For example BTRFS with zstd compression uses 136 GB of disk space for 228 GB of total file size (use `btrfs filesystem df /mountpoint` to get the used disk space).

`rake stats` will show the number and total size of the different specification files.

future
======

- set commit date to specification release (or file) date
- set author to working group
- add a task to get all specification up to a version (should create a new branch and pick the commits of the specification up to a version using the commit tags)
- improve specification version extraction
- improve exception and version list
- extract release notes from document and put in commit message
