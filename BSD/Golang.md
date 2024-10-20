
To update FreeBSD port:

- Edit Makefile to edit DISTVERSION and optionally remove PORTREVISION
- Remove `distinfo`
- Run `make fetch` to fetch dependencies
- Run `make makesum` to update distinfo
- Run `gomod-vendor` (needs ports-mgmt/modules2tuple) and copy `GH_TUPLE=` lines to Makefile 
- Run `make makesum` to update distinfo
- Run `portlint .`

NetBSD:

- Edit Makefile to edit DISTNAME and optionally remove PKGREVISION
- Reset RCS tags setting the first line like this: `# $NetBSD$`
- Run `make fetch` to fetch dependencies
- Run `make show-go-modules > go-modules.mk`
- Run `make makesum` to update distinfo
- Run `pkglint .`

Before submitting:

- Test building & installing with `make` & `make install`
- Run `git commit`
- Run `git format-patch HEAD~1` to submit to Bugzilla with `maintainer-approval` flag
- Optionally check compilation on 32-bit arch when using Go 1.22+

More info:
- https://docs.freebsd.org/en/books/porters-handbook/book/
- https://www.netbsd.org/docs/pkgsrc/creating.html
