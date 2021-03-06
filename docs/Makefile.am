IMAGE_FILES = images/asciidoc.png \
	images/hostedby.png \
	images/nut_layering.png \
	images/nut-logo.png \
	images/note.png \
	images/warning.png \
	images/blue-arrow.png \
	images/simple.png \
	images/advanced.png \
	images/bigbox.png \
	images/bizarre.png \
	images/old-cgi.png

# Only track here the local deps
SHARED_DEPS = nut-names.txt daisychain.txt asciidoc.conf asciidoc.txt

USER_MANUAL_DEPS = acknowledgements.txt cables.txt config-notes.txt	\
 configure.txt download.txt documentation.txt features.txt history.txt	\
 outlets.txt scheduling.txt security.txt support.txt user-manual.txt

DEVELOPER_GUIDE_DEPS = contact-closure.txt design.txt developers.txt	\
 developer-guide.txt hid-subdrivers.txt macros.txt new-clients.txt	\
 new-drivers.txt net-protocol.txt nutdrv_qx-subdrivers.txt	\
 snmp-subdrivers.txt sock-protocol.txt

CABLES_DEPS = cables/apc-rs500-serial.txt	\
 cables/apc.txt cables/ge-imv-victron.txt cables/imv.txt		\
 cables/mgeups.txt cables/powerware.txt cables/repotec.txt		\
 cables/sms.txt

CABLES_IMAGES = images/cables/73-0724.png images/cables/940-0024C.jpg \
 images/cables/belkin-f6cx-rkm-xu-cable.jpg images/cables/Lansafecable.jpg \
 images/cables/mac-940-0024C.png images/cables/mge-66049.png \
 images/cables/mge-db9-rj12.jpg images/cables/mge-db9-rj45.jpg \
 images/cables/mge-usb-rj45.jpg \
 images/cables/SOLA-330.png

ALL_TXT_SRC = nut-names.txt daisychain.txt \
 $(USER_MANUAL_DEPS) $(DEVELOPER_GUIDE_DEPS) \
 $(CABLES_DEPS) FAQ.txt nut-qa.txt packager-guide.txt snmp.txt

NUT_SPELL_DICT = nut.dict
EXTRA_DIST = $(ALL_TXT_SRC) $(SHARED_DEPS) $(IMAGE_FILES) \
 $(CABLES_IMAGES) $(NUT_SPELL_DICT) \
 common.xsl xhtml.xsl chunked.xsl asciidoc.txt

ASCIIDOC_HTML_SINGLE = user-manual.html \
	developer-guide.html \
	packager-guide.html \
	FAQ.html

ASCIIDOC_HTML_CHUNKED = user-manual.chunked \
	developer-guide.chunked \
	packager-guide.chunked \
	FAQ.html

ASCIIDOC_PDF = user-manual.pdf \
	developer-guide.pdf \
	packager-guide.pdf \
	cables.pdf	\
	FAQ.pdf

SUBDIRS = man
SUFFIXES = .txt .html .pdf

all: doc

doc: @DOC_BUILD_LIST@

pdf: $(ASCIIDOC_PDF)
# also build the HTML manpages with these targets
html-single: $(ASCIIDOC_HTML_SINGLE)
html-chunked: $(ASCIIDOC_HTML_CHUNKED)

clean-local:
	rm -rf *.pdf *.html *.chunked docbook-xsl.css *.bak

### TODO: automatic dependency generation
# Add other directory deps (not for local EXTRA_DIST) and generated contents
FULL_USER_MANUAL_DEPS = $(USER_MANUAL_DEPS) $(SHARED_DEPS) ../README \
	../INSTALL.nut ../UPGRADING ../TODO ../scripts/ufw/README
FULL_DEVELOPER_GUIDE_DEPS = $(DEVELOPER_GUIDE_DEPS) $(SHARED_DEPS) \
	../scripts/augeas/README ../TODO ../lib/README \
	../tools/nut-scanner/README

