# Tutorial Makefile
#
# Copyright (C) 2001-2009 NLTK Project
# Author: Steven Bird <sb@csse.unimelb.edu.au>
#         Edward Loper <edloper@gradient.cis.upenn.edu>
# URL: <http://www.nltk.org/>
# For license information, see LICENSE.TXT

PAPER_SIZE = a4

CHAPTERS = ch00.rst ch01.rst ch02.rst ch03.rst ch04.rst ch05.rst ch06.rst\
           ch07.rst ch08.rst ch09.rst ch10.rst ch11.rst ch12.rst\
           app-tagsets.rst app-modules.rst

EXTRAS = ch00-extras.rst ch04-extras.rst ch08-extras.rst ch10-extras.rst

CONTENT = $(CHAPTERS) $(EXTRAS) term_index.rst

REF_EXTENSION = .ref

PUBLISH = ../../../doc

PDF := $(CONTENT:.rst=.pdf)
TEX := $(CONTENT:.rst=.tex)
HTML := $(CONTENT:.rst=.html)
ERRS := $(CONTENT:.rst=.errs)

XML := $(CHAPTERS:.rst=.xml)
REF := $(CHAPTERS:.rst=$(REF_EXTENSION))
PY := $(CHAPTERS:.rst=.py)

BIBTEX_FILE = ../refs.bib

PYTHON = python
PDFLATEX = TEXINPUTS=".:..:ucs:" pdflatex -halt-on-error
EXAMPLES = ../examples.py
LATEXHACKS = ../latexhacks.py
RST = ../rst.py
RST2REF = $(RST) --ref
RST2HTML = $(RST) --html
RST2DOCBOOK = $(RST) --docbook
RST2LATEX = $(RST) --latex --$(PAPER_SIZE)
DOCTEST = PYTHONPATH=../.. $(PYTHON) ../../nltk/test/doctest_driver.py
BIBTEX = bibtex -terse
#get from environment
#DOCBOOKDTD = /sw/share/xml/dtd/docbookx/4.4.0/docbookx.dtd
#DOCBOOKDTD = /opt/local/share/xml/docbook/4.4/docbookx.dtd

# <http://www.spinellis.gr/sw/textproc/bib2xhtml/>
BIB2XHTML = bib2xhtml -s named
LATEX_STYLESHEET_PATH = ../definitions.sty
RSTHACKS = python ../rsthacks.py

.SUFFIXES: .rst .rst2 .html .pdf .errs .py

help: usage
usage:
	@echo "Usage:"
	@echo "    make all         -- Build HTML & PDF output"
	@echo "    make html        -- Build HTML output"
	@echo "    make pdf         -- Build PDF output"
	@echo "    make xml         -- Build XML output"
	@echo "    make errs        -- Run doctest to generate .errs files"
	@echo "    make clean       -- Remove all built files"

all: html pdf examples clean_up
html: $(HTML) bibliography.html 
pdf: $(PDF)
xml: $(XML)
examples: $(PY)
book: book.pdf book.html
book.rst: $(CHAPTERS) $(REF)
book.html: book.rst
book.tex: book.rst

errs: $(ERRS)
	ls -l $(ERRS)

clean:	clean_up
	rm -f $(PDF) $(HTML) $(ERRS) $(REF) $(PY) $(TEX)
	rm -f book.pdf book.html book.tex bibliography.html *.rst2
	rm -rf tree_images

clean_up:
	rm -f *.log *.aux *.out *.errs *~ *.idx *.ilg *.ind *.toc *.blg

.PRECIOUS: $(REF)

%$(REF_EXTENSION): %.rst
	$(RST2REF) $<

%.html: %.rst $(REF) revision.rst
	$(RSTHACKS) --format=html $<
	$(RST2HTML) $(REF) `basename $< .rst`.rst2

%.xml: %.rst $(RST) ../docbook.py $(REF) revision.rst
	$(RSTHACKS) --format=xml $<
	$(RST2DOCBOOK) $(REF) `basename $< .rst`.rst2
	@(if [ "`grep -i 'preface::' $*.rst`" ]; then\
	    sed -e "s/chapter>/preface>/" -i "" $*.xml;\
	    sed -e "s/DOCTYPE chapter/DOCTYPE preface/" -i "" $*.xml;\
	fi)
#	$(PYTHON) ../xmlpp.py $@ > tmp.xml
#	mv tmp.xml $@
	xmllint --noout --dtdvalid $(DOCBOOKDTD) $@

%.tex: %.rst $(REF) revision.rst
	$(RSTHACKS) --format=latex $<
	$(RST2LATEX) $(REF) `basename $< .rst`.rst2
	$(LATEXHACKS) $@

revision.rst: $(CHAPTERS)
	echo "This document is" > revision.rst
	svn info | grep Revision >> revision.rst
	date >> revision.rst

%.bbl: %.tex $(BIBTEX_FILE)
	rm -f $*.bbl
	@echo "pdflatex $< -> $*.pdf (for bibtex)"
	@(true | $(PDFLATEX) $< >/dev/null) || \
	         (cat $*.log && rm -f $*.pdf && false)
	@echo $(BIBTEX) $*
	@(true | $(BIBTEX) $* || (rm -f $*.pdf $*.bbl && false))

bibliography.html: $(BIBTEX_FILE) bib_template.html
	cp bib_template.html $@
	$(BIB2XHTML) $< $@

# This fairly complex target here will cause pdflatex to only generate
# output if it fails; and to clean up after itself if it fails.  Also,
# it will only run a second pass if it detects that it's necessary (by
# scanning the log file for a warning message).
%.pdf: %.tex
	rm -f $*.ind
	@echo "pdflatex $< -> $*.pdf (first pass)"
	@(true | $(PDFLATEX) $< >/dev/null) || \
	         (cat $*.log && rm -f $*.pdf && false)
	@if [ "`grep -i 'Warning.*Rerun' $*.log`" ]; \
	    then \
	        echo "pdflatex $< -> $*.pdf (second pass)";\
	        (true | $(PDFLATEX) $< >/dev/null) || \
	        (cat $*.log && rm -f $*.pdf && false);\
	    fi
	@(if [ -e $*.idx ]; \
	    then \
	        echo "makeindex $*.idx"; makeindex $*.idx; \
	    fi)
	@echo "pdflatex $< -> $*.pdf (last pass)"
	@(true | $(PDFLATEX) $< >/dev/null) || \
	         (cat $*.log && rm -f $*.pdf && false)
	@echo "cleaning up latex temp files..."
	rm -f $*.log $*.aux $*.out $*.errs $*.bbl 
	rm -f $*.idx $*.ilg $*.ind $*.toc $*.blg

book.pdf: book.bbl book.tex

.rst.errs:
	$(DOCTEST) $< --ellipsis --normalize_whitespace --udiff &> $@

.rst.py:
	$(PYTHON) $(EXAMPLES) $< > $@

publish:
	find . \( -name '*.html' -or -name '*.pdf' -or -name '*.py' -or -name '*.png' \) -print |\
	        cpio -padu $(PUBLISH)/en/
	python ../../tools/svnmime.py $(PUBLISH)/en/* $(PUBLISH)/en/*/*
	svn add $(PUBLISH)/en/*
	echo Note that "make -C en publish" omits images and stylesheets