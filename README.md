This python script makes creating TeX/LaTeX document indexes much easier. It takes a LaTeX source
and an index terms file, and produces LaTeX source with \index{...} commands inserted.

When you want to create indexes in LaTeX, there are two main difficulties:
# \index{...} commands will make the document source quite difficult to read and edit.
# It is quite easy to miss locations where a desired index term appears.

Both of these tasks are much easier done by computer rather than by a human. texindex.py 
uses an index terms file which has the following syntax (lines starting with # are comments):

    # an entry like "term:"  causes "term" to be used as an index term
    Python:

in effect, the above will cause any appearance of "Python" in the document to be replaced by
Python\index{Python}. 

texindex.py *ignores the case*, i.e. "pYthon" will be replaced by "pYthon\index{Python}". Also it
will *match the term as a prefix*. i.e. "pythonized" will be replaced by "python\index{Python}ized".

There are a few other ways to define terms:

    # an entry like "term-label:alternative1, alternative2, ..." causes any of the alternatives to be indexed as the same term
    agent-based:agent-based,agent based

The above example will index either "agent-based" or "agent based" (or "is-agent-based" or "Agent Based", for that matter) as index entry "agent-based".

    # an entry like "label!sublabel:" or "label!sublabel:alternative1, alternative2, ..."
    # will create leveled indexes.
    Programming!Python:
    Programming!C: C++, objective-c

The above example will create sub-index entries.

You must make sure longer entries which contain others come first. Because texindex will search and replace the terms
in the order they appear in index terms file. For example the following is correct:
    cognitive distance:
    cognitive:

However, if you instead write:
    cognitive:
    cognitive distance:

Then you will not find any "cognitive distance" index terms. Because once the term "cognitive" is taken care of, any instances of "cognitive distance" will be *dirty* as "cognitive\index{cognitive} distance"; therefore texindex.py will
not find the second term.

Running the program
-------------------

    $ python texindex.py mybook.tex indexterms.txt > mybookindexed.tex

Multi-part files
-------------------
texindex.py will replace any appearance of \include{somefile.tex} with the actual contents of the file, prior to
creating index terms. Thus it will generate a complete source file for LaTeX projects that are split into multiple source
files.