user-manual.html user-manual.chunked user-manual.pdf: $(FULL_USER_MANUAL_DEPS)
developer-guide.html developer-guide.chunked developer-guide.pdf: $(FULL_DEVELOPER_GUIDE_DEPS)
packager-guide.html packager-guide.chunked packager-guide.pdf: packager-guide.txt asciidoc.conf

# Note: without the "-v", asciidoc (circa 8.6.2) sometimes hangs when
# generating the chunked HTML. In this case, export the environment
# variable ASCIIDOC_VERBOSE to "-v", ie:
#   $ ASCIIDOC_VERBOSE=-v make
A2X_COMMON_OPTS = $(ASCIIDOC_VERBOSE) --attribute icons \
    --xsltproc-opts "--nonet" \
    --xsltproc-opts "--stringparam nut.localdate \"`TZ=UTC date +%Y-%m-%d`\"" \
    --xsltproc-opts "--stringparam nut.localtime \"`TZ=UTC date +%H:%M:%S`\"" \
    --xsltproc-opts "--stringparam nut.nutversion \"@PACKAGE_VERSION@\"" \
    --attribute iconsdir=$(srcdir)/images \
    --attribute=badges \
    --attribute=external_title \
    --attribute tree_version=@TREE_VERSION@ \
    -a toc -a numbered --destination-dir=.
# NOTE: a2x newer than 8.6.8 says "--destination-dir" is only valid for HTML.
# As of version 8.6.9 it lies, and the argument is required for our distcheck.
# For more details see issues https://github.com/asciidoc/asciidoc/issues/44
# and https://github.com/networkupstools/nut/pull/281 (in short, attempts
# to "fix" this warning broke NUT build). If this is to be retried later, see
# https://github.com/networkupstools/nut/pull/281/commits/fe17861c4ea12679b3ebfefa8a6d692d79d99f2d
# and do not forget to fix up docs/man/Makefile.am too ;)

.txt.html: common.xsl xhtml.xsl
	$(A2X) $(A2X_COMMON_OPTS) --attribute=xhtml11_format --format=xhtml --xsl-file=$(srcdir)/xhtml.xsl $<

.txt.chunked: common.xsl chunked.xsl
	$(A2X) $(A2X_COMMON_OPTS) --attribute=chunked_format --format=chunked --xsl-file=$(srcdir)/chunked.xsl $<

.txt.pdf: docinfo.xml
	$(A2X) $(A2X_COMMON_OPTS) --attribute=pdf_format --format=pdf -a docinfo1 $<

if HAVE_ASPELL
# Non-interactively spell check all documentation source files.
# This is useful for Buildbot and automated QA processing
# FIXME: how to present output (std{out,err}, single file or per target)?
SPELLCHECK_SRC = $(ALL_TXT_SRC) ../README ../INSTALL.nut ../UPGRADING  ../NEWS \
	../TODO ../scripts/ufw/README ../scripts/augeas/README ../lib/README \
	../tools/nut-scanner/README

