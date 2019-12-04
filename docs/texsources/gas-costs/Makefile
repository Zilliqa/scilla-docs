all: gas-doc.pdf

quick: Makefile *.tex *.bib *.cls
	pdflatex gas-doc

gas-doc.pdf: Makefile *.tex *.bib *.cls
	pdflatex gas-doc

clean:
	rm -f *.aux *.bbl *.blg *.log *.out *.rel gas-doc.pdf gas-doc.idx

.PHONY: all clean *.tex *.bib *.cls
