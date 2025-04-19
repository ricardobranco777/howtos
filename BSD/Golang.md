
To update FreeBSD port:

- Edit Makefile to edit DISTVERSION and optionally remove PORTREVISION
- Remove `distinfo`
- Run `make fetch` to fetch dependencies
- Run `make makesum` to update distinfo
- Run `make gomod-vendor` (needs ports-mgmt/modules2tuple) and copy `GH_TUPLE=` lines to Makefile 
- Run `make makesum` to update distinfo
- Run `portlint .`

NetBSD:

- Edit Makefile to edit DISTNAME and optionally remove PKGREVISION
- Reset RCS tag: `sed -i -e '1s/# \$NetBSD.*/# $NetBSD$/' Makefile`
- Run `make fetch` to fetch dependencies
- Run `make show-go-modules > go-modules.mk`
- Run `make makesum` to update distinfo
- Run `pkglint .`

OpenBSD:

- Edit version on Makefile
- Remove `distinfo`
- Run `go install github.com/$gh_account/$project@latest` to get `MODGO_VERSION`
- Run `make modgo-gen-modules > modules.inc`
- Run `make makesum` to update distinfo
- Send uncompressed tarball or diff to ports@openbsd.org

Before submitting:

- Test building & installing with `make` & `make install`
- Run `git commit`
- Run `git format-patch HEAD~1` to submit to Bugzilla with `maintainer-approval` flag
- Optionally check compilation on 32-bit arch when using Go 1.22+

More info:
- https://docs.freebsd.org/en/books/porters-handbook/special/#using-go
- https://www.netbsd.org/docs/pkgsrc/creating.html
- https://man.openbsd.org/go-module.5