# Note: de-facto our documentation is beyond ASCII (at least in names of
# international committers). The grep tests below look if the aspell output
# contained something other than the OK lines (tagged with asterisk) and
# aspell's version (tagged with @) and if it did - those lines must be the
# spellcheck complaints. Empty OUT is ok.
# We also must indent the input, because certain piped-in characters are
# interpreted as commands, and seems this feature can not be turned off.
# See also http://aspell.net/man-html/Through-A-Pipe.html
# TODO: Is "grep -a" or "grep -b" (treat input as ascii/bin) portable enough?
# Is "egrep == grep -E" always valid? (maybe all a job for configure.ac)
# Set SPELLCHECK_ERROR_FATAL=no if there are some unavoidable issues
# due to spellchecking, to temporarily not fail builds due to this.
# For Travis CI in particular, see ci_build.sh in NUT codebase root.
SPELLCHECK_ERROR_FATAL = yes
SPELLCHECK_ENV_DEBUG = no
ASPELL_NUT_COMMON_ARGS = -p $(abs_srcdir)/$(NUT_SPELL_DICT)
ASPELL_NUT_COMMON_ARGS += -d en --lang=en --ignore-accents
ASPELL_NUT_COMMON_ARGS += --encoding=utf-8
ASPELL_ENV_LANG = en.UTF-8
ASPELL_OUT_NOTERRORS = (^[ \t]*[\*\@]|^$$)
spellcheck: 
	@if test "$(SPELLCHECK_ENV_DEBUG)" != no ; then \
		echo "ASPELL DEBUG : information about the setup follows:"; \
		LANG=$(ASPELL_ENV_LANG); LC_ALL=$(ASPELL_ENV_LANG); export LANG; export LC_ALL; \
		$(ASPELL) --help || true; \
		dpkg -l |grep -i aspell || true ; \
		echo "ASPELL automatic execution line is : ( sed 's,^\(.*\)$$, \1,' < docfile.txt | $(ASPELL) -a -t $(ASPELL_NUT_COMMON_ARGS) | egrep -b -v '$(ASPELL_OUT_NOTERRORS)' )" ; \
		echo "ASPELL proceeding to spellchecking job..."; \
	 else true; fi
	@FAILED="" ; LANG=C; LC_ALL=C; export LANG; export LC_ALL; \
	 for docsrc in $(SPELLCHECK_SRC); do \
		echo "  ASPELL   Spell checking on $$docsrc"; \
		OUT="`sed 's,^\(.*\)$$, \1,' < $$docsrc | $(ASPELL) -a -t $(ASPELL_NUT_COMMON_ARGS) 2>&1`" && \
			{ if test -n "$$OUT" ; then OUT="`echo "$$OUT" | grep -E -b -v '$(ASPELL_OUT_NOTERRORS)' `" ; fi; \
			  test -z "$$OUT" ; } || \
			{ echo "FAILED : Aspell reported errors here:" >&2 && echo "----- vvv" >&2 && echo "$$OUT" >&2 && echo "----- ^^^" >&2 && \
			  FAILED="$$FAILED $$docsrc"; } ; \
	 done ; \
	 if test -n "$$FAILED" ; then \
		echo "=====================================================================" ; \
		echo "FAILED automatic spellcheck for the following sources: $$FAILED" ; \
		echo "=====================================================================" ; \
		echo "Please 'cd $(abs_srcdir) && make spellcheck-interactive'"; \
		echo "to either fix document sources or update the dictionary of accepted"; \
		echo "words and spellings listed in the '$(NUT_SPELL_DICT)' file there."; \
		echo "Either way, please follow up by posting a pull request or a patch"; \
		echo "to integrate your fixes into the common NUT codebase."; \
		echo "=====================================================================" ; \
		test x"$(SPELLCHECK_ERROR_FATAL)" = xno || exit 1; \
		echo "NOTE: SPELLCHECK_ERROR_FATAL == no so this make does not break the build!"; \
		echo "=====================================================================" ; \
	 fi >&2 ; exit 0

# Interactively spell check all documentation source files (so a human
# can edit the documentation errors and/or add words to custom dictionary).
# Note that here we do not restrain reported issues, so this might catch more
# than the automated test above.
spellcheck-interactive: 
	@FAILED="" ; for docsrc in $(SPELLCHECK_SRC); do \
		echo "Spell checking on $$docsrc"; \
		LANG=$(ASPELL_ENV_LANG) LC_ALL=$(ASPELL_ENV_LANG) $(ASPELL) check $(ASPELL_NUT_COMMON_ARGS) $$docsrc || \
			FAILED="$$FAILED $$docsrc"; \
	done ; \
	if test -n "$$FAILED" ; then \
		echo "FAILED interactive spellcheck for the following sources: $$FAILED" >&2 ; \
		exit 1; \
	fi ; exit 0
else !HAVE_ASPELL
spellcheck:
	@echo "Documentation spell check not available since 'aspell' was not found."
spellcheck-interactive:
	@echo "Documentation spell check not available since 'aspell' was not found."
endif !HAVE_ASPELL

.PHONY: html html-single pdf
