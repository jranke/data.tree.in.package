PKGSRC  := $(shell basename $(CURDIR))
PKGNAME := $(shell sed -n "s/Package: *\([^ ]*\)/\1/p" DESCRIPTION)
PKGVERS := $(shell sed -n "s/Version: *\([^ ]*\)/\1/p" DESCRIPTION)
TGZ     := $(PKGNAME)_$(PKGVERS).tar.gz
RBIN ?= $(shell dirname "`which R`")

all: install

# Needs to be manually edited to include files that should be watched
# for changes
pkgfiles = \
	.Rbuildignore \
  DESCRIPTION \
	NAMESPACE \
	data/* \
	R/*

roxy:
	$(RBIN)/Rscript -e "roxygen2::roxygenize(roclets = c('rd', 'collate', 'namespace'))"

$(TGZ): $(pkgfiles)
	"$(RBIN)/R" CMD build . 2>&1 | tee log/build.log

build: roxy $(TGZ)

pd: roxy
	"$(RBIN)/Rscript" -e 'pkgdown::build_site()'

quickcheck: build
	_R_CHECK_CRAN_INCOMING_REMOTE_=false "$(RBIN)/R" CMD check $(TGZ) --no-tests

check: roxy build
	_R_CHECK_CRAN_INCOMING_REMOTE_=false "$(RBIN)/R" CMD check --as-cran --no-tests $(TGZ) 2>&1 | tee log/check.log

test: build
	"$(RBIN)/Rscript" -e 'library(devtools); devtools::test()' 2>&1 | tee log/test.log
	sed -i -e "s/\r.*\r//" test.log

install: build
	"$(RBIN)/R" CMD INSTALL --no-multiarch $(TGZ)

.PHONEY: check quickcheck install pd roxy test
